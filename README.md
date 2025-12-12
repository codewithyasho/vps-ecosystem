# 🚀 Self-Hosted VPS Ecosystem

This repository documents a **production-ready, self-hosted VPS ecosystem** built and managed on a single Linux VPS.  
The goal of this project was to **learn real-world backend, DevOps, and system deployment** by building and running multiple live services end-to-end.

All services are **live, secured with HTTPS, auto-restart on reboot, and accessible via custom subdomains**.

---

## 🌐 Live Services

| Service | Description | URL |
|------|------------|-----|
| SSC Tutor | Online tutoring platform for SSC exam preparation | https://ssc.yashodeep.app |
| Health Chatbot | AI-powered health assistance chatbot | https://health.yashodeep.app |
| Cloud Storage | Self-hosted file storage (Google Drive–like) | https://cloud.yashodeep.app |
| VPN Service | Private VPN for secure internet access | https://vpn.yashodeep.app |
| Classroom | Netflix-style educational video streaming platform | https://classroom.yashodeep.app |

---

## 🧠 Motivation For Building Own Classroom: https://classroom.yashodeep.app

While studying on public platforms like YouTube, distractions significantly reduced focus.  
This project was built to create a **private, distraction-free learning ecosystem** with full control over content, security, and availability.

---

## 🛠️ Tech Stack

- **Operating System:** Ubuntu Linux (VPS)
- **Web Server / Proxy:** Nginx
- **SSL / HTTPS:** Let’s Encrypt (Certbot)
- **Containerization:** Docker
- **Backend Framework:** Flask
- **Media Server:** Jellyfin
- **VPN:** WireGuard / OpenVPN
- **File Storage:** FileBrowser
- **Process Management:** systemd
- **Networking:** Reverse Proxy, Subdomains, Firewall rules

---

## 🏗️ System Architecture (High Level)

```
Internet
   │
   ▼
Nginx (Reverse Proxy + SSL)
   │
   ├── ssc.yashodeep.app        → SSC Tutor (Flask)
   ├── health.yashodeep.app     → Health Chatbot (Flask + AI)
   ├── cloud.yashodeep.app      → FileBrowser
   ├── vpn.yashodeep.app        → VPN Server
   └── classroom.yashodeep.app  → Jellyfin Media Server
```

Each service runs independently and is isolated at the application level.

---

## ⚙️ Key Features

- Multiple production services on a **single VPS**
- Custom subdomains for each service
- HTTPS enabled for all services
- Auto-start on system reboot
- Linux permission and storage management
- Secure VPN access
- Media streaming without third-party platforms
- Private cloud storage with user separation

---

## 📂 Storage Structure

```
/srv/
 └── media/
     ├── tutorials/
     ├── coding/
     ├── docs/
```
All media permissions are managed to allow Jellyfin read/write access.

---

## 🔄 Service Reliability

- All services are registered with **systemd** or Docker restart policies
- Services automatically restart after:
  - VPS reboot
  - Unexpected crashes
- Nginx and SSL certificates renew automatically

---

## 🔐 Security Practices

- HTTPS on all endpoints
- Reverse proxy isolation
- Private VPN access
- Linux user/group permission control
- Firewall rules to expose only required ports

---

## 📈 What I Learned

- Linux server administration
- Reverse proxy & SSL configuration
- Running multiple services on one VPS
- Networking and VPN fundamentals
- Media streaming infrastructure
- Production deployment mindset
- Debugging real server issues

---

## 🎯 Future Improvements

- Centralized authentication (SSO)
- Monitoring and uptime alerts
- Automated backups
- Docker Compose for all services
- CI/CD pipeline integration

---

## 👤 Author

**Yashodeep Hundiwale**  
Computer Science Student  
Interests: Data Science, AI ML, Backend Development, DevOps, System Design

---

## ⭐ Final Note

This project was built as a **learning-by-building initiative** and reflects real-world system deployment rather than theoretical examples.

Feel free to explore, get inspired, or adapt ideas from this ecosystem.
