# Panduan Instalasi Nginx dan Certbot

![Nginx Logo](https://nginx.org/nginx.png)

Ini adalah panduan langkah demi langkah untuk menginstal Nginx dan Certbot pada server Linux Anda. Nginx adalah server web yang ringan dan kuat, sedangkan Certbot adalah alat untuk mengelola sertifikat SSL/TLS dari Let's Encrypt.

## Instalasi Nginx

### 1. Instalasi snapd

```bash
sudo apt update
sudo apt install snapd
```

### 2. Instalasi Nginx Menggunakan Snap

```bash
sudo snap install nginx
```
### 2.1 Instalasi Nginx Menggunakan APT

```bash
sudo apt install nginx
```
# 2.1.1 Start Nginx 
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```
## Konfigurasi Nginx
### 1.Buat konfigurasi untuk situs Anda di /etc/nginx/sites-available/.
```bash
sudo nano /etc/nginx/sites-available/app1
```
Contoh konfigurasi untuk kocak.my.id:

```bash
server {
    listen 80;
    server_name app1.kocak.my.id;

    location / {
        proxy_pass http://localhost:2409;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_cache_bypass $http_upgrade;
    }
}
```
## Konf UFW dulu agar bsa pakai http dan HTTPS
```bash
sudo ufw enable
sudo ufw allow 'Nginx Full'
sudo ufw reload
```
## 2. Aktifkan Konfigurasi NGINX
```bash
sudo ln -s /etc/nginx/sites-available/app1 /etc/nginx/sites-enabled/
```
## 3. Periksa Konfigurasi NGINX
```bash
sudo nginx -t
```
#### Jika tidak ada kesalahan, restart Nginx:
```bash
sudo systemctl restart nginx
```
## Dapatkan Sertifikat SSL dengan Certbot
Gunakan Certbot untuk mendapatkan sertifikat SSL untuk masing-masing domain. Certbot akan mengonfigurasi Nginx secara otomatis untuk menggunakan HTTPS:
1. Install Dulu..
```bash
sudo snap install --classic certbot
```
2. Prepare..
```bash
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```
3. Mulai Jalankan Pembuatan Cert..
```bash
sudo certbot --nginx
sudo certbot certonly --nginx
```
4. Automatic renewal..
```bash
sudo certbot --nginx
sudo certbot certonly --nginx
```
### Ubah Lagi di /etc/nginx/sites-available pangil aja nano
ini tuh versi yang udah di ubah mengatur portnya ke 443
### Sesuaikan Dengan Nama Appmu
```bash
nano /etc/nginx/sites-available/app1
```
### Ganti Menjadi ini 
```bash
server {
    server_name nameserver.com;

    location / {
        if ($request_method = 'OPTIONS') {
            add_header 'Access-Control-Allow-Origin' '*';
            add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS, PUT, PATCH, DELETE';
            add_header 'Access-Control-Allow-Headers' 'Authorization, Content-Type, Accept';
            add_header 'Access-Control-Max-Age' 1728000;
            add_header 'Content-Type' 'text/plain charset=UTF-8';
            add_header 'Content-Length' 0;
            return 204;
        }

        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS, PUT, PATCH, DELETE';
        add_header 'Access-Control-Allow-Headers' 'Authorization, Content-Type, Accept';

        proxy_pass http://localhost:2006;  # Mengarahkan permintaan ke server app.js
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_cache_bypass $http_upgrade;
        proxy_set_header Authorization $http_authorization;  # Meneruskan header Authorization
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/nameserver.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/nameserver.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

server {
    if ($host = nameserver.com) {
        return 301 https://$host$request_uri; # Mengarahkan HTTP ke HTTPS
    } # managed by Certbot

    listen 80;
    server_name nameserver.com;
    return 404; # Jika tidak bisa mengalihkan, kembali dengan 404
}

```
### Lanjutt

```bash
sudo nginx -t
sudo systemctl restart nginx

```
### Kalau Bisa Jangan Logout Dulu Dari Vps, Karena takut tidak bisa login di port 22

#Jika VPS Anda tidak dapat diakses melalui port 22 (SSH) setelah menginstal Nginx dan Certbot, ada beberapa langkah yang dapat Anda coba untuk mengatasi masalah ini:

1. Periksa Status SSH Service
Pastikan layanan SSH (OpenSSH) berjalan dengan benar di server Anda. Jalankan perintah berikut untuk memeriksa status layanan SSH:
```bash
sudo systemctl status ssh
```
#Jika layanan tidak berjalan, mulai ulang dengan perintah berikut:
```bash
sudo systemctl start ssh
sudo systemctl enable ssh
```
2. Periksa Konfigurasi Firewall
Pastikan firewall tidak memblokir port 22. Jika Anda menggunakan UFW, buka port 22 dengan perintah berikut:
```bash
sudo ufw allow 22/tcp
sudo ufw reload
```
##Pastikan konfigurasi Nginx tidak mempengaruhi akses SSH. Biasanya, Nginx seharusnya tidak mempengaruhi port 22, tetapi pastikan tidak ada kesalahan konfigurasi yang aneh. Jalankan perintah berikut untuk memeriksa konfigurasi Nginx:
```bash
sudo nginx -t
```
#Jika ada kesalahan, perbaiki dan muat ulang Nginx dengan perintah berikut:
```bash
sudo systemctl reload nginx
```

# Cara Mendaftarkan Domain Baru Pada NGINX dan CERTBOT
## 1. Menyiapkan Nginx untuk Domain Baru
### Buat file konfigurasi baru untuk domain Anda di direktori /etc/nginx/sites-available/. Misalnya, jika domain Anda adalah newdomain.com, buat file newdomain.com:
```bash
sudo nano /etc/nginx/sites-available/appname
```
### Isi file tersebut dengan konfigurasi dasar untuk domain baru:
```bash
server {
    listen 80;
    server_name newdomain.com;

    location / {
        proxy_pass http://localhost:2006;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_cache_bypass $http_upgrade;
    }
}
```
### Aktifkan situs dengan membuat symbolic link ke direktori sites-enabled:
```bash
sudo ln -s /etc/nginx/sites-available/appname /etc/nginx/sites-enabled/
```
## 2. Verifikasi bahwa symbolic link telah dibuat dengan benar
### Anda harus melihat link mc-jkt.lanzzstore.com yang mengarah ke file di sites-available.
```bash
ls -l /etc/nginx/sites-enabled/
```
## 3. Uji Conf Nginx & Reload Nginx
```bash
sudo nginx -t
sudo systemctl reload nginx
```
## 4. Mendapatkan Sertifikat SSL Menggunakan Certbot
### Instal Certbot dan plugin Nginx jika belum!
### Jalankan Certbot untuk mendapatkan sertifikat SSL untuk domain baru Anda:
```bash
sudo certbot --nginx -d newdomain.com
```
# Note : Udah si gitu aja yang penting kelen baca aja ya klu ada eror tanya starkoverflow / chatgpt. Maaf kalau ada yang kurang. Terima Kasih



