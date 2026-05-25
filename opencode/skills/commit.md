Buat git commit yang baik untuk perubahan saat ini.

Delegasikan SELURUH pekerjaan commit ke `github-assistant` agent menggunakan Agent tool dengan subagent_type `github-assistant`.

Instruksi untuk agent:
- Jalankan `git status` untuk melihat file yang berubah
- Jalankan `git diff --staged` dan `git diff` untuk melihat isi perubahan
- Jalankan `git log --oneline -5` untuk melihat gaya commit sebelumnya
- Analisis semua perubahan
- Stage file yang relevan dengan `git add <file>` (jangan gunakan `git add -A` kecuali diminta)
- Buat commit message yang:
  - Menggunakan format Conventional Commits: `<type>(<scope>): <description>`
  - Type: `feat`, `fix`, `refactor`, `style`, `docs`, `test`, `chore`
  - Ringkas (max 72 karakter untuk baris pertama)
  - Wajib menggunakan bahasa Indonesia
  - Fokus pada apa yang sudah dikerjakan berdasarkan perubahan aktual di kode
  - Jangan merujuk todo, spec, atau rencana — hanya fakta dari diff
  - Tanpa emoji
  - TANPA atribut AI (Co-Authored-By, Generated with, dll)
- Gunakan format HEREDOC untuk commit message
- Jalankan `git status` untuk verifikasi setelah commit

PENTING: Jangan push ke remote kecuali diminta eksplisit.
