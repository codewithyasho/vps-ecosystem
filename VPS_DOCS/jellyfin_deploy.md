# Jellyfin Deployment Guide (classroom.yashodeep.app)

This document contains **all steps and commands** used to deploy your private educational video streaming platform using **Jellyfin** on a VPS.

---

## 1. Server Details
- VPS: Hostinger KVM 2
- OS: Ubuntu
- CPU: 2 cores
- RAM: 8 GB
- Storage: 100 GB
- Domain: classroom.yashodeep.app
- Purpose: Private Netflix-style **educational** video platform

---

## 2. Install Jellyfin

```bash
sudo apt update
sudo apt install -y apt-transport-https curl gnupg
curl -fsSL https://repo.jellyfin.org/install-debuntu.sh | sudo bash
sudo apt install -y jellyfin
```

Enable and start Jellyfin:
```bash
sudo systemctl enable jellyfin
sudo systemctl start jellyfin
```

---

## 3. Create Media Directories

```bash
sudo mkdir -p /srv/media/videos
sudo mkdir -p /srv/media/tutorials
sudo mkdir -p /srv/media/coding
sudo mkdir -p /srv/media/docs
```

---

## 4. Set Permissions (IMPORTANT)

```bash
sudo chown -R jellyfin:jellyfin /srv/media
sudo chmod -R 775 /srv/media
```

This allows Jellyfin to read/write media files.

---

## 5. Upload Videos

### Option A: Using WinSCP (Recommended)
- Protocol: SCP
- Host: VPS IP
- Username: root
- Password: VPS password
- Upload videos to:
  ```
  /srv/media/videos
  ```

### Option B: Using SCP command
```bash
scp *.mp4 root@YOUR_VPS_IP:/srv/media/videos/
```

---

## 6. Organize Videos into Categories

```bash
mv /srv/media/videos/*LangChain* /srv/media/tutorials/
mv /srv/media/videos/*DSA* /srv/media/coding/
mv /srv/media/videos/*RAG* /srv/media/docs/
```

(You can also do this using WinSCP drag & drop)

---

## 7. Jellyfin Initial Setup

Open in browser:
```
http://YOUR_VPS_IP:8096
```

or (after domain setup):
```
https://classroom.yashodeep.app
```

Steps:
1. Create admin user
2. Set language
3. Skip metadata providers (optional)

---

## 8. Add Media Libraries

Go to:
**Dashboard → Libraries → Add Media Library**

### Tutorials
- Content type: Movies
- Name: Tutorials
- Folder: `/srv/media/tutorials`

### Coding
- Content type: Movies
- Name: Coding
- Folder: `/srv/media/coding`

### Docs
- Content type: Movies
- Name: Docs
- Folder: `/srv/media/docs`

Click **Scan All Libraries** after adding.

---

## 9. How to Watch Videos
- Open Jellyfin home page
- Select a library (Tutorials / Coding / Docs)
- Click any video → ▶ Play

Works on:
- Browser
- Mobile
- Smart TV
- Friends (with user access)

---

## 10. Auto Restart (24/7 Service)

Jellyfin runs as a system service:
```bash
sudo systemctl status jellyfin
```

It will:
- Start automatically on reboot
- Run 24/7 without manual start

---

## 11. Optional: Domain + HTTPS (Nginx + SSL)
- Domain: classroom.yashodeep.app
- Reverse proxy via Nginx
- SSL via Certbot (Let’s Encrypt)

---

## 12. Final Result

✔ Private Netflix-style educational platform  
✔ Zero distractions  
✔ Clean UI  
✔ Share with friends  
✔ Fully under your control  

---

**Project Name Suggestions**
- Classroom
- StudyFlix
- EduVault
- LearnBox
- FocusStream

---

Happy Learning 🚀
