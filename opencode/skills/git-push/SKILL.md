---
name: git-push
description: >
  Kelompokkan semua untracked dan modified files menjadi beberapa commit terpisah berdasarkan
  module, jenis file, dan jenis pengerjaan (feat/fix/refactor/style/chore), lalu push ke remote.
  Gunakan skill ini setiap kali user minta "push", "push semua", "commit dan push",
  "kirim ke git", "sync ke remote", atau variasi lainnya yang bermaksud mengirim perubahan
  ke repository remote. Juga aktifkan saat user bilang "grupkan commit", "commit per module",
  atau "commit terpisah lalu push". Skill ini BERBEDA dari skill /commit yang hanya buat
  satu commit tanpa push — skill ini buat BANYAK commit terkelompok lalu push otomatis.
---

# Git Push — Multi-Commit Terkelompok + Auto Push

Skill ini menganalisis semua perubahan (untracked, modified, staged), mengelompokkannya
secara cerdas, membuat commit terpisah per kelompok, lalu push ke remote.

## Prinsip Pengelompokan

Gabungkan tiga dimensi untuk menentukan kelompok:

1. **Module/Folder** — file dalam folder yang sama atau module yang sama jadi satu grup
   - `modules/Attendance/*` = satu grup
   - `modules/WhatsApp/*` = satu grup
   - `resources/js/pages/activity-log/*` = satu grup
   - `app/Models/*` + `app/Http/Controllers/*` yang terkait = bisa digabung jika saling berkaitan

2. **Jenis Pengerjaan** — tentukan dari isi diff, bukan dari asumsi
   - `feat` — file/fungsi baru, fitur baru
   - `fix` — perbaikan bug, koreksi logic
   - `refactor` — restruktur tanpa ubah behavior
   - `style` — perubahan tampilan, CSS, layout
   - `chore` — config, dependency, file pendukung
   - `docs` — dokumentasi
   - `test` — file test

3. **Jenis File** — sebagai tiebreaker jika masih ambigu
   - Backend (PHP): controller, model, service, migration, lang
   - Frontend (TSX/TS): page, component, hook, types
   - Config: .env, config/*.php, vite.config, tailwind.config
   - Assets: gambar, font, file statis

### Aturan Pengelompokan

- File yang **secara logis terkait** harus satu commit meskipun beda folder.
  Contoh: controller + migration + model untuk fitur yang sama = satu commit `feat`.
- File yang **beda module** WAJIB beda commit, kecuali perubahan cross-cutting
  (misal: shared component yang dipakai banyak module).
- Jika satu module punya campuran feat + fix, pisahkan jadi 2 commit.
- Maksimal ~15 file per commit. Jika lebih, pecah lagi.
- Jika hanya ada 1-3 file terkait, boleh jadi 1 commit kecil.

## Langkah Eksekusi

### 1. Analisis Perubahan

```bash
git status
git diff --stat
git diff --cached --stat
git log --oneline -5
```

Baca output, identifikasi semua file yang berubah/baru.

### 2. Baca Diff untuk Konteks

Untuk setiap file yang berubah, baca diff-nya untuk memahami **apa yang dikerjakan**:

```bash
git diff -- <file>
git diff --cached -- <file>
```

Untuk untracked files, baca isi file untuk memahami purpose-nya.

### 3. Rancang Kelompok Commit

Buat rencana pengelompokan. Tampilkan rencana ke user, lalu **LANGSUNG eksekusi commit dan push tanpa menunggu konfirmasi**.

Format tampilan rencana:

```
Rencana commit:

1. feat(attendance): tambah komponen report table dan skeleton
   - modules/Attendance/resources/js/pages/attendance/components/report-table.tsx
   - modules/Attendance/resources/js/pages/attendance/components/report-table-skeleton.tsx

2. feat(whatsapp): tambah komponen dashboard dan template
   - modules/WhatsApp/resources/js/pages/whatsapp/dashboard/components/delivery-overview.tsx
   - modules/WhatsApp/resources/js/pages/whatsapp/dashboard/components/stats-cards.tsx
```

**JANGAN** tampilkan "Lanjut push? (y/n)" atau konfirmasi apapun. Langsung eksekusi setelah menampilkan rencana.

### 4. Eksekusi Commit

Untuk setiap kelompok, jalankan secara berurutan:

```bash
git add <file1> <file2> ...
git commit -m "$(cat <<'EOF'
<type>(<scope>): <deskripsi bahasa Indonesia>
EOF
)"
```

### 5. Push ke Remote

Setelah semua commit selesai:

```bash
git push
```

Jika branch belum punya upstream:

```bash
git push -u origin <branch-name>
```

### 6. Verifikasi

```bash
git status
git log --oneline -<jumlah-commit-baru>
```

Tampilkan ringkasan: berapa commit dibuat, sudah di-push ke branch apa.

## Format Commit Message

- Bahasa Indonesia WAJIB
- Format: `<type>(<scope>): <deskripsi>`
- Scope = nama module/fitur dalam lowercase (attendance, whatsapp, billing, dashboard, dll)
- Deskripsi singkat, max 72 karakter, lowercase setelah colon
- Fokus pada APA yang dikerjakan berdasarkan diff aktual
- TANPA emoji
- TANPA atribut AI (Co-Authored-By, Generated with Claude, dll)
- TANPA referensi task, todo, spec, atau rencana
- Gunakan HEREDOC untuk commit message

### Contoh Commit Message yang Benar

```
feat(attendance): tambah komponen report table dan skeleton
fix(billing): perbaiki kalkulasi diskon pada invoice
refactor(whatsapp): pisahkan template form ke komponen terpisah
style(dashboard): perbaiki spacing dan alignment stat cards
chore(config): update konfigurasi queue untuk billing module
```

### Contoh yang SALAH (Dilarang)

```
feat: add attendance components          # bahasa Inggris
feat(attendance): Tambah Report Table    # huruf besar
feat(attendance): tambah report table 🎉  # emoji
Update files                              # tidak deskriptif
```

## Situasi Khusus

- **Tidak ada perubahan**: beritahu user, jangan buat commit kosong.
- **Ada file sensitif** (.env, credentials, secrets): JANGAN commit. Peringatkan user.
- **Conflict dengan remote**: beritahu user untuk resolve manual dulu.
- **Branch protected**: beritahu user, sarankan buat branch baru.

## Yang TIDAK Boleh Dilakukan

- Jangan jalankan `bun dev`, `bun run build`, `vendor/bin/pint`, atau build command lainnya
- Jangan `git add -A` atau `git add .` — selalu add file spesifik
- Jangan force push (`--force` atau `--force-with-lease`) kecuali user minta eksplisit
- Jangan amend commit yang sudah ada
- Jangan skip hooks (`--no-verify`)
