# DDTI Agent Skills

Kumpulan _[Agent Skills](https://agentskills.io/home)_ yang dikembangkan oleh **Bidang Data, Digitalisasi, dan Teknologi Informasi Universitas Islam Malang** untuk mendukung pekerjaan pengembangan perangkat lunak di lingkungan Perpustakaan.

_Agent Skills_ adalah modul pengetahuan dan instruksi yang dimuat oleh AI Agents ketika menghadapi pekerjaan yang relevan, sehingga jawabannya mengikuti praktik terbaik.

Repositori ini menjadi tempat kami mengumpulkan pengetahuan teknis ke dalam skill yang dapat dipakai ulang, mulai dari sistem yang kami kelola saat ini hingga kebutuhan baru yang muncul seiring waktu. Koleksinya akan terus bertambah.

- **Terbuka** - siapa pun boleh mempelajari, menyalin, atau memakainya.
- **Berbasis pengalaman** - disusun dari praktik dan sistem nyata yang kami tangani, bukan tebakan model.
- **Bertumbuh** - skill ditambahkan sesuai kebutuhan pengembangan, tidak terbatas pada satu sistem.

## Cara Menggunakan

Setiap folder di `skills/` adalah satu skill (berkas `SKILL.md` + berkas pendukung). Salin folder skill yang diinginkan ke direktori skill agen Anda, lalu agen akan memuatnya otomatis saat pekerjaan relevan terdeteksi.

### Claude Code

- Salin folder skill ke salah satu lokasi:
  - Global (semua proyek): `~/.claude/skills/<nama-skill>/`
  - Per-proyek: `.claude/skills/<nama-skill>/`
- Skill aktif otomatis saat relevan, atau panggil manual dengan `/<nama-skill>`.

### Codex

- Salin folder skill ke `~/.codex/skills/<nama-skill>/`, lalu mulai ulang Codex agar skill terdeteksi.
- Codex membaca `name` dan `description` dari `SKILL.md` dan memuat skill saat dibutuhkan.

> Pastikan seluruh isi folder (`SKILL.md`, `references/`, `assets/`) ikut tersalin — di situlah detail rujukannya berada.
