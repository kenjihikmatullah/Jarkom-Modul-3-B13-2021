# Jarkom-Modul-3-B13-2021

untuk menjalankan instalasi yang diperlukan

**Foosha**

   ```
   auto eth0
   iface eth0 inet dhcp

   auto eth1
   iface eth1 inet static
      address 192.183.1.1
      netmask 255.255.255.0

   auto eth2
   iface eth2 inet static
      address 192.183.2.1
      netmask 255.255.255.0

   auto eth3
   iface eth3 inet static
      address 192.183.3.1
      netmask 255.255.255.0
   ```

**EniesLobby**

   ```
   auto eth0
   iface eth0 inet static
      address 192.183.2.2
      netmask 255.255.255.0
      gateway 192.183.2.1
   ```

**Water7**

   ```
   auto eth0
   iface eth0 inet static
      address 192.183.2.3
      netmask 255.255.255.0
      gateway 192.183.2.1
   ```

**Jipangu**
   ```
   auto eth0
   iface eth0 inet static
      address 192.183.2.4
      netmask 255.255.255.0
      gateway 192.183.2.1
   ```

Kemudian, jalankan perintah ini pada **Foosha**

```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.183.0.0/16
```

Dan, jalankan perintah ini pada **EniesLobby**, **Water7**, dan **Jipangu**

```
echo nameserver 192.168.122.1 > /etc/resolv.conf
```

## Soal 1

Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server

1. Instalasi ISC-DHCP-Server pada **Jipangu**
   - Update package lists di **Jipangu**
     ```
     apt-get update
     ```
   - Install isc-dhcp-server
     ```
     apt-get install isc-dhcp-server -y
     ```
   - Pastikan isc-dhcp-server telah terinstall dengan perintah
     ```
     dhcpd --version
     ```
2. Konfigurasi DHCP Server pada **Jipangu**
   - edit file konfigurasi isc-dhcp-server
     ```
     nano /etc/default/isc-dhcp-server
     ```
   - interface yang diberikan layanan DHCP adalah **eth0**. Maka, pada bagian baris akhir di file `/etc/default/isc-dhcp-server` diubah menjadi
     ```
     INTERFACES = "eth0"
     ```
3. Konfigurasi Proxy Server pada **Water7**
   - Update package lists di **Water7**
     ```
     apt-get update
     ```
   - install squid
     ```
     apt-get install squid -y
     ```
   - Cek status Squid dan pastikan sudah _running_
     ```
     service squid status
     ```
   - Backup file konfigurasi default yang telah disediakan Squid
     ```
     cp /etc/squid/squid.conf /etc/squid/squid.conf.bak
     ```
   - Buat konfigurasi Squid baru
     ```
     vim /etc/squid/squid.conf
     ```
   - Kemudian, tambahkan
     ```
     http_port 8080
     visible_hostname Water7
     ```
   - Restart squid
     ```
     service squid restart
     ```
     
## Soal 2

Foosha sebagai DHCP Relay

**Pembahasan:**

1.  Instalasi ISC-DHCP-Relay pada **Foosha**

    - Update package lists di **Foosha**
      ```
      apt-get update
      ```
    - Install isc-dhcp-relay
      ```
      apt-get install isc-dhcp-relay
      ```
    - Saat instalasi berlangsung, diminta untuk input **SERVERS** yang diisi dengan IP **Jipangu**
      ```
      192.183.2.4
      ```
    - Kemudian, diminta juga input **INTERFACES** yakni
      ```
      eth1 eth2 eth3
      ```
## Soal 3 - 6

- **No 3**
  Client yang melalui Switch1 mendapatkan range IP dari 192.183.1.20 - 192.183.1.99 dan 192.183.1.150 - 192.183.1.169
- **No 4**
  Client yang melalui Switch3 mendapatkan range IP dari 192.183.3.30 - 192.183.3.50
- **No 5** Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut.
- **No 6** Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit.


1. Install bind9 pada EniesLobby dan jadikan sebagai DNS Forwaders

   - install bind9
     ```
     apt-get install bind9 -y
     ```
   - Edit file /etc/bind/named.conf.options pada server EniesLobby dengan uncomment pada
     ```
     forwarders {
         192.168.122.1;
     };
     ```
   - comment
     ```
     //dnssec-validation auto;
     ```
   - tambahkan
     ```
     allow-query{any;};
     ```

