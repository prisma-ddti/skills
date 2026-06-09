# DDTI Agent Skills

Kumpulan Agent Skills yang dikembangkan oleh **Bidang Data, Digitalisasi, dan Teknologi Informasi Universitas Islam Malang** untuk mendukung pekerjaan pengembangan perangkat lunak di lingkungan Perpustakaan.

[Agent Skills](https://agentskills.io/home) adalah modul pengetahuan dan instruksi yang dimuat oleh AI Agents ketika menghadapi pekerjaan yang relevan, sehingga jawabannya mengikuti praktik terbaik.

Repositori ini menjadi tempat kami mengumpulkan pengetahuan nonteknis/teknis ke dalam _skill_ yang dapat dipakai ulang, mulai dari sistem yang kami kelola saat ini hingga kebutuhan baru yang muncul seiring waktu.

- **Terbuka** - siapa pun boleh mempelajari, menyalin, atau memakainya.
- **Berbasis pengalaman** - disusun dari praktik dan sistem nyata yang kami tangani, bukan tebakan model.
- **Bertumbuh** - skill ditambahkan sesuai kebutuhan pengembangan, tidak terbatas pada satu sistem.

## Cara Menggunakan

Setiap folder di `skills/` adalah satu skill (berkas `SKILL.md` + berkas pendukung). Salin folder skill yang diinginkan ke direktori skill agen Anda, lalu agen akan memuatnya otomatis saat pekerjaan relevan terdeteksi.

### Claude Code

- Salin folder skill ke salah satu lokasi:
  - User (semua workspace): `~/.claude/skills/<nama-skill>/`
  - Per-workspace: `.claude/skills/<nama-skill>/`
- Skill aktif otomatis saat relevan, atau panggil manual dengan `/<nama-skill>`

### Codex

- Salin folder skill ke salah satu lokasi:
  - User (semua workspace): `~/.codex/skills/<nama-skill>/`
  - Per-workspace: `.agents/skills/<nama-skill>/`
- Muat ulang Codex agar skill terdeteksi. Skill aktif otomatis saat relevan, atau panggil manual dengan `$<nama-skill>`

> Pastikan seluruh isi folder (`SKILL.md`, `references/`, `assets/`) ikut tersalin — di situlah detail rujukannya berada.
