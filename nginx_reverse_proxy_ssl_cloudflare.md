## 1. Pastikan VM sudah di buat dengan format 1 reverse proxy 2-3 apps.
Untuk VM bisa dibuat secara manual via virtualbox, atau lewat docker. dan Pastikan untuk VM nya verjalan di VPS yang mempunyai IP Public atau jika local bisa di tunneling terlebih dahulu via Ngrok, Cloudflare atau semacamnya.

```

                                      Public
                                      Network
                                        |
                                        |  app-1.yourdomain.com
                                        |  app-2.yourdomain.com
                                        |
                                        v
                                +------------------+
                                |   VM1 (RP)       |  --> NGINX (Reverse Proxy)
                                |  192.168.0.11    |
                                +------------------+
                                        |   
                          ------------------------------              
                          |                            |
                          |                            |
                          v                            v
                +---------------------+          +--------------------+
                |   VM2 (app1)        |          |   VM3 (app2)       |
                |  192.168.0.12:80    |          |  192.168.0.13:80   |
                +---------------------+          +--------------------+
```

Jika menggunakan VPS jangan lupa tambahkan konfigurasi menggunakan **Nameserver** yang ada di cloudflare. untuk konfigurasi di VPS dengan *Update Nameserver* di platform VPS nya. Lalu jangan lupa tambahkan *DNS Record* pada Cloudflare yaitu dengan: **DNS** -> **Records** -> **Add record** -> **IP Address:** `{IP Public VPS}`, **Name:** `@` (for root), **Nonaktifkan Proxy status** (optional), **Save.** DNS bisa dicek di `whatsmydns.net`.

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
  sudo nano /etc/nginx/sites-available/app-1
  sudo nano /etc/nginx/sites-available/app-2
  sudo nano /etc/nginx/sites-available/app-3
  ```

  ```bash
  #App 1
  server {
  	   listen 80;
          server_name app-1.yourdomain.com;

          location / {
     	   proxy_pass http://localhost:80;
          }
  }

  #App 2
  server {
  	   listen 80;
          server_name app-2.yourdomain.com;

          location / {
     	   proxy_pass http://localhost:80;
          }
  }

  #App 3
  server {
  	   listen 80;
          server_name app-3.yourdomain.com;

          location / {
     	   proxy_pass http://localhost:80;
          }
  }
  ```

  ```bash
  sudo ln -s /etc/nginx/sites-available/app-1 /etc/nginx/sites-enabled/
  sudo ln -s /etc/nginx/sites-available/app-2 /etc/nginx/sites-enabled/
  sudo ln -s /etc/nginx/sites-available/app-3 /etc/nginx/sites-enabled/

  sudo nginx -t
  sudo systemctl restart nginx
  sudo systemctl status nginx
  ```

- Coba akses service/website via Domain __app-1.yourdomain.com__, __app-2.yourdomain.com__, __app-3.yourdomain.com__ .

## 4. Menambahkan Konfigurasi SSL/HTTPS

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d "app-1.yourdomain.com"
sudo certbot --nginx -d "app-2.yourdomain.com"
sudo certbot --nginx -d "app-3.yourdomain.com"
```

Coba cek lagi service/website nya apakah sudah muncul encrypt/https-nya atau belum.

_Secara otomatis nanti certbot akan menambahkan konfigurasi SSL pada file `/etc/nginx/sites-available/app-1,app-2,app-3`_