2. Settings DHCP Server pada **Jipangu**

   - edit file pada `/etc/dhcp/dhcpd.conf`
     ```
     nano /etc/dhcp/dhcpd.conf
     ```
   - Tambahkan script berikut

     ```
     subnet 192.183.2.0 netmask 255.255.255.0{
     }

     subnet 192.183.1.0 netmask 255.255.255.0{
         range 192.183.1.20 192.183.1.99;
         range 192.183.1.150 192.183.1.169;
         option routers 192.183.1.1;
         option broadcast-address 192.183.1.255;
         option domain-name-servers 192.183.2.2;
         default-lease-time 360;
         max-lease-time 7200;
     }

     subnet 192.183.3.0 netmask 255.255.255.0{
         range 192.183.3.30 192.183.3.50;
         option routers 192.183.3.1;
         option broadcast-address 192.183.3.255;
         option domain-name-servers 192.183.2.2;
         default-lease-time 720;
         max-lease-time 7200;
     }
     ```
 
## Soal 7
- Luffy dan Zoro berencana menjadikan Skypie sebagai server untuk jual beli kapal yang dimilikinya dengan alamat IP yang tetap dengan IP [prefix IP].3.69

Ubah konfigurasi jaringan pada Skypie dan atur hwaddress-nya:
```
auto eth0
iface eth0 inet dhcp
hwaddress ether 52:21:08:91:8d:49
```

Tambahkabn konfigurasi di Jipangu sebagai DHCP Server agar memberikan IP yang tetap ke Skypie dimana dia memiliki hwaddress sesuai yang sudah diatur di atas 
```
host Skypie {
  hardware ethernet 52:21:08:91:8d:49;
  fixed-address 192.168.3.69;
}
```

## Soal 8 
- Loguetown digunakan sebagai client Proxy agar transaksi jual beli dapat terjamin keamanannya, juga untuk mencegah kebocoran data transaksi.
Pada Loguetown, proxy harus bisa diakses dengan nama `jualbelikapal.b13.com` dengan port yang digunakan adalah `5000`


     **EniesLobby**

    Pertama kita buat dulu domain `jualbelikapal.b13.com` di EniesLobby, buka file `/etc/bind/named.conf.local` dan tambahkan script di bawah ini di dalamnya:
    ```
    zone "jualbelikapal.b13.com" {
            type master;
            file "/etc/bind/jarkom/jualbelikapal.b13.com";
    };
    ```
    selanjutnya buat folder jarkom di dalam /etc/bind/jarkom

    ```
    mkdir /etc/bind/jarkom
    ```

    kemudian Copykan file db.local pada path `/etc/bind` ke dalam folder jarkom yang baru saja dibuat dan ubah namanya menjadi `jualbelikapal.b13.com`

	```
	cp /etc/bind/db.local /etc/bind/jarkom/jualbelikapalb13.com

  ```

Kemudian buka file jualbelikapal.b13.com dan masukkan script di bawah ini:
```

  ; BIND data file for local loopback interface
  ;
  $TTL    604800
  @       IN      SOA     jualbelikapal.b13.com. root.jualbelikapal.b13.com. (
                                2         ; Serial
                          604800         ; Refresh
                            86400         ; Retry
                          2419200         ; Expire
                          604800 )       ; Negative Cache TTL
  ;
  @       IN      NS      jualbelikapal.b13.com.
  @       IN      A       192.183.2.3 ;IP Water7
  ```

IP yang digunakan merupakan IP Water7 yang merupakan proxy server. Selanjutnya restart bind9
```
service bind9 restart
```

IP jualbelikapal.b13.com sudah mengarah ke IP Water7

Water7

jika sudah lakukan konfigurasi squid di node Water7. Backup terlebih dahulu file konfigurasi default yang disediakan Squid.

```
mv /etc/squid/squid.conf /etc/squid/squid.conf.bak
```

kemudain buka file `/etc/squid/squid.conf` dan masukan

