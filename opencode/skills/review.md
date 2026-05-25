Review kode yang baru diubah atau yang ditentukan user.

Fokus review pada:
1. **Bug dan logic errors** - kesalahan yang bisa menyebabkan runtime error
2. **Security vulnerabilities** - SQL injection, XSS, command injection, dll
3. **Performance issues** - N+1 query, loop yang tidak efisien
4. **Code quality** - duplikasi, nama variabel yang tidak jelas
5. **Adherence to project conventions** - ikuti pola yang sudah ada di codebase

Format output:
- Gunakan severity: CRITICAL, WARNING, INFO
- Sertakan file:line_number untuk setiap issue
- Hanya laporkan issue yang benar-benar penting (high confidence)
- Jangan laporkan style preferences yang tidak ada dampak fungsional

Jika tidak ada issue signifikan, nyatakan "Tidak ada issue kritis ditemukan."
