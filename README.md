# 🚀 Deploy Docker + Nginx + SSL on AWS EC2 (Ubuntu)

This guide installs **Docker on EC2**, runs **Nginx**, maps a **domain**, and secures it using **Let’s Encrypt SSL** — all using Docker.

---

# ✅ 1. Update Server

```bash
sudo apt update -y
sudo apt upgrade -y
```

---

# ✅ 2. Install Docker

```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
```

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

```bash
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

```bash
sudo apt update -y
sudo apt install docker-ce -y
```

Enable & start Docker:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Check:

```bash
docker --version
```

---

# ✅ 3. Run Nginx (HTTP)

```bash
sudo docker run -d --name nginx-container -p 80:80 nginx
```

Open in browser:

```
http://YOUR_DOMAIN_OR_IP
```

---

# 🌐 4. Add DNS Records

Go to your domain provider and add:

| Type | Name | Value |
|------|-------|----------------|
| A | @ | YOUR_EC2_PUBLIC_IP |
| A | www | YOUR_EC2_PUBLIC_IP |

Wait 2–5 minutes.

---

# 🔐 5. Install Certbot (SSL Generator)

```bash
sudo apt install certbot -y
sudo apt install python3-certbot-nginx -y
```

Stop Nginx container so Certbot can use port 80:

```bash
sudo docker stop nginx-container
```

---

# 🔑 6. Generate SSL Certificate

```bash
sudo certbot certonly --standalone -d yourdomain.com -d www.yourdomain.com
```

Certificates will be stored in:

```
/etc/letsencrypt/live/yourdomain.com/
```

---

# ⚙️ 7. Create Custom Nginx SSL Config

```bash
sudo nano /home/ubuntu/default.conf
```

Paste:

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name yourdomain.com www.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
}
```

Save → `CTRL + O` → Enter → `CTRL + X`

---

# 🧹 8. Remove Old Container

```bash
sudo docker rm nginx-container
```

---

# 🚀 9. Run Nginx With SSL (HTTPS)

```bash
sudo docker run -d --name nginx-container \
  -p 80:80 -p 443:443 \
  -v /etc/letsencrypt:/etc/letsencrypt \
  -v /home/ubuntu/default.conf:/etc/nginx/conf.d/default.conf \
  nginx
```

---

# 🔎 10. Verify Everything

Check container:

```bash
sudo docker ps
```

Check ports:

```bash
sudo ss -tulpn | grep -E '80|443'
```

Open in browser:

```
https://yourdomain.com
```

You should see **Secure 🔒 HTTPS**.

---

# 🎉 DONE!

Your EC2 now hosts:

- Docker  
- Nginx  
- Domain mapped  
- Fully working HTTPS (Let’s Encrypt SSL)  
- Auto-redirect HTTP → HTTPS  
