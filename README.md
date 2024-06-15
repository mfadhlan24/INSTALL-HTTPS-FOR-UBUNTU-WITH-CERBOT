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
```bash
server {
    listen 443 ssl; # managed by Certbot
    server_name app1.kocak.my.id;

    ssl_certificate /etc/letsencrypt/live/app1.kocak.my.id/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/app1.kocak.my.id/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

    location / {
        proxy_pass http://localhost:2005;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_cache_bypass $http_upgrade;
    }
}

server {
    listen 80;
    server_name app1.kocak.my.id;

    if ($host = app1.kocak.my.id) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    return 404; # managed by Certbot
}

```
### Lanjutt

```bash
sudo nginx -t
sudo systemctl restart nginx

```

# Note : Udah si gitu aja yang penting kelen baca aja ya klu ada eror tanya starkoverflow / chatgpt. Maaf kalau ada yang kurang. Terima Kasih
