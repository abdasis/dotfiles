Buat Pull Request untuk branch saat ini.

Langkah:
1. Jalankan `git status` dan `git log` untuk melihat state branch
2. Jalankan `git diff [base-branch]...HEAD` untuk melihat semua perubahan
3. Analisis SEMUA commit yang akan masuk ke PR (bukan hanya yang terakhir)
4. Draft PR dengan format:

**Title**: Ringkas, max 70 karakter, mulai dengan verb (Add, Fix, Update, Remove)

**Body**:
- Summary: 2-3 bullet points perubahan utama
- Test plan: checklist testing manual yang perlu dilakukan
- Breaking changes: jika ada

5. Gunakan `gh pr create` dengan HEREDOC untuk body
6. Return URL PR setelah berhasil dibuat

PENTING:
- Pastikan branch sudah di-push ke remote sebelum buat PR
- Jangan force push ke main/master
- Konfirmasi ke user sebelum push jika belum ada di remote
