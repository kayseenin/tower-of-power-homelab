# Getting Started Guide

This guide walks through the initial setup of the homelab infrastructure from scratch.

## Prerequisites

- Mac Mini M4 (or any macOS/Linux server)
- MacBook or laptop for remote management
- Basic terminal knowledge
- Network connectivity between devices

---

## Step 1: Enable Remote Access

### On Mac Mini

1. Open **System Settings**
2. Navigate to **General → Sharing**
3. Enable **Remote Login**
4. Ensure your user account is in the "Allow access for" list
5. Note the IP address shown (e.g., `192.168.1.50`)

### From Your MacBook

Test SSH connectivity:

```bash
ssh yourusername@mac-mini-ip
# Enter your password when prompted
```

If successful, you should see the Mac Mini's terminal prompt.

---

## Step 2: Install Homebrew (Package Manager)

On Mac Mini via SSH:

```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Follow the post-install instructions to add Homebrew to PATH
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"

# Verify installation
brew --version
```

---

## Step 3: Install Docker Desktop

```bash
# Install Docker via Homebrew
brew install --cask docker

# Launch Docker Desktop
open -a Docker

# Wait for Docker to start (30 seconds)
sleep 30

# Verify Docker is running
docker --version
docker ps
```

You should see Docker version info and an empty container list.

---

## Step 4: Deploy Portainer

Portainer provides a web UI for managing Docker containers.

```bash
# Create persistent storage for Portainer
docker volume create portainer_data

# Run Portainer container
docker run -d \
  -p 9000:9000 \
  --name portainer \
  --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest

# Verify Portainer is running
docker ps | grep portainer
```

### Access Portainer

1. On your MacBook, open browser to: `http://mac-mini-ip:9000`
2. **First-time setup:**
   - Create admin username and password (save these!)
   - Click "Create user"
3. Click "Get Started"
4. Select "Local" Docker environment
5. You should now see the Portainer dashboard

---

## Step 5: Deploy Uptime Kuma (Monitoring)

### Via Portainer Web UI

1. Navigate to **Stacks** → **Add stack**
2. Name: `uptime-kuma`
3. **Web editor**, paste this:

```yaml
version: '3.8'

services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    volumes:
      - uptime-kuma:/app/data
      - /var/run/docker.sock:/var/run/docker.sock:ro
    ports:
      - 3001:3001
    restart: unless-stopped

volumes:
  uptime-kuma:
```

4. Click **Deploy the stack**
5. Wait for deployment to complete

### Access Uptime Kuma

1. Navigate to: `http://mac-mini-ip:3001`
2. **First-time setup:**
   - Create admin username and password (save these!)
   - Click "Create"
3. You're now on the monitoring dashboard

---

## Step 6: Configure Monitoring

### Add Your First Monitor in Uptime Kuma

**Monitor Portainer:**

1. Click **"+ Add New Monitor"**
2. Fill in:
   - Monitor Type: `HTTP(s)`
   - Friendly Name: `Portainer`
   - URL: `http://mac-mini-ip:9000`
   - Heartbeat Interval: `60` seconds
3. Click **Save**

You should see it turn green with "Up" status!

**Add More Monitors:**

Repeat for these:

| Monitor Type | Friendly Name | URL/Host | Notes |
|--------------|---------------|----------|-------|
| HTTP(s) | Uptime Kuma | http://mac-mini-ip:3001 | Monitor itself |
| Ping | Mac Mini | mac-mini-ip | Device availability |
| Port | Mac Mini SSH | mac-mini-ip:22 | SSH access |

---

## Step 7: Set Up External Monitoring

Since Uptime Kuma runs ON the Mac Mini, if the Mac Mini goes down, you won't know. Use external monitoring as a backup.

### Configure Healthchecks.io

1. Go to https://healthchecks.io
2. Create free account
3. Create a new check:
   - Name: `Mac Mini Heartbeat`
   - Period: `5 minutes`
   - Grace: `2 minutes`
4. Copy the ping URL (looks like: `https://hc-ping.com/abc123-def456`)

### Set Up Cron Job on Mac Mini

SSH to Mac Mini and create a cron job:

```bash
# Open crontab editor
crontab -e

# Press 'i' to enter insert mode
# Add this line (replace with YOUR healthchecks URL):
*/5 * * * * curl -fsS --retry 3 https://hc-ping.com/YOUR-UNIQUE-ID > /dev/null

# Press Esc
# Type :wq and press Enter to save
```

**Test it:**

```bash
# Run the curl command manually
curl -fsS --retry 3 https://hc-ping.com/YOUR-UNIQUE-ID

# Check healthchecks.io dashboard
# Should show "Last Ping: just now"
```

---

## Step 8: Verify Everything Works

### Checklist

- [ ] Can SSH to Mac Mini from MacBook
- [ ] Docker is running: `docker ps` works
- [ ] Portainer accessible at `:9000`
- [ ] Uptime Kuma accessible at `:3001`
- [ ] Monitors showing "Up" in Uptime Kuma
- [ ] Healthchecks.io receiving pings every 5 minutes

---

## Troubleshooting

### Can't SSH to Mac Mini
- Check Remote Login is enabled in System Settings
- Verify IP address is correct
- Try: `ssh -v username@ip` for verbose output

### Docker commands fail
- Make sure Docker Desktop is running
- Check: `open -a Docker`
- Verify: `docker --version`

### Can't access Portainer
- Check container is running: `docker ps | grep portainer`
- Verify port 9000 isn't blocked
- Try from Mac Mini itself: `curl localhost:9000`

### Uptime Kuma monitors show "Down"
- Verify URLs are exactly correct
- Check containers are running: `docker ps`
- Ensure firewall isn't blocking ports

### Cron job not working
- Check cron syntax: `crontab -l`
- Test curl command manually
- Check system logs: `grep CRON /var/log/system.log`

---

## Next Steps

Now that you have the foundation:

1. **Add more services** - Homepage dashboard, Dozzle logs viewer, etc.
2. **Set up Raspberry Pi** - Deploy Pi-hole for network-wide ad blocking
3. **Configure Tailscale** - Access homelab remotely from anywhere
4. **Implement VLANs** - Segment your network with OpenWRT and MikroTik

Continue to the [Docker Setup](docker-setup.md) guide for more advanced container configurations.

---

## Quick Reference

### Useful Commands

```bash
# SSH to Mac Mini
ssh username@mac-mini-ip

# Check running containers
docker ps

# View all containers (including stopped)
docker ps -a

# View container logs
docker logs container-name

# Restart a container
docker restart container-name

# Stop a container
docker stop container-name

# Remove a container
docker rm container-name

# View Docker volumes
docker volume ls

# Access Portainer
http://mac-mini-ip:9000

# Access Uptime Kuma
http://mac-mini-ip:3001
```

### Default Ports

| Service | Port |
|---------|------|
| SSH | 22 |
| Portainer | 9000 |
| Uptime Kuma | 3001 |

---

## Security Notes

- Change default passwords immediately
- Keep SSH access limited to your local network (or use Tailscale)
- Don't expose Docker or Portainer to the internet without proper authentication
- Regularly update Docker images for security patches
- Use strong, unique passwords for all services

---

**Congratulations!** You now have a functioning homelab with remote management, containerization, and monitoring. 🎉
