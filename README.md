# Lapres Modul 2 Jarkom 2020 - T03

**Oleh:**
  * I Made Dindra Setyadharma (05311840000008)
  * Rindi Kartika Sari (05311840000013)

---

## Preface

Sebelum masuk kedalam soal. Kami melakukan konfigurasi terlebih dahulu pada file `topologi.sh` untuk menambahkan server **PROBOLINGGO**.

![image](https://user-images.githubusercontent.com/17781660/99147909-7e081500-26bf-11eb-9d93-81580b9ffa44.png)


Pada file `bye.sh` juga ditambahkan untuk server **PROBOLINGGO**.

![image](https://user-images.githubusercontent.com/17781660/99147972-f53da900-26bf-11eb-8fe6-5150dbdbb501.png)

Lalu sesuai pada modul pengenalan UML, pada router **SURABAYA**, IP Forwardingnya diaktifkan pada `/etc/sysctl.conf` dan command `sysctl -p` untuk mengaktifkan perubahannya.

![image](https://user-images.githubusercontent.com/17781660/99148004-39c94480-26c0-11eb-9e47-8a26e0f60be9.png)


Lalu setiap UML disetting IPnya pada file `/etc/network/interfaces` sebagai berikut:

**SURABAYA (Sebagai Router)**

![image](https://user-images.githubusercontent.com/17781660/99148031-74cb7800-26c0-11eb-93fd-dc163664eec7.png)
![image](https://user-images.githubusercontent.com/17781660/99148044-801ea380-26c0-11eb-8b5f-902a73ae5b53.png)

**MALANG (Sebagai DNS Master Server)**

![image](https://user-images.githubusercontent.com/17781660/99148051-9298dd00-26c0-11eb-9c10-445937eb55fd.png)

**MOJOKERTO (Sebagai DNS Slave Server)**

![image](https://user-images.githubusercontent.com/17781660/99148079-ac3a2480-26c0-11eb-93c6-16af7b27d305.png)

**PROBOLINGGO (Sebagai Web Server)**

![image](https://user-images.githubusercontent.com/17781660/99148090-c07e2180-26c0-11eb-9162-13bfe0b26aa3.png)

**SIDOARJO (Sebagai Klien)**

![image](https://user-images.githubusercontent.com/17781660/99148098-ce33a700-26c0-11eb-9a74-42c640894c73.png)

**GRESIK (Sebagai Klien)**

![image](https://user-images.githubusercontent.com/17781660/99148108-dd1a5980-26c0-11eb-8abb-c9fc54ee27cb.png)

Lalu pada setiap UML settingan IP tersebut diaktifkan menggunakan command `service networking restart`.\
Pada Router **SURABAYA** dijalankan command `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.168.0.0/16` agar dapat terkoneksi dengan internet.\
Lalu pada masing-masing UML, dibuatkan file `proxy.sh` untuk memudahkan export proxy setiap kali menjalankan UML. File tersebut dapat dijalankan dengan perintah `source proxy.sh`

![image](https://user-images.githubusercontent.com/17781660/99148122-f3281a00-26c0-11eb-95b3-502b84cb653a.png)

Lalu pada server **MALANG** dan **MOJOKERTO**, install aplikasi bind9 menggunakan perintah:

```
apt-get install bind9 -y
```

Pada server **PROBOLINGGO**, install aplikasi apache2 menggunakan perintah:

```
apt-get install apache2 -y
```

## Soal 1

Konfigurasikan DNS pada server **MALANG** agar domain `http://semerut03.pw` mengarah ke IP server **PROBOLINGGO**. 

**Solusi**\
Pada server **MALANG**, pertama kita akan mengedit file `/etc/bind/named.conf.local` untuk menambahkan zone `semerut03.pw`, dengan syntax berikut:

![image](https://user-images.githubusercontent.com/17781660/99148183-5619b100-26c1-11eb-855b-ebabd8f0fe9b.png)

lalu dibuat folder **jarkom** di `/etc/bind`. Lalu copy `/etc/bind/db.local` menjadi `/etc/bind/jarkom/semerut03.pw`. Lalu konfigurasi file tersebut agar memiliki SOA `semerut03.pw.`, NS `semerut03.pw.`, dan record A yang mengarah ke IP **PROBOLINGGO**.

![image](https://user-images.githubusercontent.com/17781660/99148223-83665f00-26c1-11eb-9a95-6148749aead9.png)

## Soal 2

Konfigurasikan DNS pada server **MALANG** agar domain `http://semerut03.pw` memiliki alias `http://www.semerut03.pw`. 

**Solusi**\
Untuk membuat alias pada domain tersebut, tinggal menambahkan record CNAME pada `/etc/bind/jarkom/semerut03.pw`

![image](https://user-images.githubusercontent.com/17781660/99148250-a42eb480-26c1-11eb-81f1-06b8d8a57dec.png)

## Soal 3

Konfigurasikan DNS pada server **MALANG** agar domain `http://semerut03.pw` memiliki subdomain `http://penanjakan.semerut03.pw`. 

**Solusi**\
Untuk membuat subdomain tersebut, menambahkan record A pada `/etc/bind/jarkom/semerut03.pw` dengan subdomain `penanjakan`.

![image](https://user-images.githubusercontent.com/17781660/99148271-be689280-26c1-11eb-8d25-45166d8a0c4a.png)

## Soal 4

Konfigurasikan DNS pada server **MALANG** agar domain `http://semerut03.pw` memiliki reverse domain ke IP **PROBOLINGGO**.

**Solusi**\
Untuk membuat reverse domain, pertama kita harus menambahkan zone `73.151.10.in-addr.arpa` pada `/etc/bind/named.conf.local`.

![image](https://user-images.githubusercontent.com/17781660/99148312-fc65b680-26c1-11eb-852b-c4b6bc235571.png)

Lalu kita copy file `/etc/bind/db.local` lagi menjadi `/etc/bind/jarkom/73.151.10.in-addr.arpa`. File tersebut dikonfigurasikan agar memiliki record PTR pada IP **PROBOLINGGO** yang mengarah ke domain `semerut03.pw.`.

![image](https://user-images.githubusercontent.com/17781660/99148341-215a2980-26c2-11eb-8249-7e13fd236d18.png)

## Soal 5

Buatkan DNS Server Slave pada **MOJOKERTO** sebagai slave dari DNS **MALANG**.

**Solusi**\
Pertama, kita setting ulang zone `semerut03.pw` pada file `/etc/bind/named.conf.local` pada server **MALANG**, agar menjadi DNS master dan mengarahkan DNS slave ke IP **MOJOKERTO**.

![image](https://user-images.githubusercontent.com/17781660/99148381-4cdd1400-26c2-11eb-90c2-26e2ab090b85.png)

Lalu pada server **MOJOKERTO**, setting zone `semerut03.pw` agar menjadi DNS slave dari **MALANG** pada file `/etc/bind/named.conf.local`.

![image](https://user-images.githubusercontent.com/17781660/99148422-7e55df80-26c2-11eb-90dd-d2cf96e013f5.png)

## Soal 6

Buatkan subdomain `http://gunung.semerut03.pw` pada **MALANG** dan didelegasikan pada server **MOJOKERTO** dan mengarah ke IP **PROBOLINGGO**.

**Solusi**\
Pertama kita edit record `/etc/bind/jarkom/semerut03.pw` agar menambahkan record A yang menuju ke IP **MOJOKERTO** dengan nama `ns1`, dan menambahkan record NS menuju `ns1` dengan nama `gunung`.

![image](https://user-images.githubusercontent.com/17781660/99148443-9fb6cb80-26c2-11eb-9ab3-1272cdf13a9f.png)

pada zone `semerut03.pw` di server **MALANG** juga ditambahkan line `allow-transfer { 'IP MOJOKERTO'; };`

![image](https://user-images.githubusercontent.com/17781660/99148476-d4c31e00-26c2-11eb-835b-b51f2420b01b.png)

Lalu untuk server **MALANG** dan **MOJOKERTO**, pada file `/etc/bind/named.conf.options` comment bagian `dnssec-validation auto;` dan menambahkan line `allow-query{any;};`.

![image](https://user-images.githubusercontent.com/17781660/99148735-79922b00-26c4-11eb-93e6-6e86c8c23ab3.png)

Pada server **MOJOKERTO** dibuat zone untuk `gunung.semerut03.pw` pada `/etc/bind/named.conf.local`. 

![image](https://user-images.githubusercontent.com/17781660/99148802-ca098880-26c4-11eb-9e6d-f7dda46ecb8e.png)

Lalu dibuatkan konfigurasi DNSnya pada file `/etc/bind/jarkom/gunung.semerut03.pw` yang mengarah ke IP **PROBOLINGGO**.

![image](https://user-images.githubusercontent.com/17781660/99148828-ead1de00-26c4-11eb-81ef-ae00e8b9877d.png)

## Soal 7

Buatkan subdomain `http://naik.gunung.semerut03.pw` yang mengarah ke IP **PROBOLINGGO**

**Solusi**\
Pada file `/etc/bind/delegasi/gunung.semerut03.pw` ditambahkan record A untuk menambahkan subdomain `naik.gunung.semerut03.pw`

![image](https://user-images.githubusercontent.com/17781660/99148838-fde4ae00-26c4-11eb-8013-e9e3ac426e2b.png)

## Soal 8

Konfigurasikan web server **PROBOLINGGO** agar dapat mengakses `http://semerut03.pw`.

**Solusi**\
Pada web server **PROBOLINGGO** dibuatkan konfigurasi virtual host dengan mengcopy file `/etc/apache2/sites-available/default` menjadi `/etc/apache2/sites-available/semerut03.pw`. Lalu akan disetting agar memiliki line `ServerName semerut03.pw`, `ServerAlias www.semerut03.pw`, `DocumentRoot /var/www/semerut03.pw`, dan tag `<Directory>` mengarah ke `/var/www/semerut03.pw`.

![image](https://user-images.githubusercontent.com/17781660/99148866-397f7800-26c5-11eb-931f-8663b8165dab.png)

Lalu web yang telah didownload menggunakan `wget` di move ke `/var/www/semerut03.pw`, dan site diaktifkan menggunakan command `a2ensite semerut03.pw`. Website sudah dapat dibuka.

![image](https://user-images.githubusercontent.com/17781660/99148883-59af3700-26c5-11eb-8b85-d36ad9668d06.png)

## Soal 9

Aktifkan `mod_rewrite` agar url `http://semerut03.pw/index.php/home` menjadi `http://semerut03.pw/home`.

**Solusi**\
Agar modul tersebut dapat bekerja, pertama diaktifkan menggunakan command `a2enmod rewrite`. Lalu pada konfigurasi virtual host `/etc/apache2/sites-available/semerut03.pw`, tag `<Directory>` yang mengarah ke `/var/www/semerut03.pw` diedit line `AllowOverride` menjadi `All`.

![image](https://user-images.githubusercontent.com/17781660/99148909-911de380-26c5-11eb-8d6c-3522ecfae3df.png)

Lalu kita edit file `/var/www/semerut03.pw/.htaccess` menjadi seperti berikut agar akses `http://semerut03.pw/home` akan di-*rewrite* menjadi `http://semerut03.pw/index.php/home`.

![image](https://user-images.githubusercontent.com/17781660/99148934-bb6fa100-26c5-11eb-9ae3-03bf0cbc3425.png)
![image](https://user-images.githubusercontent.com/17781660/99148947-ccb8ad80-26c5-11eb-949e-5e01ca4a6535.png)

## Soal 10

Konfigurasikan web server **PROBOLINGGO** agar dapat mengakses `http://penanjakan.semerut03.pw`.

**Solusi**\
Sama seperti soal 8, file `/etc/apache2/sites-available/default` dicopy menjadi `/etc/apache2/sites-available/penanjakan.semerut03.pw`. Lalu disetting agar memiliki line `ServerName penanjakan.semerut03.pw`, `DocumentRoot /var/www/penanjakan.semerut03.pw`, dan tag `<Directory>` mengarah ke `/var/www/penanjakan.semerut03.pw`.

![image](https://user-images.githubusercontent.com/17781660/99149030-7bf58480-26c6-11eb-86db-fb4e4355e9ed.png)

Lalu web yang telah didownload menggunakan `wget` di move ke `/var/www/penanjakan.semerut03.pw`, dan site diaktifkan menggunakan command `a2ensite penanjakan.semerut03.pw`. Website sudah dapat dibuka.

![image](https://user-images.githubusercontent.com/17781660/99149049-9cbdda00-26c6-11eb-8988-ce128e3dfa0e.png)

## Soal 11

Konfigurasikan website `http://penanjakan.semerut03.pw` agar directory `/public` dibolehkan untuk dilisting, tetapi untuk subfoldernya tidak diperbolehkan.

**Solusi**\
Pada file `/etc/apache2/sites-available/penanjakan.semerut03.pw` ditambahkan tag `<Directory>` untuk folder `/public` dan setiap sub-foldernya. Untuk `/public` ditambahkan `Options +Indexes` sementara untuk subfoldernya ditambahakan `Options -Indexes`.

![image](https://user-images.githubusercontent.com/17781660/99149060-be1ec600-26c6-11eb-9e6c-223e2f85af90.png)

## Soal 12

Konfigurasikan website `http://penanjakan.semerut03.pw` agar mengganti HTTP Error code 404 menjadi file `/errors/404.html`.

**Solusi**\
Pada file `/etc/apache2/sites-available/penanjakan.semerut03.pw` ditambahkan line `ErrorDocument 404 /errors/404.html` agar tiap akses yang filenya tidak ketemu akan mengarah langsung ke error page yang telah disediakan.

![image](https://user-images.githubusercontent.com/17781660/99149072-da226780-26c6-11eb-816e-ea5fdf8f25ff.png)
![image](https://user-images.githubusercontent.com/17781660/99149087-e7d7ed00-26c6-11eb-9828-d4fab68f74a4.png)

## Soal 13

Konfigurasikan directory alias pada `http://penanjakan.semerut03.pw/public/javascripts` menjadi `http://penanjakan.semerut03.pw/js`

**Solusi**\
Untuk membuat directory alias, tinggal menambahkan `Alias "/js" "/var/www/penanjakan.semerut03.pw/public/javascripts` pada virtual host `/etc/apache2/sites-available/penanjakan.semerut03.pw`.

![image](https://user-images.githubusercontent.com/17781660/99149102-fe7e4400-26c6-11eb-9704-44a108b4052a.png)
![image](https://user-images.githubusercontent.com/17781660/99149119-0d64f680-26c7-11eb-8b10-f695986e0379.png)

*Note: forbidden karena akses ke /public/javascripts tidak dibolehkan pada soal sebelumnya. Jika tidak berhasil dialiaskan, seharusnya akan memunculkan halaman 404.html*

## Soal 14

Konfigurasikan web server **PROBOLINGGO** agar dapat mengakses `http://naik.gunung.semerut03.pw` pada port 8888.

**Solusi**\
Sama seperti soal sebelumnya, file `/etc/apache2/sites-available/default` dicopy menjadi `/etc/apache2/sites-available/naik.gunung.semerut03.pw`. Tag `<VirtualHost>` diedit agar memiliki port `8888`. Lalu disetting agar memiliki line `ServerName naik.gunung.semerut03.pw`, `DocumentRoot /var/www/naik.gunung.semerut03.pw`, dan tag `<Directory>` mengarah ke `/var/www/naik.gunung.semerut03.pw`.

![image](https://user-images.githubusercontent.com/17781660/99149138-31283c80-26c7-11eb-8185-25f4d35336ec.png)

Lalu pada file `/etc/apache2/ports.conf`, ditambahkan line `Listen 8888` agar port 8888 didengar oleh web server.

![image](https://user-images.githubusercontent.com/17781660/99149156-443b0c80-26c7-11eb-852a-949f376c3a8e.png)

Lalu web yang telah didownload menggunakan `wget` di move ke `/var/www/naik.gunung.semerut03.pw`, dan site diaktifkan menggunakan command `a2ensite naik.gunung.semerut03.pw`. Website sudah dapat dibuka.

![image](https://user-images.githubusercontent.com/17781660/99149175-5ddc5400-26c7-11eb-8c72-7d687cd86409.png)

## Soal 15

Buatkan autentikasi password ketika mengakses web `http://naik.gunung.semerut03.pw` dengan username "**semeru**" dan password "**kuynaikgunung**".

**Solusi**\
Pertama, jalankan command `htpasswd /etc/apache2/.htpasswd semeru` untuk membuat file yang menyimpan username dan password kedalam file `/etc/apache2/.htpasswd` dengan user `semeru`, lalu akan ada prompt untuk memasukkan passwordnya.

![image](https://user-images.githubusercontent.com/17781660/99149196-7b112280-26c7-11eb-887a-c880f5b430b0.png)

Pada file virtual host `/etc/apache2/sites-available/naik.gunung.semerut03.pw`, tag `<Directory>` yang mengarah ke `/var/www/naik.gunung.semerut03.pw` diedit agar memiliki line `AllowOverride All`.

![image](https://user-images.githubusercontent.com/17781660/99149214-9bd97800-26c7-11eb-86c1-ed765758b760.png)

Lalu kita edit file `/var/www/naik.gunung.semerut03.pw/.htaccess` agar memiliki autentikasi basic dan memiliki konfigurasi username dan password sesuai dengan `/etc/apache2/.htpasswd`.

![image](https://user-images.githubusercontent.com/17781660/99149226-b14ea200-26c7-11eb-8fe8-c16b97d75fb0.png)

Website dapat dibuka dan sekarang akan diminta username dan password.

![image](https://user-images.githubusercontent.com/17781660/99149247-c0cdeb00-26c7-11eb-8883-40342fcae7d3.png)

## Soal 16

Konfigurasikan agar laman default Apache agar dialihkan secara otomatis ke `http://semerut03.pw`.

**Solusi**\
Pada `/etc/apache2/sites-available/default`, ditambahkan line `Redirect 301 / http://semerut03.pw` agar setiap kali ada yang mengakses IP **PROBOLINGGO** akan langsung diredirect ke halaman site `http://semerut03.pw`.

![image](https://user-images.githubusercontent.com/17781660/99149273-d9d69c00-26c7-11eb-8074-13e1aa23e87e.png)

## Soal 17

Konfigurasikan agar semua request pada `http://penanjakan.semerut03.pw/` yang memiliki substring "semeru" akan diarahkan menuju `/public/images/semeru.jpg`.

**Solusi**\
Pertama, aktifkan bagian `AllowOverride All` pada konfigurasi virtual host `/etc/apache2/sites-available/penanjakan.semerut03.pw`.

![image](https://user-images.githubusercontent.com/17781660/99149308-202bfb00-26c8-11eb-8723-d980cb9c2ecd.png)

Lalu edit `/var/www/penanjakan.semerut03.pw` agar memiliki `RewriteRule` untuk mengarahkan semua akses yang memiliki substring "semeru" menjadi menuju `/public/images/semeru.jpg`.

![image](https://user-images.githubusercontent.com/17781660/99149317-2e7a1700-26c8-11eb-9755-b0f87ec81f56.png)
![image](https://user-images.githubusercontent.com/17781660/99149325-389c1580-26c8-11eb-8048-1ed0d99b5c18.png)