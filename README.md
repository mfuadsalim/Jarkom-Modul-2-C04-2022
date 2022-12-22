# Jarkom-Modul-2-C04-2022

Laporan Resmi Praktikum Modul 4 Kelompok C04

## Kelompok C04

| **Nama**                  | **NRP**    |
| ------------------------- | ---------- |
| Muhammad Fuad Salim | 5025201057 |
| Handitanto Herprasetyo | 5025201077 |
| Sastiara Maulikh | 5025201257 |

IP Prefix Kelompok C04 : `192.181`

## Soal Praktikum Modul 4
# Topologi

# Nomor 1
WISE akan dijadikan sebagai DNS Master, Berlint akan dijadikan DNS Slave, dan Eden akan digunakan sebagai Web Server. Terdapat 2 Client yaitu SSS, dan Garden. Semua node terhubung pada router Ostania, sehingga dapat mengakses internet 

Konfigurasi sesuai modul GNS. Menggunakan IP Prefix 192.181 -> IP Kelompok C04.

### Konfigurasi Ostania

```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 192.181.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 192.181.2.1
	netmask 255.255.255.0

auto eth3
iface eth3 inet static
	address 192.181.3.1
	netmask 255.255.255.0
```

### Konfigurasi SSS

```
auto eth0
iface eth0 inet static
	address 192.181.1.2
	netmask 255.255.255.0
	gateway 192.181.1.1
```

### Konfigurasi Garden

```
auto eth0
iface eth0 inet static
	address 192.181.1.3
	netmask 255.255.255.0
	gateway 192.181.1.1
```

### Konfigurasi Berlint

```
auto eth0
iface eth0 inet static
	address 192.181.2.2
	netmask 255.255.255.0
	gateway 192.181.2.1
```

### Konfigurasi Eden

```
auto eth0
iface eth0 inet static
	address 192.181.2.3
	netmask 255.255.255.0
	gateway 192.181.2.1
```

### Konfigurasi WISE sebagai DNS Master

```
auto eth0
iface eth0 inet static
	address 192.181.3.2
	netmask 255.255.255.0
	gateway 192.181.3.1
```

### Setting di Web Console

#### Untuk Ostania

```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.181.0.0/16

cat /etc/resolv.conf
```

#### Untuk Node Lain

```
echo nameserver 192.168.122.1 > /etc/resolv.conf
```

Lalu `ping google.com` dan liat hasilnya telah terkoneksi.</br>


</br>

Hasil Screenshot Node yang telah dibuat: </br>


</br>

# Nomor 2
Untuk mempermudah mendapatkan informasi mengenai misi dari Handler, bantulah Loid membuat website utama dengan akses wise.yyy.com dengan alias www.wise.yyy.com pada folder wise

Untuk mempermudah mendapatkan informasi mengenai misi dari Handler, bantulah Loid membuat website utama dengan akses wise.yyy.com dengan alias www.wise.yyy.com pada folder wise

### Setting domain WISE

#### Install Bind

```
apt-get update

apt-get install bind9 -y
```

#### Setting Domain

```
nano /etc/bind/named.conf.local

zone "wise.c04.com" {
	type master;
	file "/etc/bind/wise/wise.c04.com";
};

mkdir /etc/bind/wise

cp /etc/bind/db.local /etc/bind/wise/wise.c04.com

nano /etc/bind/wise/wise.c04.com

isi ini
wise.c04.com. IN SOA wise.c04.com. root.wise.c04.com. (
    2022100601 ; serial -> 2 ; serial
    604800 ; refresh
    86400 ; retry
    2419200 ; expire
    604800 ; minimum ttl
)
192.181.3.2 ; IP WISE


service bind9 restart
```

### Konfigurasi Berlint sebagai DNS Slave

Pada WISE tuliskan command berikut:

```
nano /etc/bind/named.conf.local

zone "wise.c04.com" {
    type master;
    notify yes;
    also-notify { 192.181.2.2; };
    allow-transfer { 192.181.2.2; };
    file "/etc/bind/wise/wise.c04.com";
};

service bind9 restart
```

Pada Berlint

```
apt-get update

apt-get install bind9 -y

nano /etc/bind/named.conf.local

zone "wise.c04.com" {
    type slave;
    masters { 192.181.3.2; }; // Masukan IP WISE tanpa tanda petik
    file "/var/lib/bind/wise.c04.com";
};

service bind9 restart
```

### Setting Nameserver pada Client (SSS & Garden)

```
nano /etc/resolv.conf

nameserver 192.181.3.2 ; IP WISE
nameserver 192.181.2.2 ; IP Berlint

ping wise.c04.com -c 5
```

Sehingga hasilnya akan seperti ini </br>
# Nomor 3
Setelah itu ia juga ingin membuat subdomain eden.wise.yyy.com dengan alias www.eden.wise.yyy.com yang diatur DNS-nya di WISE dan mengarah ke Eden

### Membuat Subdomain - WISE

```
nano /etc/bind/wise/wise.c04.com

tambahkan
eden 	IN   A  192.181.2.2 ; IP Berlint

service bind9 restart
```

### Test di Client

```
ping eden.wise.c04.com -c 5
```

Maka hasilnya adalah sebagai berikut : </br>
# Nomor 4
Buat juga reverse domain untuk domain utama

### Reverse Domain (Wise)

