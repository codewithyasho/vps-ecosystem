
# VPS Cloud Storage — setup steps (FileBrowser + Nginx + HTTPS + systemd)

**Filename:** `vps_cloud_storage.md`  
**Summary:** Commands and steps that were successfully run to get FileBrowser cloud storage working with nginx reverse proxy, Let's Encrypt TLS, systemd autostart, separate user folders and 10GB upload support.

---

## Prerequisites
- Ubuntu (server) with root/sudo access.
- DNS A record pointing `cloud.yashodeep.app` -> VPS IP.
- `nginx`, `certbot`, `sqlite` tools installed. (Commands shown below.)
- FileBrowser binary downloaded to `/opt/filebrowser`.

---

## 0. Useful packages (if missing)
```bash
sudo apt update
sudo apt install -y nginx certbot python3-certbot-nginx
```

---

## 1. Create storage directories and user
```bash
# create mount/storage root
sudo mkdir -p /srv/cloud-data/yashodeep
sudo mkdir -p /srv/cloud-data/vyanku

# create system user to run filebrowser and set ownership
sudo useradd --system --home /srv/cloud-data --shell /usr/sbin/nologin filebrowser || true
sudo chown -R filebrowser:filebrowser /srv/cloud-data
```

---

## 2. Put FileBrowser binary in place
(You already have `/opt/filebrowser` and `filebrowser.db` in `/opt/filebrowser`.)

If not, example:
```bash
# example download (replace with actual latest link)
sudo mkdir -p /opt/filebrowser
# download and make executable:
# sudo wget -O /opt/filebrowser/filebrowser https://github.com/filebrowser/filebrowser/releases/download/vX.Y.Z/filebrowser-linux-amd64
sudo chmod +x /opt/filebrowser/filebrowser
```

---

## 3. systemd unit for FileBrowser
Create `/etc/systemd/system/filebrowser.service` with these contents:
```ini
[Unit]
Description=File Browser
After=network.target

[Service]
User=filebrowser
Group=filebrowser
ExecStart=/opt/filebrowser/filebrowser --database /opt/filebrowser/filebrowser.db --root /srv/cloud-data --port 8080
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Reload and start:
```bash
sudo systemctl daemon-reload
sudo systemctl enable filebrowser
sudo systemctl start filebrowser
sudo systemctl status filebrowser --no-pager
```

- If you see errors, check logs:
```bash
sudo journalctl -u filebrowser -n 200 --no-pager
```

- Common problems:
  - Permission denied for `/opt/filebrowser/filebrowser.db` → `sudo chown filebrowser:filebrowser /opt/filebrowser/filebrowser.db` or run FileBrowser with correct `--database` path and permissions.
  - `too many colons in address` → use `--port 8080` or `--address 127.0.0.1` (not both with duplicate syntax).

---

## 4. Test local FileBrowser
From the VPS:
```bash
# confirm process listening
ss -ltpn | grep ":8080\b"

# quick curl test
curl -v http://127.0.0.1:8080/
```

You should see HTML returned.

---

## 5. Nginx reverse proxy (HTTP -> FileBrowser)
Create a site file `/etc/nginx/sites-available/cloud.yashodeep.app` (example):
```nginx
server {
    listen 80;
    server_name cloud.yashodeep.app www.cloud.yashodeep.app;

    client_max_body_size 10G;    # allow large uploads

    location / {
        proxy_pass http://127.0.0.1:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 3600s;
    }
}
```
Enable and test:
```bash
sudo ln -s /etc/nginx/sites-available/cloud.yashodeep.app /etc/nginx/sites-enabled/cloud.yashodeep.app
sudo nginx -t
sudo systemctl restart nginx
```

---

## 6. Obtain and deploy TLS (Certbot)
Run certbot nginx plugin:
```bash
sudo certbot --nginx -d cloud.yashodeep.app -d www.cloud.yashodeep.app
```
Certbot will:
- verify your domain,
- obtain certificate,
- update your nginx site to use the certificate,
- and reload nginx.

Confirm HTTPS works in browser at `https://cloud.yashodeep.app`.