```
http_port 5000 
visible_hostname jualbelikapal.b13.com
lalu save dan restart squid

service squid restart
Loguetown
```

Karena Loguetown digunakan sebagai proxy client, kita set proxy di node Loguetown

export http_proxy="http://jualbelikapal.b13.com:5000"

## Soal 9
- Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy dipasang autentikasi user proxy dengan enkripsi MD5 dengan dua username, yaitu luffybelikapalyyy dengan password luffy_yyy dan zorobelikapalyyy dengan password zoro_yyy

Buat file credential dengan perintah berikut
```
htpasswd -cm /etc/squid/passwd luffybelikapalb13
htpasswd -m /etc/squid/passwd zorobelikapalb13
```

Include kedua file tersebut pada squid.conf
```
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Proxy
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED
```


## Soal 10
- Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari Senin-Kamis pukul 07.00-11.00 dan setiap hari Selasa-Jum’at pukul 17.00-03.00 keesokan harinya (sampai Sabtu pukul 03.00)

**Water7**

buat file `acl.conf` didalam folder squid

```
vim /etc/squid/acl.conf
```
dan tambahkan:

```
acl AVAILABLE_WORKING time MTWH 07:00-11:00
acl AVAILABLE_WORKING_2 time TWHF 17:00-23:59
acl AVAILABLE_WORKING_3 time WHFA 00:00-03:00
```
save file tersebut, kemudian edit file `squid.conf` dengan menambahkan:

```
include /etc/squid/acl.conf
http_access allow USERS AVAILABLE_WORKING
http_access allow USERS AVAILABLE_WORKING_2
http_access allow USERS AVAILABLE_WORKING_3
```

save file tersebut dan lakukan restart pada squid.

## Soal 11
- Agar transaksi bisa lebih fokus berjalan, maka dilakukan redirect website agar mudah mengingat website transaksi jual beli kapal. Setiap mengakses google.com, akan diredirect menuju super.franky.yyy.com dengan website yang sama pada soal shift modul 2. Web server super.franky.yyy.com berada pada node Skypie

Buat *acl* agar setiap ada request yang mengarah ke google.com, akan ditolak dan diarahkankan ke super.franky.b13.com
```
acl GOOGLE dstdomain .google.com
http_access deny GOOGLE
deny_info http://super.franky.b13.com/ GOOGLE
```

## Soal 12
-Saatnya berlayar! Luffy dan Zoro akhirnya memutuskan untuk berlayar untuk mencari harta karun di super.franky.yyy.com. Tugas pencarian dibagi menjadi dua misi, Luffy bertugas untuk mendapatkan gambar (.png, .jpg), sedangkan Zoro mendapatkan sisanya. Karena Luffy orangnya sangat teliti untuk mencari harta karun, ketika ia berhasil mendapatkan gambar, ia mendapatkan gambar dan melihatnya dengan kecepatan 10 kbps 

**Water7**

untuk mengatur bandwith buat file `acl-bandwith.conf` pada folder squid

```
vim /etc/squid/acl-bandwith.conf
```

dan ketikkan

```
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd

acl luffy proxy_auth luffybelikapalb13
acl zoro proxy_auth zorobelikapalb13
acl download  url_regex -i \.jpg$ \.png$

delay_pools 2
delay_class 1 1
delay_parameters 1 1250/1250
delay_access 1 allow luffy download
delay_access 1 deny all

http_access allow luffy download
```

delay_pools 2 dua karena untuk luffy dan zoro
delay_parameters 1 1250/1250 set bandwith. karena squid menggunakan byte maka 10 kbps di konversi terlebih dahulu menjadi byte.
delay_access 1 allow luffy download izinkan luffy untuk mendownload img

save file dan edit file konfigurasi squid dengan menambahkan

```
include /etc/squid/acl-bandwidth.conf
```

save file dan lakukan restart pada squid

## Soal 13
- Sedangkan, Zoro yang sangat bersemangat untuk mencari harta karun, sehingga kecepatan kapal Zoro tidak dibatasi ketika sudah mendapatkan harta yang diinginkannya
Sebelumnya bandwidth user Zoro belum pernah dibatasi jadi sudah aman
