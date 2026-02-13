# Homelab Quick Reference

Essential commands and URLs for daily homelab management.

---

## 🔗 Access URLs

| Service | URL | Credentials |
|---------|-----|-------------|
| Portainer | http://10.69.10.3:9000 | admin / [saved in password manager] |
| Uptime Kuma | http://10.69.10.3:3001 | admin / [saved in password manager] |
| Healthchecks.io | https://healthchecks.io | [account email] |

**Replace `10.69.10.3` with your actual Mac Mini IP**

---

## 💻 SSH Access

```bash
# Connect to Mac Mini
ssh username@10.69.10.3

# Or using hostname (if mDNS works)
ssh username@mac-mini.local
```

---

## 🐳 Docker Commands

### Container Management

```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# View container logs
docker logs container-name

# Follow logs in real-time
docker logs -f container-name

# Restart a container
docker restart container-name

# Stop a container
docker stop container-name

# Start a stopped container
docker start container-name

# Remove a container (must be stopped first)
docker stop container-name && docker rm container-name

# Execute command in running container
docker exec -it container-name /bin/bash
```

### Stack Management

```bash
# Deploy stack from compose file
docker-compose up -d

# Stop and remove stack
docker-compose down

# View stack logs
docker-compose logs

# Pull latest images
docker-compose pull

# Rebuild and restart
docker-compose up -d --build
```

### Volume Management

```bash
# List volumes
docker volume ls

# Inspect volume
docker volume inspect volume-name

# Remove volume (container must be stopped)
docker volume rm volume-name

# Remove all unused volumes
docker volume prune
```

### Cleanup

```bash
# Remove all stopped containers
docker container prune

# Remove unused images
docker image prune -a

# Remove unused volumes
docker volume prune

# Remove everything unused (nuclear option)
docker system prune -a --volumes
```

---

## 📊 Monitoring

### Check Service Status

```bash
# Via Docker
docker ps

# Via curl (check if service responds)
curl http://localhost:9000      # Portainer
curl http://localhost:3001      # Uptime Kuma

# Via ping
ping 10.69.10.3                # Mac Mini
```

### Uptime Kuma Monitors

| Monitor | Type | Target |
|---------|------|--------|
| Portainer | HTTP | http://10.69.10.3:9000 |
| Uptime Kuma | HTTP | http://10.69.10.3:3001 |
| Mac Mini | Ping | 10.69.10.3 |
| Mac Mini SSH | Port | 10.69.10.3:22 |

### Healthchecks.io

```bash
# Manual ping
curl -fsS https://hc-ping.com/YOUR-UNIQUE-ID

# View cron jobs
crontab -l

# Edit cron jobs
crontab -e
```

---

## 🔧 Troubleshooting

### Container Won't Start

```bash
# Check logs for errors
docker logs container-name

# Check if port is already in use
lsof -i :PORT_NUMBER

# Verify image exists
docker images | grep image-name

# Try manual run for debugging
docker run -it --rm image-name /bin/bash
```

### Can't Access Web Interface

```bash
# Check container is running
docker ps | grep container-name

# Check container ports
docker port container-name

# Test from Mac Mini itself
curl http://localhost:PORT

# Check firewall (macOS rarely blocks)
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate
```

### Cron Job Not Running

```bash
# List cron jobs
crontab -l

# Check cron is running
ps aux | grep cron

# View cron logs (macOS)
log show --predicate 'process == "cron"' --last 1h
```

### Docker Out of Space

```bash
# Check disk usage
docker system df

# Clean up everything
docker system prune -a --volumes

# Check macOS disk space
df -h
```

---

## 📝 Common Tasks

### Adding a New Monitor in Uptime Kuma

1. Open Uptime Kuma: http://10.69.10.3:3001
2. Click "+ Add New Monitor"
3. Select monitor type (HTTP/Ping/Port)
4. Fill in details
5. Click "Save"

### Deploying New Container

**Via Portainer:**
1. Stacks → Add stack
2. Name it
3. Paste docker-compose YAML
4. Deploy

**Via CLI:**
```bash
# From docker-compose file
docker-compose up -d

# Single container
docker run -d --name name image
```

### Updating a Container

**Via Portainer:**
1. Containers → Select container
2. Recreate → Pull latest image

**Via CLI:**
```bash
docker pull image:tag
docker stop container-name
docker rm container-name
docker run ...  # same command as before
```

### Backing Up Data

```bash
# Backup a volume
docker run --rm \
  -v volume-name:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/backup.tar.gz -C /data .

# Restore a volume
docker run --rm \
  -v volume-name:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/backup.tar.gz -C /data
```

---

## 🌐 Network Information

### Current Setup

| Device | IP Address | Purpose |
|--------|------------|---------|
| Mac Mini | 10.69.10.3 | Docker host |
| Raspberry Pi | 10.69.10.2 | Pi-hole (planned) |
| OpenWRT Router | 192.168.8.1 | Network gateway (planned) |
| MacBook | Dynamic | Management device |

### Ports in Use

| Port | Service |
|------|---------|
| 22 | SSH |
| 3001 | Uptime Kuma |
| 9000 | Portainer |

---

## 📦 Installed Services

### Currently Running

- ✅ Docker Engine
- ✅ Portainer (container management)
- ✅ Uptime Kuma (monitoring)

### Planned

- ⏳ Pi-hole (DNS ad blocking)
- ⏳ Tailscale (VPN mesh)
- ⏳ Homepage (dashboard)
- ⏳ Jellyfin (media server)

---

## 🔐 Security Reminders

- [ ] All services use strong passwords
- [ ] Passwords stored in password manager
- [ ] SSH key-based auth (optional but recommended)
- [ ] Services not exposed to internet
- [ ] Regular backups configured
- [ ] Docker socket only mounted to trusted containers

---

## 📚 Documentation Links

- [Getting Started Guide](docs/getting-started.md)
- [Docker Setup](docs/docker-setup.md)
- [Monitoring Setup](docs/monitoring-setup.md)
- [Lessons Learned](docs/lessons-learned.md)

---

## 🆘 When Things Break

1. **Check logs:** `docker logs container-name`
2. **Check it's running:** `docker ps`
3. **Restart it:** `docker restart container-name`
4. **Check the docs:** Look at this repository
5. **Google it:** Include exact error message
6. **Ask Claude:** Describe the issue in detail

---

## 📊 Health Check

Run this to verify everything is working:

```bash
#!/bin/bash
echo "=== Docker Status ==="
docker ps

echo -e "\n=== Uptime Kuma ==="
curl -s http://localhost:3001 > /dev/null && echo "✅ Up" || echo "❌ Down"

echo -e "\n=== Portainer ==="
curl -s http://localhost:9000 > /dev/null && echo "✅ Up" || echo "❌ Down"

echo -e "\n=== Healthchecks.io Ping ==="
curl -fsS https://hc-ping.com/YOUR-UNIQUE-ID && echo "✅ Pinged" || echo "❌ Failed"

echo -e "\n=== Disk Space ==="
df -h | grep -v tmpfs | grep -v devfs
```

Save as `health-check.sh`, make executable, run when needed.

---

*Last updated: 2025-02-13*
