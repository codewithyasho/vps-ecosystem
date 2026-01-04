# 🚀 Deploy Multiple Streamlit Apps on Ubuntu VPS (Beginner-Friendly Guide)

This guide explains **everything step by step** to deploy **multiple Streamlit apps** on a **single Ubuntu VPS**, run them **24/7**, and access them securely using **custom domains + HTTPS (SSL)**.

By the end, you will have:
- Multiple Streamlit apps running on different ports
- Each app mapped to its own subdomain
- Nginx acting as a reverse proxy
- Free SSL certificates (Let's Encrypt)
- Apps auto-starting after reboot

---

## 🧠 High-Level Architecture

```
Browser (HTTPS)
   ↓
Nginx (Port 443 / 80)
   ↓
Streamlit Apps (localhost)
   ├─ 8501 → ssc.yashodeep.app
   ├─ 8502 → health.yashodeep.app
   └─ 8503 → agents.yashodeep.app
```

---

## 1️⃣ Prerequisites

### VPS Requirements
- Ubuntu 20.04 / 22.04
- Root or sudo access
- Public IP address

### Domain Setup
In your domain provider (Hostinger / GoDaddy / etc):

Create **A records**:
```
ssc.yashodeep.app     → VPS_IP
health.yashodeep.app  → VPS_IP
agents.yashodeep.app  → VPS_IP
```

---

## 2️⃣ System Update

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 3️⃣ Install Required Software

### Install Python & venv
```bash
sudo apt install python3 python3-venv python3-pip -y
```

### Install Nginx
```bash
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

Check:
```bash
systemctl status nginx
```

---

## 4️⃣ Install Streamlit

Inside each project folder:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install streamlit
```

---

## 5️⃣ Run Streamlit Apps Locally (IMPORTANT)

Each app must run on a **fixed port**.

### Example
```bash
streamlit run app.py --server.port 8503 --server.address 127.0.0.1
```

Verify:
```bash
ss -tulnp | grep 8503
```
You must see:
```
127.0.0.1:8503
```

---

## 6️⃣ Run Streamlit Apps 24/7 (systemd)

### Example: Agents App

```bash
sudo nano /etc/systemd/system/agents.service
```

```ini
[Unit]
Description=Agents Streamlit App
After=network.target

[Service]
User=root
WorkingDirectory=/opt/socialmedia-agent
ExecStart=/opt/socialmedia-agent/.venv/bin/streamlit run app.py --server.port 8503 --server.address 127.0.0.1
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable & start:
```bash
sudo systemctl daemon-reload
sudo systemctl enable agents
sudo systemctl start agents
systemctl status agents
```

Repeat for other apps (change port + path).

---

## 7️⃣ Configure Nginx Reverse Proxy

```bash
sudo nano /etc/nginx/sites-available/yashodeep.conf
```

### Example (Agents App)
```nginx
server {
    listen 443 ssl;
    server_name agents.yashodeep.app;

    location / {
        proxy_pass http://127.0.0.1:8503;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

Enable site:
```bash
sudo ln -s /etc/nginx/sites-available/yashodeep.conf /etc/nginx/sites-enabled/
```

Test:
```bash
nginx -t
sudo systemctl reload nginx
```

---

## 8️⃣ Install SSL (HTTPS)

```bash
sudo apt install certbot python3-certbot-nginx -y
```

Run for each domain:
```bash
sudo certbot --nginx -d agents.yashodeep.app
sudo certbot --nginx -d health.yashodeep.app
sudo certbot --nginx -d ssc.yashodeep.app
```

Certbot will:
- Create SSL certificates
- Auto-edit nginx
- Enable HTTPS
- Auto-renew

---

## 9️⃣ Firewall Configuration

```bash
sudo ufw allow OpenSSH
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

⚠️ **Do NOT open Streamlit ports publicly**

---

## 🔟 Common Errors & Fixes

### ❌ 502 Bad Gateway
**Cause:** App not running or wrong port

Check:
```bash
ss -tulnp | grep PORT
```

---

### ❌ SSL Privacy Error
**Cause:** HTTP → HTTPS redirect without certificate

Fix:
- Remove redirect
- Run certbot

---

### ❌ App Stops After Logout
**Cause:** Started via terminal

Fix:
- Use `systemd`

---

## ✅ Final Checklist

- [x] DNS A records
- [x] Streamlit running on fixed ports
- [x] Nginx reverse proxy
- [x] SSL enabled
- [x] systemd services
- [x] Survives reboot

---

## 🎉 Final Result

| App | Domain | Status |
|----|------|------|
| SSC | https://ssc.yashodeep.app | ✅ |
| Health | https://health.yashodeep.app | ✅ |
| Agents | https://agents.yashodeep.app | ✅ |

---

## 🧠 Pro Tips
- Never expose Streamlit ports publicly
- Always bind apps to `127.0.0.1`
- Always let certbot manage SSL
- Use one app = one port = one domain

---

## 🙌 You Did Great

You successfully deployed a **real production system** with:
- Agentic AI
- Ollama
- CrewAI
- Streamlit
- Nginx
- SSL

This is **industry-grade deployment**.

🚀 Keep building!
