# Jarkom-Modul-1-E01-2021

- Clarence 05111940000104
- Nur Putra Khanafi 05111940000020
- Husnan 05111940007002

## 1

> EniesLobby akan dijadikan sebagai DNS Master, Water7 akan dijadikan DNS Slave, dan Skypie akan digunakan sebagai Web Server. Terdapat 2 Client yaitu Loguetown, dan Alabasta. Semua node terhubung pada router Foosha, sehingga dapat mengakses internet.

**Foosha untuk node dapat terhubung.**
```shell
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.200.0.0/16
cat /etc/resolv.conf
```

**Loguetown dll**
```shell
echo "nameserver 192.168.122.1" > /etc/resolv.conf
```

![soal1](https://user-images.githubusercontent.com/65794806/139520421-5e4085fe-af51-41c0-9bbd-b6db4e9ee10f.png)
Ping google semua node berhasil.

## 2

> Luffy ingin menghubungi Franky yang berada di EniesLobby dengan denden mushi. Kalian diminta Luffy untuk membuat website utama dengan mengakses franky.yyy.com dengan alias www.franky.yyy.com pada folder kaizoku.

Disini akan dibuat domain untuk franky.e01.com dan www.franky.e01.com.

**EniesLobby, modul1/named.conf.local**
```shell
zone "franky.e01.com" {
    type master;
    file "/etc/bind/kaizoku/franky.e01.com";
    allow-transfer { 192.200.2.3; }; // Masukan IP Water7 tanpa tanda petik
    // bikin jadi slave
    // also-notify { 192.200.2.3; };
    // notify yes;
};

zone "2.200.192.in-addr.arpa" {
    type master;
    file "/etc/bind/kaizoku/2.200.192.in-addr.arpa";
};

// ini zone in addr arpa sebenernya belom kepake, nanti dipake di nomor 4
```
`cp modul1/named.conf.local /etc/bind/named.conf.local`

**cp modul1/named.conf.local /etc/bind/named.conf.local**
```shell
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky.e01.com. root.franky.e01.com. (
                     2021100401         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      franky.e01.com.
@       IN      A       192.200.2.2
www     IN      CNAME   franky.e01.com.
```
`cp modul1/franky.e01.com /etc/bind/kaizoku/franky.e01.com`

![soal2](https://user-images.githubusercontent.com/65794806/139520503-86673686-3c34-4dff-bd4d-2c0547b22a19.png)

## 3

> Setelah itu buat subdomain super.franky.yyy.com dengan alias www.super.franky.yyy.com yang diatur DNS nya di EniesLobby dan mengarah ke Skypie.

Pada franky kita cukup menambahkan super.

**EniesLobby modul1/franky.e01.com**
```shell
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky.e01.com. root.franky.e01.com. (
                     2021100401         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      franky.e01.com.
@       IN      A       192.200.2.2
www     IN      CNAME   franky.e01.com.
super   IN      A       192.200.2.4
www.super IN    CNAME   super.franky.e01.com.
```
Kemudian menambahkan `ping super.franky.e01.com` dan `ping www.super.franky.e01.com` pada 2 line paling bawah. Hasilnya sebagai berikut.

![soal3](https://user-images.githubusercontent.com/65794806/139520648-e5e6b609-2937-43e9-831e-c7785aaa0d67.png)

## 4

> Buat juga reverse domain untuk domain utama.

Karena pada nomor 2 telah dideclare di named.conf.local, maka tinggal menambahkan config addr arpanya.

**EniesLobby modul1/2.200.192.in-addr.arpa**
```shell
;
; BIND data file for local loopback interface
;
$TTL	604800
@	IN	SOA	franky.e01.com. root.franky.e01.com. (
			2021100401	; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
2.200.192.in-addr.arpa. IN	NS	franky.e01.com.
2 			IN	PTR	franky.e01.com.

```
`cp modul1/2.200.192.in-addr.arpa /etc/bind/kaizoku/2.200.192.in-addr.arpa`

Cek: `host -t PTR 192.200.2.2`

![soal4](https://user-images.githubusercontent.com/65794806/139520789-af2d3fd3-6382-4c72-b9e6-193fa285b511.png)

## 5

> Supaya tetap bisa menghubungi Franky jika server EniesLobby rusak, maka buat Water7 sebagai DNS Slave untuk domain utama.

Menambahkan Notify

**EniesLobby /etc/bind/named.conf.local**
```shell
zone "franky.e01.com" {
    type master;
    file "/etc/bind/kaizoku/franky.e01.com";
    allow-transfer { 192.200.2.3; }; // Masukan IP Water7 tanpa tanda petik
    // bikin jadi slave
    also-notify { 192.200.2.3; };
    notify yes;
};

zone "2.200.192.in-addr.arpa" {
    type master;
    file "/etc/bind/kaizoku/2.200.192.in-addr.arpa";
};
```
Menyalakan notify

**Water7 run script.sh**
```shell
echo "nameserver 192.168.122.1" > /etc/resolv.conf

apt-get update
apt-get install bind9 -y
echo "# nameserver 192.168.122.1
nameserver 192.200.2.2
nameserver 192.200.2.3" > /etc/resolv.conf
```

**Water7 modul1/named.conf.local**
```shell
// SLAVE
zone "franky.e01.com" {
    type slave;
    masters { 192.200.2.2; }; // Masukan IP EniesLobby tanpa tanda petik
    file "/var/lib/bind/franky.e01..com";
};
```
`cp modul1/named.conf.local /etc/bind/named.conf.local`

![soal5 1](https://user-images.githubusercontent.com/65794806/139520902-586295cf-99a4-4017-a2eb-6f0a738a8622.png)

![soal5 2](https://user-images.githubusercontent.com/65794806/139520904-b9f09b19-3345-4402-a955-2d6446a65c04.png)

## 6

> Setelah itu terdapat subdomain mecha.franky.yyy.com dengan alias www.mecha.franky.yyy.com yang didelegasikan dari EniesLobby ke Water7 dengan IP menuju ke Skypie dalam folder sunnygo.

Pada soal ini melakukan delegasi domain.

**Enieslobby /etc/bind/kaizoku/franky.e01.com.**
```shell
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky.e01.com. root.franky.e01.com. (
                     2021100401         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      franky.e01.com.
@       IN      A       192.200.2.2
www     IN      CNAME   franky.e01.com.
super   IN      A       192.200.2.4
www.super IN    CNAME   super.franky.e01.com.
mecha   IN      NS      ns1
```
Kemudian, melakukan pengeditan pada named.conf.options dnnsec, dan menambahkan allow-query{any;};

**Water7 /etc/bind/named.conf.local**
```shell
// SLAVE
zone "franky.e01.com" {
    type slave;
    masters { 192.200.2.2; }; // Masukan IP EniesLobby tanpa tanda petik
    file "/var/lib/bind/franky.e01.com";
};

// DELEGASI
zone "mecha.franky.e01.com" {
    type master;
    file "/etc/bind/sunnygo/mecha.franky.e01.com"
};
```
Edit named.conf.options dnnsec dan menambahkan allow-query{any;};. Selain itu Delegasi juga ditambahkan.

**Water7 /etc/bind/sunnygo/mecha.franky.e01.com**
```shell
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     mecha.franky.e01.com. root.mecha.franky.e01.com. (
                              2021100401                ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      mecha.franky.e01.com.
@       IN      A       192.200.2.4
www     IN      CNAME   mecha.franky.e01.com.
```
Menambahkan Aliasnya.

`ping mecha.franky.e01.com`
`ping www.mecha.franky.e01.com  `

![soal6](https://user-images.githubusercontent.com/65794806/139521039-0ec050f5-7fa7-4b9b-838d-35994e37ca79.png)

## 7

> Untuk memperlancar komunikasi Luffy dan rekannya, dibuatkan subdomain melalui Franky dengan nama general.mecha.frank.yyy.com dengan alias www.general.mecha.franky.yyy.com yang mengarah ke Skypie.

Membuat sub-domain kemudian menambahkan `alias general.mecha.franky.e01.com.`.

**Water7 /etc/bind/sunnygo/mecha.franky.e01.com**
```shell
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     mecha.franky.e01.com. root.mecha.franky.e01.com. (
                              2021100401                ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      mecha.franky.e01.com.
@       IN      A       192.200.2.4
www     IN      CNAME   mecha.franky.e01.com.
general IN      A       192.200.2.4
www.general IN  CNAME   general.mecha.franky.e01.com.
```

`ping general.mecha.franky.e01.com`
`ping www.general.mecha.franky.e01.com`

![soal7](https://user-images.githubusercontent.com/65794806/139521280-b16ba373-2efb-4c3e-ac08-8f05f8d9594a.png)

## 8

> Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver. Pertama dengan webserver www.franky.yyy.com. Pertama, luffy membutuhkan webserver dengan DocumentRoot pada /var/www/franky.yyy.com

Karena menggunakan web server, maka Config EniesLobby terlebih dahulu diarahkan ke Skypie.

**EniesLobby modul1/webserver/franky.e01.com**
```shell
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky.e01.com. root.franky.e01.com. (
                        4               ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      franky.e01.com.
@       IN      A       192.200.2.4
www     IN      CNAME   franky.e01.com.
super   IN      A       192.200.2.4
www.super IN    CNAME   super.franky.e01.com.
mecha   IN      NS      ns1
```

**EniesLobby modul1/webserver/2.200.192.in-addr-arpa**
```shell
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky.e01.com. root.franky.e01.com. (
                        2               ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
2.200.192.in-addr.arpa. IN      NS      franky.e01.com.
2                       IN      PTR     franky.e01.com.
```

Install apache2 & php di water7 dan memasukkan dokumen htmlnya.
**Skypie modul1/franky.e01.com**
```bash
<VirtualHost *:80>
  ...
	ServerName franky.e01.com
  ServerAlias www.franky.e01.com

	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/franky.e01.com
  
  ...

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

  ...
</VirtualHost>

```
`lynx http://www.franky.e01.com`

![soal8](https://user-images.githubusercontent.com/65794806/139521462-0b028254-894d-4d1b-9735-a03bf5e30126.png)

## 9

> Setelah itu, Luffy juga membutuhkan agar url www.franky.yyy.com/index.php/home dapat menjadi menjadi www.franky.yyy.com/home.

Membuat alias dari home yang akan mengarah ke index.php/home

**Skypie modul1/franky.e01.com**
```bash
<VirtualHost *:80>
        ...
        ServerName franky.e01.com
        ServerAlias www.franky.e01.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/franky.e01.com

        Alias "/home" "/var/www/franky.e01.com/index.php/home"

        ...

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        ...
</VirtualHost>

```
`lynx http://franky.e01.com/home`

![soal9](https://user-images.githubusercontent.com/65794806/139521813-929fd2fe-c606-4c3c-84ad-8fbd43353eb9.png)

## 10

> Setelah itu, pada subdomain www.super.franky.yyy.com, Luffy membutuhkan penyimpanan aset yang memiliki DocumentRoot pada /var/www/super.franky.yyy.com.

**Skypie /etc/apache2/sites-available/super.franky.e01.com.conf**
```bash
<VirtualHost *:80>
        ...
        ServerName super.franky.e01.com
        ServerAlias www.super.franky.e01.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/super.franky.e01.com

        ...

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        ...
</VirtualHost>
```
Clone file github `git config --global http.sslVerify false`

cek lynx ke super.franky.e01.com
`lynx http://www.super.franky.e01.com`

![soal10](https://user-images.githubusercontent.com/65794806/139521940-61d24205-55e8-401f-983c-950d0b4161ae.png)

Bentuknya berupa directory listing karena tidak ada default htmlnya.

## 11

> Akan tetapi, pada folder /public, Luffy ingin hanya dapat melakukan directory listing saja.

Selanjutnya merupakan directory listing untuk public, dengan menambah directory dan option +indexes.

**Skypie super.franky.e01.com.conf**
```bash
<VirtualHost *:80>
        ServerName super.franky.e01.com
        ServerAlias www.super.franky.e01.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/super.franky.e01.com

				<Directory /var/www/super.franky.e01.com>
                Options +Indexes
        </Directory>

				<Directory /var/www/super.franky.e01.com/error>
                Options -Indexes
        </Directory>

        <Directory /var/www/super.franky.e01.com/public>
                Options +Indexes
        </Directory>

        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

`lynx http://www.super.franky.e01.com/public`

![soal11](https://user-images.githubusercontent.com/65794806/139522136-a144deba-0250-4f97-a76d-8e49c774b047.png)

## 12

> Tidak hanya itu, Luffy juga menyiapkan error file 404.html pada folder /errors untuk mengganti error kode pada apache.

Menambahkan penggantian error ke dokumen html pada baris terakhir.

**Skypie superfranky conf**
```bash
<VirtualHost *:80>
        ServerName super.franky.e01.com
        ServerAlias www.super.franky.e01.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/super.franky.e01.com

        <Directory /var/www/super.franky.e01.com>
                Options +Indexes
        </Directory>

        <Directory /var/www/super.franky.e01.com/public>
                Options +Indexes
        </Directory>


        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        ErrorDocument 404 /error/404.html
</VirtualHost>
```

`lynx http://www.super.franky.e01.com/ngasal`

![soal12](https://user-images.githubusercontent.com/65794806/139522257-981de7ab-e35c-4e24-bf23-4ffc5119a704.png)

## 13

> Luffy juga meminta Nami untuk dibuatkan konfigurasi virtual host. Virtual host ini bertujuan untuk dapat mengakses file asset www.super.franky.yyy.com/public/js menjadi www.super.franky.yyy.com/js.

## 14

> Dan Luffy meminta untuk web www.general.mecha.franky.yyy.com hanya bisa diakses dengan port 15000 dan port 15500.

## 15

> Dengan authentikasi username luffy dan password onepiece dan file di /var/www/general.mecha.franky.yyy.

## 16

> Dan setiap kali mengakses IP Skypie akan diahlikan secara otomatis ke www.franky.yyy.com.

## 17

> Dikarenakan Franky juga ingin mengajak temannya untuk dapat menghubunginya melalui website www.super.franky.yyy.com, dan dikarenakan pengunjung web server pasti akan bingung dengan randomnya images yang ada, maka Franky juga meminta untuk mengganti request gambar yang memiliki substring “franky” akan diarahkan menuju franky.png. Maka bantulah Luffy untuk membuat konfigurasi dns dan web server ini!

