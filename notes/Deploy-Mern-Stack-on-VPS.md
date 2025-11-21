<h1 align="center">🚀 Deploying a MERN Stack Project on Hostinger VPS</h1>
<p align="center">
  <em>Complete Step-by-Step Deployment Guide with <strong>Nginx</strong>, <strong>PM2</strong>, and <strong>SSL</strong></em>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=node.js&logoColor=white"/>
  <img src="https://img.shields.io/badge/React-61DAFB?style=for-the-badge&logo=react&logoColor=black"/>
  <img src="https://img.shields.io/badge/Express-000000?style=for-the-badge&logo=express&logoColor=white"/>
  <img src="https://img.shields.io/badge/Next.js-000000?style=for-the-badge&logo=next.js&logoColor=white"/>
  <img src="https://img.shields.io/badge/Hostinger-673DE6?style=for-the-badge&logo=hostinger&logoColor=white"/>
  <img src="https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black"/>
</p>

---

## 🧰 Prerequisites
Before starting, make sure you have:
- ✅ A **Hostinger VPS**
- ✅ **SSH access** to your VPS
- ✅ A **domain name** (and subdomains if you have multiple apps)
- ✅ Your **MERN project repo** on GitHub (frontend & backend)
- ✅ (Optional) MongoDB Atlas or MongoDB/MySQL installed on the VPS

---

## 1️⃣ Prepare Your VPS Environment

**Login to your VPS**
```bash
ssh root@your_vps_ip
````

**Update & upgrade system**

```bash
sudo apt update
```
```bash
sudo apt list --upgradable
```
```bash
sudo apt upgrade
```
```bash
sudo apt dist-upgrade
```

**Install Node.js & npm using NVM**

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
```

```bash
\. "$HOME/.nvm/nvm.sh"
```

```bash
nvm install 25
```

```bash
node -v
```

```bash
npm -v
```

**Install Git**

```bash
sudo apt install -y git
```

---

## 2️⃣ Set Up Database

If you want to setup ***MySQL*** on VPS, follow this guide: [CLICK HERE→](./MySQL-DB-VPS-Setup.md)  
If you want to setup ***MongoDB*** on VPS, follow this guide: [CLICK HERE→](./Mongo-DB-VPS-Setup.md)  
Alternatively, you can use ***MongoDB Atlas*** and simply update your `.env` file with your connection string.  

---

## 3️⃣ Deploy the Backend (Express + Node.js)

**Clone your backend repo**

```bash
mkdir -p /var/wwww
```

```bash
cd /var/www
```

```bash
git clone https://github.com/yourusername/your-repo.git
```

```bash
cd your-repo/backend
```

**Install dependencies**

```bash
npm install
```

**Create and configure `.env`**

```bash
nano .env
```

Paste your environment variables, then press `Ctrl + X`, `Y`, and `Enter` to save.

**Install and start backend with PM2**

```bash
npm install -g pm2
```

```bash
pm2 start server.js --name project-backend
```

**Start backend on system startup**

```bash
pm2 startup
```

```bash
pm2 save
```

**Allow Backend Port in Firewall**

```bash
sudo ufw status
```

```bash
sudo ufw enable
```

```bash
sudo ufw allow 'OpenSSH'
```

```bash
sudo ufw allow 5000
```

---

## 4️⃣ Deploy the Frontend (React / Next.js)

**Navigate to your frontend directory**

```bash
cd /var/www/your-repo/frontend
```

**Install dependencies**

```bash
npm install
```

**Create `.env` file**

```bash
nano .env
```

Add your environment variables. To save, press `Ctrl + X`, then `Y`, and `Enter`.

**Build your frontend**

```bash
npm run build
```

**Start frontend using PM2**

```bash
pm2 start "npm start" --name "process-name"
```

**(Optional) Run on a custom port**

```bash
pm2 start "npm start -- -p 8080" --name "process-name"
```

**Enable PM2 on startup**

```bash
pm2 startup
```

```bash
pm2 save
```

If you have multiple Next.js or React apps, repeat these steps for each one.

---

## 5️⃣ Connect Your Domain

In your domain DNS manager, point the following records to your VPS IP address:

| Subdomain | Type | Value         |
| --------- | ---- | ------------- |
| `@`       | A    | `your_vps_ip` |
| `www`     | A    | `your_vps_ip` |
| `api`     | A    | `your_vps_ip` |

---

## 6️⃣ Enable SSL (HTTPS)

**Install Certbot**

```bash
sudo apt install -y certbot python3-certbot-nginx
```

**Get SSL Certificates**

