<img width="908" height="224" alt="image" src="https://github.com/user-attachments/assets/8fe6a4f8-274b-4d0c-ae6f-96c0a8785f95" />
Scanning
scan the target
<img width="342" height="83" alt="image" src="https://github.com/user-attachments/assets/a885dc50-13ab-411d-aa9f-4f4409062079" />
```nmap -sS -sV 10.10.225.145```
<img width="415" height="70" alt="image" src="https://github.com/user-attachments/assets/e35af6d2-892f-45eb-adc4-e1fe65f3e2e8" />
HTTP
go to the webpage, we can see a corridor with a lot of door can open
<img width="951" height="580" alt="image" src="https://github.com/user-attachments/assets/a3cc5328-c3e1-495a-9e4e-4f6e0feda073" />

when view source, i can see a lot of hash value on each door

<img width="1202" height="521" alt="image" src="https://github.com/user-attachments/assets/da026b96-62c2-44c5-ad2d-07dd8d5627c0" />

Enumeration
now, i will colect all these hashes to further research

```curl http://10.48.147.213 | grep 'alt' | cut -d '"' -f4 > hash.txt```

Cracking
crack the hash with john the ripper

```john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt```
Clearly, all hashed URL endpoints are numbers from 1 to 13.
Exploitation
let's think about IDOR vulnerability, i will change the hashed URL to over the zone, maybe 14 or 0

```echo -n 14 | md5sum```
```echo -n 0 | md5sum```
<img width="369" height="86" alt="image" src="https://github.com/user-attachments/assets/08f8bc45-d5c1-4569-b61c-57c6608ff4ff" />
nothing at room 14 but there is a flag at room 0
<img width="1128" height="569" alt="image" src="https://github.com/user-attachments/assets/ba54fa4e-a73f-406d-a0e7-9d463b41671f" />
