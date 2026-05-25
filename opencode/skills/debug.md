Debug masalah yang dilaporkan user secara sistematis.

Langkah debug:
1. Pahami symptom yang dilaporkan
2. Cari error message di log jika ada (storage/logs/laravel.log untuk Laravel)
3. Trace eksekusi kode dari entry point
4. Identifikasi root cause (bukan hanya symptom)
5. Usulkan fix dengan penjelasan mengapa ini adalah root cause
6. Verifikasi fix tidak memperkenalkan masalah baru

Untuk Laravel:
- Cek `storage/logs/laravel.log` untuk error terbaru
- Gunakan `php artisan tinker` untuk test query atau logic
- Cek `php artisan route:list` untuk routing issues

Output:
- Jelaskan root cause dengan jelas
- Tunjukkan baris kode yang bermasalah (file:line)
- Berikan fix yang minimal dan tepat sasaran
- Jangan refactor kode yang tidak berhubungan dengan bug
