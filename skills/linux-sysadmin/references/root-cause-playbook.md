# Root-Cause Playbook

This is not a command tutorial — it is a map from a surface symptom to the _non-obvious_ root causes that actually produce it, with the single command that confirms each. Work top to bottom; the cheap discriminating check comes first.

## "Disk is full" but the numbers don't add up

| Observation                                                   | Real root cause                                                                             | Confirm                                                        | Fix                                                              |
| ------------------------------------------------------------- | ------------------------------------------------------------------------------------------- | -------------------------------------------------------------- | ---------------------------------------------------------------- |
| `df` says 100% full, `du -xhd1` doesn't account for it        | A deleted file is still held open by a running process; space frees only when the fd closes | `sudo lsof +L1` (link count 0, still open)                     | Restart the holding process; don't just `rm` again               |
| `df` shows space free, writes still fail with `No space left` | **Inode** exhaustion, not bytes — millions of tiny files                                    | `df -i` (IUse% at 100%)                                        | Find and prune the offending dir (often a cache/session/maildir) |
| One filesystem full, others fine                              | A mount you forgot about, or logs/journal unbounded                                         | `du -xhd1 /var \| sort -h \| tail` ; `journalctl --disk-usage` | Rotate logs; cap journal with `SystemMaxUse=`                    |

The `-x` on `du` matters — it stays on one filesystem so a mounted volume doesn't skew the count.

## Service won't start / keeps restarting

| Symptom in `systemctl status` / `journalctl -u`              | Root cause                                                                     | Confirm                                                                                            |
| ------------------------------------------------------------ | ------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------- |
| `Address already in use`                                     | Stale instance or another daemon holds the port                                | `ss -tlnp \| grep :<port>`                                                                         |
| `Permission denied` reading config/socket, runs fine as root | Service user lacks access, **or** SELinux/AppArmor denial                      | `sudo -u <svcuser> <cmd>` ; `journalctl -b \| grep -iE 'avc\|apparmor'`                            |
| Starts then exits 0/203/217                                  | `ExecStart` path wrong, missing binary, or `User=`/`WorkingDirectory=` invalid | `systemd-analyze verify <unit>` ; `systemctl cat <unit>`                                           |
| Config changes ignored                                       | Edited the unit but never reloaded                                             | `sudo systemctl daemon-reload` then restart                                                        |
| `start request repeated too quickly`                         | Crash loop hitting `StartLimit`; the _real_ error is earlier in the log        | `journalctl -u <unit> --since "10 min ago"` and read the FIRST failure, not the rate-limit message |

The rate-limit message is a symptom of a symptom — always scroll back to the first crash in the cycle.

## Permission denied that root doesn't see → suspect MAC

If it works as root but fails as the service, and Unix permissions look correct, it is almost always SELinux (.rpm) or AppArmor (.deb):

```bash
journalctl -b | grep -iE 'avc|apparmor|denied'   # the denial, with the path/port it wanted
```

- **.rpm / SELinux** — common with a non-default port or path. Fix the label/policy, don't disable enforcement:
  - Custom SSH/web port: `sudo semanage port -a -t <type> -p tcp <port>`
  - Wrong file context after moving files in: `sudo restorecon -Rv /path`
- **.deb / AppArmor** — `sudo aa-status`; adjust the profile under `/etc/apparmor.d/`, reload with `apparmor_parser -r`.

Disabling SELinux/AppArmor "to make it work" hides the real misconfiguration and is the classic symptom-patch to call out.

## "Read-only file system" when the path is clearly writable → the unit's own sandbox

A service fails to write to a directory it owns, Unix permissions are correct, and SELinux/AppArmor show no denial. The cause is often **systemd's own filesystem protection inside the unit** — the daemon's config mounts the path read-only for the service, so the fix is to edit that config, not the filesystem.

The tell is the _error class_: `Read-only file system` (EROFS) with correct ownership points at a read-only mount/sandbox, not at perms — `Permission denied` (EACCES) is the ownership/MAC case above.

```bash
systemctl cat <unit>                                              # see the hardening directives in context
systemctl show <unit> -p ProtectSystem -p ReadOnlyPaths -p ReadWritePaths -p ProtectHome
```

Common culprits: `ProtectSystem=strict` (whole tree read-only), `ProtectSystem=full` (`/usr`, `/boot`, `/etc` read-only), `ReadOnlyPaths=`, or `ProtectHome=`. Fix by carving out exactly the path the service needs to write, via a drop-in override — don't disable the protection wholesale:

```bash
sudo systemctl edit <unit>          # creates an override.conf drop-in
```

```ini
[Service]
ReadWritePaths=/var/lib/myapp /var/log/myapp
```

```bash
sudo systemctl daemon-reload && sudo systemctl restart <unit>
```

Relaxing `ProtectSystem` entirely also "works", but throws away the hardening; `ReadWritePaths=` keeps it and grants only the exception actually needed — the minimal reversible fix.

## High load / slow box

Distinguish _what kind_ of busy before optimizing anything:

| `uptime` load high + ... | Root cause class                                                     | Confirm                                     |
| ------------------------ | -------------------------------------------------------------------- | ------------------------------------------- |
| high `%us` in `top`      | CPU-bound application                                                | `pidstat 1` for the offending PID           |
| high `%wa` (iowait)      | Disk I/O bottleneck, not CPU                                         | `iostat -xz 1 3` (look at `%util`, `await`) |
| load high but CPUs idle  | Processes stuck in **D** state (uninterruptible I/O), often NFS/disk | `ps -eo state,pid,cmd \| grep '^D'`         |
| `%sy` high               | Syscall/interrupt storm, context-switch thrash                       | `vmstat 1` (cs, in columns)                 |

Load average counts runnable _and_ uninterruptible tasks, so a high number with idle CPUs points at I/O, not compute.

## Out of memory / process killed mysteriously

A process vanishing with no app-level error is usually the kernel OOM killer:

```bash
journalctl -k | grep -i 'killed process'    # which PID, and the oom_score that doomed it
```

Root cause is rarely "not enough RAM" alone — look for a leak (RSS growing over time), an unbounded cache, or a missing `MemoryMax=`/cgroup limit letting one service starve the rest. `free -h` showing low free but high `available` is normal (page cache), not a problem.

## Network: "it can't connect"

Walk the layers; stop at the first that fails — that is the root cause, everything below it is consequence:

```bash
ip a                       # 1. does the interface have the expected address?
ip r                       # 2. is there a route (default gateway)?
ping -c3 <gateway>         # 3. L2/L3 to the gateway
dig <name>  +short         # 4. does DNS resolve? (empty = DNS, not the app)
ss -tlnp | grep :<port>    # 5. is the service even listening, and on which address?
curl -sI http://<host>:<port>   # 6. application-layer response
```

Two classic root causes that masquerade as "the app is down":

- Service bound to `127.0.0.1` instead of `0.0.0.0`/a real address — listening, but only to itself. `ss -tlnp` shows the bind address.
- DNS resolves but to a stale/wrong IP — compare `dig` output against the expected host.

## What changed? — the fastest root-cause question

When something worked yesterday and not today, the cause is usually a change, not entropy:

```bash
journalctl --since "yesterday" -p warning      # what the system logged around the break
grep -rl '.' /etc --include='*' -newermt '-2 days' 2>/dev/null   # config files edited recently
rpm -qa --last | head     # (.rpm) recently installed/updated packages
zcat -f /var/log/apt/history.log* | grep -A1 'Start-Date' | tail   # (.deb) recent apt actions
```

Pin the change, and you usually have the cause.