```
nano /etc/bind/named.conf.local

zone "2.3.181.192.in-addr.arpa" {
    type master;
    file "/etc/bind/wise/2.3.181.192.in-addr.arpa";
};

cp /etc/bind/db.local /etc/bind/wise/2.3.181.192.in-addr.arpa
```

### Dijalankan di Client (SSS & Garden)

```
apt-get update

apt-get install dnsutils

host -t PTR 192.181.3.2
```
# Nomor 5
Agar dapat tetap dihubungi jika server WISE bermasalah, buatlah juga Berlint sebagai DNS Slave untuk domain utama

### Konfigurasi Berlint sebagai DNS Slave (Seperti Nomor 2)

Pada WISE tuliskan command berikut:

```
nano /etc/bind/named.conf.local

zone "wise.c04.com" {
    type master;
    notify yes;
    also-notify { 192.181.2.2; };
    allow-transfer { 192.181.2.2; };
    file "/etc/bind/wise/wise.c04.com";
};

service bind9 restart
```

Pada Berlint

```
apt-get update

apt-get install bind9 -y

nano /etc/bind/named.conf.local

zone "wise.c04.com" {
    type slave;
    masters { 192.181.3.2; }; // Masukan IP WISE tanpa tanda petik
    file "/var/lib/bind/wise.c04.com";
};

service bind9 restart
```
# Nomor 6
Karena banyak informasi dari Handler, buatlah subdomain yang khusus untuk operation yaitu operation.wise.yyy.com dengan alias www.operation.wise.yyy.com yang didelegasikan dari WISE ke Berlint dengan IP menuju ke Eden dalam folder operation

### Setting Konfigurasi di Wise

```
nano /etc/bind/wise/wise.c04.com

ns1 	IN  	A  	192.181.2.2 ; IP Berlint
operation IN   	NS  ns1


nano /etc/bind/named.conf.options
allow-query{any;};


nano /etc/bind/named.conf.local

sesuaiin di modul

service bind9 restart
```

### Setting Konfigurasi di Berlint

```
nano /etc/bind/named.conf.options

allow-query{any;};

nano /etc/bind/named.conf.local

ganti jadi operation.wise.c04.com

mkdir /etc/bind/wise
cp /etc/bind/db.local /etc/bind/wise/operation.wise.c04.com

nano /etc/bind/wise/operation.wise.c04.com

operation.wise.c04.com. IN SOA operation.wise.c04.com. root.operation.wise.c04.com. (
	2022100601 ; serial -> 2 ; serial
	604800 ; refresh
	86400 ; retry
	2419200 ; expire
	604800 ; minimum ttl
)
@	IN	NS	operation.wise.c04.com.
@	IN	A 	192.181.2.2 ; IP Berlint

service bind9 restart

ping operation.wise.c04.com -c 5
```
# Nomor 7
Untuk informasi yang lebih spesifik mengenai Operation Strix, buatlah subdomain melalui Berlint dengan akses strix.operation.wise.yyy.com dengan alias www.strix.operation.wise.yyy.com yang mengarah ke Eden

```
nano /etc/bind/operation/operation.wise.c04.com

operation.wise.c04.com. IN SOA operation.wise.c04.com. root.operation.wise.c04.com. (
	2022100601 ; serial -> 2 ; serial
	604800 ; refresh
	86400 ; retry
	2419200 ; expire
	604800 ; minimum ttl
)
@	IN	NS	operation.wise.c04.com.
@	IN	A 	192.181.2.2 ; IP Berlint
strix IN A  192.181.2.2 ; IP Berlint

service bind9 restart

ping strix.operation.wise.c04.com -c 5
```
# Nomor 8
Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver. Pertama dengan webserver www.wise.yyy.com. Pertama, Loid membutuhkan webserver dengan DocumentRoot pada /var/www/wise.yyy.com
# Nomor 9
Setelah itu, Loid juga membutuhkan agar url www.wise.yyy.com/index.php/home dapat menjadi menjadi www.wise.yyy.com/home
# Nomor 10
Setelah itu, pada subdomain www.eden.wise.yyy.com, Loid membutuhkan penyimpanan aset yang memiliki DocumentRoot pada /var/www/eden.wise.yyy.com
# Nomor 11
Akan tetapi, pada folder /public, Loid ingin hanya dapat melakukan directory listing saja
# Nomor 12
Tidak hanya itu, Loid juga ingin menyiapkan error file 404.html pada folder /error untuk mengganti error kode pada apache
# Nomor 13
Loid juga meminta Franky untuk dibuatkan konfigurasi virtual host. Virtual host ini bertujuan untuk dapat mengakses file asset www.eden.wise.yyy.com/public/js menjadi www.eden.wise.yyy.com/js
# Nomor 14
Loid meminta agar www.strix.operation.wise.yyy.com hanya bisa diakses dengan port 15000 dan port 15500
# Nomor 15
dengan autentikasi username Twilight dan password opStrix dan file di /var/www/strix.operation.wise.yyy
# Nomor 16
dan setiap kali mengakses IP Eden akan dialihkan secara otomatis ke www.wise.yyy.com
# Nomor 17
Karena website www.eden.wise.yyy.com semakin banyak pengunjung dan banyak modifikasi sehingga banyak gambar-gambar yang random, maka Loid ingin mengubah request gambar yang memiliki substring “eden” akan diarahkan menuju eden.png. Bantulah Agent Twilight dan Organisasi WISE menjaga perdamaian!

### Terima Kasih:D
