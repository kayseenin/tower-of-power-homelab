# Homelab Infrastructure Project

![GitHub last commit](https://img.shields.io/github/last-commit/YOUR-USERNAME/homelab)
![GitHub repo size](https://img.shields.io/github/repo-size/YOUR-USERNAME/homelab)
![GitHub stars](https://img.shields.io/github/stars/YOUR-USERNAME/homelab?style=social)

A self-hosted infrastructure setup for learning modern DevOps practices, containerization, network security, and remote access technologies.

![Status](https://img.shields.io/badge/status-active-success)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![Raspberry Pi](https://img.shields.io/badge/Raspberry%20Pi-A22846?style=flat&logo=Raspberry%20Pi&logoColor=white)
![OpenWRT](https://img.shields.io/badge/OpenWRT-00B5E2?style=flat&logo=openwrt&logoColor=white)

## 🎯 Project Goals

- Learn container orchestration with Docker and Portainer
- Implement network-wide ad blocking and privacy protection
- Enable secure remote access without exposing services to the internet
- Build automated monitoring and alerting systems
- Practice Infrastructure as Code principles
- Gain hands-on experience with tools used in production environments

---

## 🏗️ Current Architecture

### Hardware
- **Mac Mini M4** - Primary server running Docker and Portainer
- **Raspberry Pi** - Dedicated DNS/ad-blocking server (planned)
- **GL.iNet GL-MT3000** - OpenWRT travel router with VPN capabilities (planned)
- **MikroTik Managed Switch** - Enterprise VLAN support (planned)
- **Nvidia Shield Pro 2019** - Primary streaming device
- **HP Omen 16** - Game streaming host with Sunshine (planned)

### Software Stack (Current)
- **Docker** - Container runtime
- **Portainer** - Container management web UI
- **Uptime Kuma** - Service monitoring and status dashboard
- **Tailscale** - Zero-trust mesh VPN (planned)
- **Healthchecks.io** - External monitoring with dead man's switch

---

## 🚀 What's Running

| Service | Purpose | Port | Status |
|---------|---------|------|--------|
| Portainer | Container Management | 9000 | ✅ Running |
| Uptime Kuma | Service Monitoring | 3001 | ✅ Running |

---

## 📊 Current Setup Progress

### ✅ Completed (Phase 1)
- [x] Mac Mini configured for remote SSH access from MacBook
- [x] Docker installed and configured on Mac Mini
- [x] Portainer deployed for container management
- [x] Uptime Kuma deployed for service monitoring
- [x] Healthchecks.io configured with cron-based heartbeat monitoring
- [x] Basic monitoring setup (HTTP and Ping monitors)
- [x] Container orchestration via docker-compose

### 🔄 In Progress (Phase 2)
- [ ] Raspberry Pi setup with Pi-hole for DNS-based ad blocking
- [ ] Tailscale mesh VPN deployment across all devices
- [ ] OpenWRT router configuration with VLANs
- [ ] MikroTik switch VLAN configuration

### 📋 Planned (Phase 3+)
- [ ] VPN policy-based routing for F1 TV bypass
- [ ] Game streaming setup (Sunshine/Moonlight)
- [ ] Homepage dashboard deployment
- [ ] Automated captive portal authentication for travel
- [ ] Media server (Jellyfin/Plex)
- [ ] File synchronization (Nextcloud)
- [ ] Advanced network monitoring (Prometheus + Grafana)

---

## 📚 Documentation

- [Getting Started Guide](docs/getting-started.md) - Initial setup walkthrough
- [Docker Setup](docs/docker-setup.md) - Docker and Portainer configuration
- [Monitoring Setup](docs/monitoring-setup.md) - Uptime Kuma and Healthchecks configuration
- [Lessons Learned](docs/lessons-learned.md) - Challenges and solutions

---

## 🛠️ Quick Start

### Prerequisites
- Mac Mini (or any server) with macOS or Linux
- Basic familiarity with terminal/SSH
- Docker installed

### Initial Setup

1. **Enable Remote Login on Mac Mini:**
   ```bash
   # On Mac Mini: System Settings → General → Sharing → Remote Login
   ```

2. **SSH from your MacBook:**
   ```bash
   ssh username@mac-mini-ip
   ```

3. **Install Docker:**
   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   brew install --cask docker
   open -a Docker
   ```

4. **Deploy Portainer:**
   ```bash
   docker volume create portainer_data
   docker run -d -p 9000:9000 --name portainer --restart always \
     -v /var/run/docker.sock:/var/run/docker.sock \
     -v portainer_data:/data \
     portainer/portainer-ce:latest
   ```

5. **Access Portainer:**
   - Navigate to `http://mac-mini-ip:9000`
   - Create admin credentials
   - Select local Docker environment

---

## 🔍 Monitoring & Alerting

### Internal Monitoring
- **Uptime Kuma** running on Mac Mini monitors all local services
- Accessible at `http://mac-mini-ip:3001`

### External Monitoring
- **Healthchecks.io** monitors Mac Mini availability via cron heartbeat
- Pings every 5 minutes
- Email alerts if heartbeat stops

### Current Monitors
- Portainer web interface (HTTP)
- Uptime Kuma web interface (HTTP)
- Mac Mini availability (Ping)
- Mac Mini SSH access (Port 22)

---

## 🎓 Skills Demonstrated

### DevOps & Infrastructure
- Container orchestration (Docker)
- Infrastructure as Code (docker-compose)
- Remote system administration (SSH)
- Service monitoring and alerting
- Dead man's switch implementation

### Networking (Planned)
- VLAN configuration and network segmentation
- VPN setup and policy-based routing
- DNS architecture and ad blocking
- Firewall configuration

### Automation
- Cron job scheduling
- Automated health checks
- Container auto-updates (planned)

---

## 🔮 Future Enhancements

**Short Term:**
- Deploy Homepage dashboard for unified service access
- Set up Pi-hole on Raspberry Pi
- Configure Tailscale for remote access
- Add Dozzle for container log viewing

**Medium Term:**
- Implement OpenWRT with VLANs (Management, Trusted, IoT, Lab, Guest)
- Configure policy-based VPN routing
- Set up game streaming with Sunshine/Moonlight
- Add automated backups

**Long Term:**
- Jellyfin media server
- Nextcloud file synchronization
- Prometheus + Grafana monitoring stack
- Home Assistant integration
- Automated captive portal bypass for travel

---

## 📝 Development Log

### 2025-02-13 - Initial Setup
- Established SSH connectivity between MacBook and Mac Mini
- Installed Docker and Portainer on Mac Mini
- Deployed Uptime Kuma for service monitoring
- Configured Healthchecks.io external monitoring with cron
- Created initial monitoring dashboards

---

## 🤝 Acknowledgments

This project was developed with guidance from Claude (Anthropic) for architecture planning, troubleshooting, and documentation.

---

## 📄 License

MIT License - Feel free to use this as inspiration for your own homelab!

---

## 📬 Contact

Questions or suggestions? Open an issue in this repository!
