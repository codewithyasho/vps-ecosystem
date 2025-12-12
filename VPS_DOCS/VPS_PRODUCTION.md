# VPS Production Deployment Guide

This guide summarizes the complete process used to deploy two Streamlit RAG applications on a Hostinger KVM VPS with HTTPS, systemd auto-start, Nginx proxy, and custom subdomains.

## 1. VPS Specs
- **2 vCPU cores**
- **8 GB RAM**
- **100 GB NVMe**
- **8 TB bandwidth**
- OS: **Ubuntu 24.04 LTS**

---

# 2. SSH Into VPS
```bash
ssh root@YOUR_VPS_IP
```

---

# 3. Install Required Packages
```bash
apt update && apt upgrade -y
apt install -y python3 python3-venv python3-pip git nginx certbot python3-certbot-nginx git-lfs ufw
git lfs install
```

Enable firewall:
```bash
ufw allow OpenSSH
ufw allow 80/tcp
ufw allow 443/tcp
ufw enable
```

---

# 4. Clone Repositories to Server
```bash
git clone https://github.com/codewithyasho/ssc-board-rag.git /opt/ssc
cd /opt/ssc
git lfs pull

git clone https://github.com/codewithyasho/RAG-HealthChatbot.git /opt/health
cd /opt/health
git lfs pull
```

---

# 5. Create Python Virtual Environments
### SSC App
```bash
cd /opt/ssc
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
pip install faiss-cpu
deactivate
```

### Health App
```bash
cd /opt/health
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
pip install faiss-cpu
deactivate
```

---

# 6. Create systemd Services (Auto-start on Reboot)

## SSC Service
Create file:
```bash
nano /etc/systemd/system/ssc.service
```

Paste:
```ini
[Unit]
Description=SSC Streamlit App
After=network.target

[Service]
User=root
WorkingDirectory=/opt/ssc
ExecStart=/opt/ssc/.venv/bin/streamlit run /opt/ssc/app.py --server.port 8501 --server.address 0.0.0.0 --server.headless true
Restart=always
RestartSec=5
Environment=PYTHONUNBUFFERED=1

[Install]
WantedBy=multi-user.target
```

---

## Health Service
Create file:
```bash
nano /etc/systemd/system/health.service
```

Paste:
```ini
[Unit]
Description=Health Streamlit App
After=network.target

[Service]
User=root
WorkingDirectory=/opt/health
ExecStart=/opt/health/.venv/bin/streamlit run /opt/health/app.py --server.port 8502 --server.address 0.0.0.0 --server.headless true
Restart=always
RestartSec=5
Environment=PYTHONUNBUFFERED=1

[Install]
WantedBy=multi-user.target
```

Reload systemd & enable:
```bash
systemctl daemon-reload
systemctl enable ssc.service
systemctl start ssc.service

systemctl enable health.service
systemctl start health.service
```

---

# 7. DNS Setup (Name.com)

Create 2 A Records:

| Host | Type | Value |
|------|------|--------|
| ssc | A | VPS_IP |
| health | A | VPS_IP |

---

# 8. Nginx Reverse Proxy

Create config:
```bash
nano /etc/nginx/sites-available/yashodeep.conf
```

Paste:
```nginx
server {
    listen 80;
    server_name ssc.yashodeep.app;

    location / {
        proxy_pass http://127.0.0.1:8501;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}

server {
    listen 80;
    server_name health.yashodeep.app;

    location / {
        proxy_pass http://127.0.0.1:8502;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

Enable:
```bash
ln -s /etc/nginx/sites-available/yashodeep.conf /etc/nginx/sites-enabled/
nginx -t
systemctl restart nginx
```

---

# 9. Enable HTTPS (Required for .app Domains)
```bash
certbot --nginx -d ssc.yashodeep.app -d health.yashodeep.app
```

Choose: **Redirect all traffic to HTTPS**

Test renew:
```bash
certbot renew --dry-run
```

---

# 10. Confirm Everything is Running
```bash
systemctl status ssc.service
systemctl status health.service
```

Check ports:
```bash
ss -ltnp | grep -E ":8501|:8502"
```

Check apps online:

- https://ssc.yashodeep.app  
- https://health.yashodeep.app  

---

# 11. Optional: Add Swap (Recommended for FAISS + LLMs)
```bash
fallocate -l 4G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

---

# 12. Deployment Workflow (After Updates)
```bash
cd /opt/ssc
git pull
systemctl restart ssc.service

cd /opt/health
git pull
systemctl restart health.service
```

---

# 🎉 Deployment Complete
Your two Streamlit RAG apps are now:
- Hosted on VPS  
- Auto-running with systemd  
- Secured with HTTPS  
- Served via Nginx reverse proxy  
- Accessible via subdomains  
- Production-ready  