---

## 7. FileBrowser users (create admin and normal user)
Stop service before writing DB to avoid lock:
```bash
sudo systemctl stop filebrowser
```

Add admin user (example):
```bash
# wrap password in single quotes if it has special chars
sudo /opt/filebrowser/filebrowser users add yashodeep 'Yashodeep@2006' --perm.admin --database /opt/filebrowser/filebrowser.db
```

Add normal friend user:
```bash
sudo /opt/filebrowser/filebrowser users add vyanku 'somepassword' --database /opt/filebrowser/filebrowser.db
```

Verify list:
```bash
sudo /opt/filebrowser/filebrowser users list --database /opt/filebrowser/filebrowser.db
```

Start again:
```bash
sudo systemctl start filebrowser
sudo systemctl status filebrowser --no-pager
```

---

## 8. Per-user scopes (isolation)
By default FileBrowser will show the same root to all users. To restrict each user to their own folder:

1. Create per-user folders under `/srv/cloud-data/` (already done earlier):
```bash
sudo mkdir -p /srv/cloud-data/yashodeep
sudo mkdir -p /srv/cloud-data/vyanku
sudo chown -R filebrowser:filebrowser /srv/cloud-data
```

2. In the FileBrowser web UI as an admin:
   - Go to **Settings → User Management**.
   - Edit each user and set **Scope** to the user's subfolder (e.g. `/yashodeep` for user `yashodeep`, `/vyanku` for user `vyanku`).
   - Save.

This ensures:
- `yashodeep` sees only `/yashodeep`
- `vyanku` sees only `/vyanku`

(You already performed these steps through the web UI.)

---

## 9. Increase upload limits (chunked upload + nginx)
- Nginx: set `client_max_body_size 10G;` in `/etc/nginx/nginx.conf` or site file (already done).
- FileBrowser UI: **Settings → Chunked Uploads** → set `10GB` (or desired limit) and save.

Don't forget nginx restart:
```bash
sudo systemctl restart nginx
```

---

## 10. Auto-start on reboot
We enabled systemd `filebrowser.service`. To confirm all services auto-start on reboot:
```bash
sudo systemctl enable filebrowser
sudo systemctl is-enabled filebrowser
```
Other services (VPN, chatbots) must also have systemd services with `WantedBy=multi-user.target` and enabled similarly to start at boot.

---

## 11. Troubleshooting
- Check service logs:
```bash
sudo journalctl -u filebrowser -n 200 --no-pager
sudo journalctl -u nginx -n 200 --no-pager
```
- Check nginx config:
```bash
sudo nginx -t
```
- Permission issues:
```bash
sudo chown -R filebrowser:filebrowser /opt/filebrowser /srv/cloud-data
sudo chmod -R 750 /srv/cloud-data
```
- If DB lock prevents user creation: stop the service before adding a user.

---

## 12. Quick checklist — what you now have
- FileBrowser running under systemd on `127.0.0.1:8080`
- Nginx reverse proxy with valid Let's Encrypt TLS (HTTPS)
- Per-user folders and user-scoped access
- 10GB chunked upload support
- FileBrowser auto-starts on reboot via systemd

---

## 13. Useful commands (cheat-sheet)
```bash
# service control
sudo systemctl start|stop|restart filebrowser
sudo systemctl status filebrowser --no-pager

# nginx
sudo nginx -t
sudo systemctl restart nginx

# certbot renew (manual)
sudo certbot renew --dry-run

# view logs
sudo journalctl -u filebrowser -n 200 --no-pager
```

---

If you want, I can:
- produce a PDF version,
- create separate README or a combined VPS setup guide (with VPN, chatbots),
- or save this file for you to download right now.

