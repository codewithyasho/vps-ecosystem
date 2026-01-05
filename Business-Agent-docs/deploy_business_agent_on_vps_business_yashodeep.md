# 🚀 Deploy Business Market Research Agent on Ubuntu VPS

This guide shows **exactly how to deploy ONE Streamlit app** (Business Market Research Agent) on your VPS and connect it to:

👉 **https://business.yashodeep.app**

It is **beginner‑friendly**, copy‑paste ready, and based on what you already did successfully.

---

## 🧠 Final Architecture

```
Browser (HTTPS)
   ↓
Nginx (443 / 80)
   ↓
Streamlit App (127.0.0.1:8504)
   ↓
Business Agent (GitHub Repo)
```

---

## 1️⃣ DNS Configuration

In your domain DNS panel, add this **A record**:

| Type | Host | Value |
|---|---|---|
| A | business | 72.61.238.224 |

TTL: `300`

⏳ Wait 1–5 minutes for DNS to propagate.

---

## 2️⃣ SSH Into Your VPS

```bash
ssh root@72.61.238.224
```

---

## 3️⃣ Clone GitHub Repository

```bash
cd /opt
git clone https://github.com/codewithyasho/business-agent.git
cd business-agent
```

---

## 4️⃣ Create Virtual Environment & Install Dependencies

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

---

## 5️⃣ Test Streamlit App Locally (IMPORTANT)

```bash
streamlit run app.py --server.port 8504 --server.address 127.0.0.1
```

Open in browser:
```
http://127.0.0.1:8504
```

✅ If app loads correctly → stop it using **Ctrl+C**

---

## 6️⃣ Run App 24/7 Using systemd

### Create service file

```bash
sudo nano /etc/systemd/system/business.service
```

### Paste this configuration

```ini
[Unit]
Description=Business Market Research Agent
After=network.target

[Service]
User=root
WorkingDirectory=/opt/business-agent
ExecStart=/opt/business-agent/.venv/bin/streamlit run app.py --server.port 8504 --server.address 127.0.0.1
Restart=always

[Install]
WantedBy=multi-user.target
```

### Enable & start service

```bash
sudo systemctl daemon-reload
sudo systemctl enable business
sudo systemctl start business
systemctl status business
```

Verify port:
```bash
ss -tulnp | grep 8504
```
You must see:
```
127.0.0.1:8504
```

---

## 7️⃣ Configure Nginx Reverse Proxy

Open nginx config:

```bash
nano /etc/nginx/sites-available/yashodeep.conf
```

### Add this block

```nginx
# =========================
# BUSINESS APP (8504)
# =========================
server {
    listen 80;
    server_name business.yashodeep.app;

    location / {
        proxy_pass http://127.0.0.1:8504;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

Reload nginx:
```bash
nginx -t
sudo systemctl reload nginx
```

Test:
```
http://business.yashodeep.app
```

---

## 8️⃣ Enable HTTPS (SSL Certificate)

```bash
sudo certbot --nginx -d business.yashodeep.app
```

Certbot will:
- Install SSL certificate
- Enable HTTPS (443)
- Redirect HTTP → HTTPS
- Auto‑renew certificate

---

## 9️⃣ Firewall Check

```bash
sudo ufw allow 80
sudo ufw allow 443
sudo ufw reload
```

⚠️ **Do NOT open port 8504 publicly**

---

## ✅ Final Result

Open in browser:

```
https://business.yashodeep.app
```

🎉 Your Business Market Research Agent is now **LIVE, secure, and running 24/7**.

---

## ❌ Common Errors & Fixes

### 502 Bad Gateway
- App not running
- Wrong port in nginx

Check:
```bash
ss -tulnp | grep 8504
```

---

### App Stops After Logout
- Started using terminal

✅ Fix: Use `systemd` (already done)

---

## 🧠 Best Practices (Remember)

- One app = one port = one domain
- Always bind Streamlit to `127.0.0.1`
- Never expose Streamlit ports publicly
- Always let **certbot manage SSL**

---

## 🚀 You Are Production‑Ready

You now know how to:
- Deploy Streamlit apps
- Use Nginx reverse proxy
- Configure SSL
- Run apps 24/7

This is **industry‑level deployment knowledge**.

🔥 Keep building!