```bash
sudo certbot --nginx -d yourdomain1.com -d www.yourdomain1.com -d yourdomain2.com -d api.yourdomain.com
```

**Verify Auto-Renewal**

```bash
sudo certbot renew --dry-run
```

---

## 7️⃣ Install & Configure Nginx

**Install Nginx**

```bash
sudo apt install -y nginx
```

```bash
sudo ufw allow 'Nginx Full'
```

---

### 🔹 FRONTEND SETUP

**Create frontend Nginx config**

```bash
sudo nano /etc/nginx/sites-available/your-domain.com.conf
```

```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name your-domain.com www.your-domain.com;

    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    # Frontend (Next.js)
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**Enable sites & restart Nginx**

```bash
sudo ln -s /etc/nginx/sites-available/your-domain.com.conf /etc/nginx/sites-enabled/
```

```bash
sudo nginx -t
```

```bash
sudo systemctl restart nginx
```

##

### 🔹 For Another React App (Optional)

```bash
sudo nano /etc/nginx/sites-available/yourdomain2.com.conf
```

```nginx
server {
    listen 80;
    server_name yourdomain2.com www.yourdomain2.com;

    location / {
        root /var/www/react-app-2/dist;
        try_files $uri /index.html;
    }
}
```

**Enable sites & restart Nginx**

```bash
sudo ln -s /etc/nginx/sites-available/your-domain2.com.conf /etc/nginx/sites-enabled/
```

```bash
sudo nginx -t
```

```bash
sudo systemctl restart nginx
```

##

### 🔹 BACKEND SETUP

**Create backend Nginx config**

```bash
sudo nano /etc/nginx/sites-available/api.your-domain.com.conf
```

```nginx
server {
    listen 80;
    server_name api.your-domain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name api.your-domain.com;

    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        proxy_pass http://localhost:5000; 
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**Enable sites & restart Nginx**

```bash
sudo ln -s /etc/nginx/sites-available/api.your-domain.com.conf /etc/nginx/sites-enabled/
```

```bash
sudo nginx -t
```

```bash
sudo systemctl restart nginx
```

##

### 🔹 FRONTEND & BACKEND SETUP 

**Nginx config**

```bash
sudo nano /etc/nginx/sites-available/your-domain.com.conf
```

```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name your-domain.com www.your-domain.com;

    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

   # --- Express backend API ---
    location /api/ {
        proxy_pass http://localhost:5000;

        # Disable caching to ensure fresh content
        proxy_no_cache 1;
        proxy_cache_bypass 1;

        # cache-control headers to prevent caching at the client side
        add_header Cache-Control no-cache, no-store, must-revalidate;
        add_header Pragma no-cache;
        add_header Expires 0;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Frontend (Next.js)
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**Enable sites & restart Nginx**

```bash
sudo ln -s /etc/nginx/sites-available/your-domain.com.conf /etc/nginx/sites-enabled/
```

```bash
sudo nginx -t
```

```bash
sudo systemctl restart nginx
```
##

## ⚙️ Useful PM2 Commands

| Command                  | Description                |
| ------------------------ | -------------------------- |
| `pm2 list`               | Show all running apps      |
| `pm2 logs`               | View logs                  |
| `pm2 restart <app_name>` | Restart specific app       |
| `pm2 stop <app_name>`    | Stop specific app          |
| `pm2 delete <app_name>`  | Remove app from PM2        |
| `pm2 save`               | Save current process list  |
| `pm2 startup`            | Run apps on system startup |

##

## 🧩 Useful System Commands

| Command                        | Description                    |
| ------------------------------ | ------------------------------ |
| `sudo systemctl status nginx`  | Check Nginx status             |
| `sudo systemctl restart nginx` | Restart Nginx                  |
| `sudo nginx -t`                | Test Nginx config syntax       |
| `sudo ufw status`              | Check firewall status          |
| `sudo ufw allow <port>`        | Allow a specific port          |
| `sudo certbot renew`           | Renew SSL certificate manually |

##

## ✅ Done!

Your MERN stack app is now live and secured with SSL!

* 🌐 **Frontend:** [https://yourdomain1.com](https://yourdomain1.com)
* 🌐 **Backend:** [https://api.yourdomain.com](https://api.yourdomain.com)
* 🔒 **Secured:** Nginx + Certbot + PM2

##

## 👨‍💻 Author

**Muzaffar Khan**  
**GitHub:** [https://github.com/muzaffar-khan](https://github.com/muzaffar-khan)

> 📝 **Tip:** Always keep your system and dependencies up to date to ensure better performance and security.
