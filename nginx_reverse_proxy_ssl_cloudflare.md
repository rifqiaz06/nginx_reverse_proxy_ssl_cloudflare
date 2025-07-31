## 1. Pastikan vm sudah di buat dengan format 1 load balancer 2-3 app untuk kasus sederhana kali ini.
untuk vm bisa dibuat secara manual via virtualbox, atau lewat docker. dan Pastikan untuk vm nya verjalan di VPS yang mempunyai IP public atau jika local bisa di tunneling terlebih dahulu via ngrok, cloudflare atau semacamnya.

jika menggunakan VPS jangan lupa tambahkan konfigurasi menggunakan nameserver yang ada di cloudflare. untuk konfigurasi di VPS dengan _Update Nameserver_ di platform VPS nya. Lalu jangan lupa tambahkan _DNS Record_ pada Cloudflare yaitu dengan: DNS -> Records -> Add record -> IP Address: {IP Public VPS}, Name: @ (for root), Nonaktifkan Proxy status (optional), Save. DNS bisa dicek di whatsmydns.net.

## 2. Install dan konfigurasi nginx terlebih dahulu:

```bash
sudo apt install nginx
sudo systemctl status nginx
ls /etc/nginx/
sudo rm /etc/nginx/sites-available/default
sudo rm /etc/nginx/sites-enabled/default
```

## 3. Pointing Domain Cloudflare

- Login dan buat project di Cloudflare
- Tambahkan DNS Record dengan masuk ke DNS -> Add records: Nama: app-1/app-2/app-3 (contoh), IP Address: {IP Server/VPS} (bisa cek dengan command: `echo $(curl -s ifconfig.me)` ), Nonaktifkan Proxy status. Lalu cek di whatsmydns.net ({nama dns}.{nama-project cloudflare}).
- Tambahkan file konfigurasi baru:

  ```bash
  sudo nano /etc/nginx/sites-available/app-1 (contoh)
  sudo nano /etc/nginx/sites-available/app-2 (contoh)
  sudo nano /etc/nginx/sites-available/app-3 (contoh)

  server {
  	   listen 80;
          server_name app-1/app-2/app-3.{nama project cloudflare};

          location / {
     	   proxy_pass http://localhost:{port app-1/2/3};
          }
  }

  sudo ln -s /etc/nginx/sites-available/app-1 /etc/nginx/sites-enabled/
  sudo ln -s /etc/nginx/sites-available/app-2 /etc/nginx/sites-enabled/
  sudo ln -s /etc/nginx/sites-available/app-3 /etc/nginx/sites-enabled/

  sudo nginx -t
  sudo systemctl restart nginx
  sudo systemctl status nginx

  ```

- Coba akses service/website via Domain {app-1/2/3}.{nama project cloudflare}

## 4. Menambahkan Konfigurasi SSL/HTTPS

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d "{app-1/2/3}.{nama project cloudflare}"
```

Coba cek lagi service/website nya apakah sudah muncul encrypt/https-nya atau belum.

- secara otomatis nanti certbot akan menambahkan konfigurasi SSL pada file /etc/nginx/sites-available/app-1,app-2,app-3
