Bantu implementasi fitur Laravel baru mengikuti pola yang ada di codebase.

Proses:
1. Eksplorasi struktur project untuk memahami pola yang digunakan (Controllers, Services, Repositories, Requests, Resources)
2. Identifikasi konvensi naming yang digunakan
3. Tanyakan detail fitur jika belum jelas
4. Implementasi dengan urutan:
   a. Migration (jika ada perubahan database)
   b. Model dengan relationships
   c. Repository/Interface jika pola repository digunakan
   d. Service layer untuk business logic
   e. Form Requests untuk validasi
   f. API Resource untuk transformasi response
   g. Controller
   h. Route
   i. Test

Aturan:
- Ikuti pola yang SUDAH ADA di codebase, jangan buat pola baru
- Gunakan `php -l` untuk syntax check setiap file baru
- JANGAN jalankan `vendor/bin/pint`, `bun run build`, atau `bun dev`
- Beritahu user untuk menjalankan migration manual
