# Complete Homelab + Game Streaming Setup - Master Plan
## "Barney Style" - Step by Step, Nothing Assumed

---

## 🎯 What You're Building

A complete home infrastructure that lets you:
- Block ads across your entire network (Pi-hole on Raspberry Pi)
- Manage services through a web interface (Portainer on Mac Mini)
- Access everything securely from anywhere (Tailscale VPN)
- **Play your full Steam library on your MacBook from anywhere** (Sunshine/Moonlight streaming)
- Build an impressive GitHub portfolio showing real DevOps skills

## 📦 Your Equipment

**In your GeeekPi T1 server rack:**
- Mac Mini M4 (runs Docker + Portainer)
- Raspberry Pi (runs Pi-hole for ad blocking)

**Separate from rack:**
- HP Omen 16 gaming laptop (will be game streaming host)
- MacBook M4 Air (your control device + game streaming client)
- Router/modem (your network)

---

## 🗺️ The Master Plan

### Phase 1: Core Network Infrastructure
Set up the foundation - Pi-hole and Portainer

### Phase 2: Remote Access with Tailscale
Connect everything so you can access it from anywhere

### Phase 3: Game Streaming Setup
Add Sunshine/Moonlight so you can game from bed

### Phase 4: GitHub Portfolio
Document everything to showcase your skills

---

# PHASE 1: CORE NETWORK INFRASTRUCTURE

## Step 1.1: Set Up the Raspberry Pi

### What you're doing:
Getting the Raspberry Pi ready to run Pi-hole (network-wide ad blocker)

### How to do it:

**1. Flash the microSD card (from your MacBook):**

a. Download Raspberry Pi Imager: https://www.raspberrypi.com/software/
b. Insert your microSD card into MacBook
c. Open Raspberry Pi Imager
d. Click "Choose Device" → pick your Raspberry Pi model
e. Click "Choose OS" → pick "Raspberry Pi OS Lite (64-bit)"
f. Click "Choose Storage" → pick your microSD card
g. Click "Next"
h. When it asks "Would you like to apply OS customization settings?" click "Edit Settings"

**2. Configure the settings:**

In the General tab:
- Hostname: `pihole`
- Username: `pi` (or whatever you want)
- Password: (create one and WRITE IT DOWN)
- If using WiFi: enter your WiFi name and password
- Timezone: pick yours

In the Services tab:
- Turn on "Enable SSH"
- Pick "Use password authentication"

Click "Save" → "Yes" → "Yes"

**3. Wait for it to finish (5-10 minutes)**

**4. Remove the microSD card and put it in your Raspberry Pi**

**5. Plug in the Raspberry Pi and wait 2-3 minutes for it to boot**

---

## Step 1.2: Connect to Your Raspberry Pi

### What you're doing:
Logging into the Pi remotely from your MacBook so you can control it

### How to do it:

**1. Find the Pi's IP address**

Option A - Scan from MacBook:
```bash
# Open Terminal on MacBook
arp -a | grep -i "b8:27\|dc:a6\|e4:5f"
```

Option B - Check your router:
- Log into your router (usually 192.168.1.1 or 192.168.0.1)
- Look for "Connected Devices" or similar
- Find "pihole" or "raspberrypi"
- Write down its IP (like 192.168.1.100)

**2. Connect via SSH**

```bash
# In Terminal on MacBook, type:
ssh pi@192.168.1.100
# (replace 192.168.1.100 with YOUR Pi's IP)

# Type "yes" when it asks about authenticity
# Enter the password you created earlier
```

You're now inside the Raspberry Pi! Everything you type goes to the Pi.

**3. Update the Pi**

```bash
# Type these commands one at a time:
sudo apt update
sudo apt upgrade -y
sudo reboot
```

Wait 1 minute, then SSH back in (same command as step 2 above).

---

## Step 1.3: Install Pi-hole

### What you're doing:
Installing Pi-hole, which will block ads for your entire network

### How to do it:

**1. Still in SSH to the Pi, run this:**

```bash
curl -sSL https://install.pi-hole.net | bash
```

**2. During installation:**

- Press Enter on all the intro screens
- Network interface: pick `eth0` (ethernet) or `wlan0` (WiFi)
- Upstream DNS: pick Cloudflare (or doesn't really matter)
- Blocklists: Yes
- Admin interface: Yes
- Web server: Yes
- Logging: Yes
- Privacy mode: "Show everything" (or your choice)

**3. IMPORTANT: At the end, it shows a password - WRITE THIS DOWN!**

This is your Pi-hole admin password.

**4. Test it**

On your MacBook, open a browser and go to:
```
http://192.168.1.100/admin
```
(replace with your Pi's IP)

You should see the Pi-hole dashboard!

---

## Step 1.4: Set Up the Mac Mini

### What you're doing:
Preparing the Mac Mini to run Docker containers

### How to do it:

**1. On the Mac Mini itself (one time only):**

- Go to System Settings → General → Sharing
- Turn on "Remote Login"
- Make sure your user is allowed to connect
- Write down the Mac Mini's IP address (shown at the top)

**2. From your MacBook, SSH into the Mac Mini:**

```bash
# In Terminal on MacBook:
ssh yourusername@192.168.1.50
# (replace with YOUR Mac Mini's username and IP)

# Enter your Mac Mini password
```

You're now controlling the Mac Mini remotely!

**3. Install Homebrew (package manager):**

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# It will tell you to run some commands to add it to your PATH
# They look like this (run them):
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

**4. Install Docker:**

```bash
brew install --cask docker
open -a Docker
sleep 30  # Wait for Docker to start
```

**5. Install Portainer (container management):**

```bash
# Create storage for Portainer
docker volume create portainer_data

# Run Portainer
docker run -d -p 9000:9000 --name portainer --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

**6. Access Portainer**

On your MacBook browser, go to:
```
http://192.168.1.50:9000
```
(replace with your Mac Mini's IP)

- Create a username and password (WRITE THESE DOWN)
- Click "Get Started"
- Click on the local Docker environment

You now have a web interface to manage Docker containers!

---

## Step 1.5: Make Your Network Use Pi-hole

### What you're doing:
Making all your devices use Pi-hole for DNS, so ads get blocked everywhere

### How to do it:

**Option A: Test on just your MacBook first (recommended)**

1. On MacBook: System Settings → Network
2. Click your connection (WiFi or Ethernet) → Details
3. Click DNS tab
4. Click the + button
5. Add your Pi's IP (192.168.1.100)
6. Drag it to the top of the list
7. Click OK

Now visit some websites and check the Pi-hole dashboard - you should see queries!

**Option B: Enable for whole network**

1. Log into your router (usually 192.168.1.1)
2. Find DHCP or DNS settings
3. Set Primary DNS to your Pi's IP (192.168.1.100)
4. Set Secondary DNS to 8.8.8.8 (backup)
5. Save and restart router

Now ALL devices use Pi-hole!

---

# PHASE 2: REMOTE ACCESS WITH TAILSCALE

## Step 2.1: Install Tailscale on Mac Mini

### What you're doing:
Setting up a VPN that lets you access your homelab from anywhere

### How to do it:

**1. Still in SSH to Mac Mini:**

```bash
brew install tailscale
sudo tailscale up
```

**2. You'll see a URL like:**
```
https://login.tailscale.com/a/xxxxxx
```

**3. Copy that URL and paste it into your MacBook browser**

**4. Sign in with Google, Microsoft, or GitHub**

**5. Once it says "Success", go back to the Mac Mini SSH and type:**

```bash
tailscale ip -4
```

**6. Write down this IP - it looks like `100.x.x.x`**

This is your Mac Mini's Tailscale IP. You can use it from ANYWHERE.

---

## Step 2.2: Install Tailscale on MacBook

### What you're doing:
Connecting your MacBook to the same VPN network

### How to do it:

**1. On your MacBook (regular Terminal, NOT in SSH):**

```bash
brew install tailscale
sudo tailscale up
```

**2. Open the URL it gives you in your browser**

**3. Sign in (you'll be auto-added to the same network)**

**4. Test it:**

```bash
tailscale status
```

You should see both Mac Mini and MacBook listed!

**5. Try accessing Portainer with the Tailscale IP:**

In browser:
```
http://100.x.x.x:9000
```
(use the Mac Mini's Tailscale IP you wrote down)

If it works, you can now access your homelab from ANYWHERE!

---

## Step 2.3: Install Tailscale on Raspberry Pi

### What you're doing:
Adding the Pi to your VPN so you can access Pi-hole from anywhere

### How to do it:

**1. SSH into the Pi from MacBook:**

```bash
ssh pi@192.168.1.100
```

**2. Install Tailscale:**

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

**3. Open the URL in your MacBook browser and sign in**

**4. Get the Pi's Tailscale IP:**

```bash
tailscale ip -4
```

Write this down too (another `100.x.x.x` address).

---

## Step 2.4: Install Tailscale on Windows Gaming Laptop

### What you're doing:
Adding your HP Omen to the VPN so you can access it remotely

### How to do it:

**1. On your HP Omen 16:**

- Download Tailscale for Windows: https://tailscale.com/download/windows
- Run the installer
- Click "Connect" or "Sign in"
- Sign in with the same account (Google/Microsoft/GitHub)

**2. Once connected, find your Tailscale IP:**

- Open Command Prompt (Win+R, type `cmd`, Enter)
- Type: `ipconfig`
- Look for "Tailscale" adapter
- Find the IPv4 address (100.x.x.x)
- Write this down

---

## Step 2.5: Enable MagicDNS (Makes Life Easy)

### What you're doing:
Using names instead of IP addresses (way easier to remember)

### How to do it:

**1. On your MacBook browser, go to:**
```
https://login.tailscale.com/admin/dns
```

**2. Turn on "MagicDNS"**

**3. Now you can use names instead of IPs:**

- Mac Mini: `http://mac-mini:9000` (Portainer)
- Raspberry Pi: `http://pihole/admin` (Pi-hole)
- HP Omen: `hp-omen` (for SSH or streaming)

---

# PHASE 3: GAME STREAMING SETUP

## Step 3.1: Install Sunshine on HP Omen 16

### What you're doing:
Setting up the Windows laptop as a game streaming server

### How to do it:

**1. On your HP Omen 16:**

- Download Sunshine: https://github.com/LizardByte/Sunshine/releases
- Get the latest `.exe` installer
- Run it and install
- It might ask for permissions - say Yes

**2. Launch Sunshine:**

- It should appear in your system tray (bottom right)
- Right-click the Sunshine icon → "Open Sunshine"
- Your browser will open to `https://localhost:47990`

**3. Create credentials:**

- Set a username and password (WRITE THESE DOWN)
- Click "Save"

**4. Configure Sunshine:**

Click "Configuration" tab:
- Everything can stay default
- Optional: Set "Sunshine Name" to something like "Gaming-PC"

**5. Allow through firewall (important!):**

Windows will ask if you want to allow Sunshine through firewall - say YES to all.

**6. Test locally first:**

Leave this window open - we'll test it next.

---

## Step 3.2: Install Moonlight on MacBook

### What you're doing:
Installing the client that streams games from Windows to Mac

### How to do it:

**1. On your MacBook:**

- Download Moonlight: https://moonlight-stream.org
- Click "Download" → get the Mac version
- Open the .dmg file
- Drag Moonlight to Applications

**2. Launch Moonlight:**

- Open Applications → Moonlight
- First time it opens, it might ask for permissions (allow them)

---

## Step 3.3: Connect Moonlight to Sunshine

### What you're doing:
Pairing your MacBook to the Windows gaming laptop

### How to do it:

**1. Make sure BOTH devices are on the same network (or Tailscale)**

**2. On your MacBook in Moonlight:**

- It should auto-detect your HP Omen if on same WiFi
- OR click the + button and manually add the Tailscale IP or hostname

**3. Click on your PC when it appears**

**4. You'll see a PIN number**

**5. On the HP Omen, in Sunshine web interface:**

- Click "Pin" tab
- Enter the PIN from Moonlight
- Click "Submit"

**6. On MacBook, Moonlight should now say "Online"**

You're paired!

---

## Step 3.4: Test Game Streaming

### What you're doing:
Making sure you can actually stream games

### How to do it:

**1. On HP Omen:**

- Launch Steam (or any game)
- Make sure a game is installed and can run

**2. On MacBook in Moonlight:**

- You should see "Desktop" and any apps Sunshine detected
- Click "Desktop" to stream the full Windows desktop
- OR click a game to launch it directly

**3. Test the stream:**

- You should see your Windows desktop/game on MacBook
- Try moving the mouse, typing, etc.
- Test a game if you want!

**4. To disconnect:**

- Press Cmd+Option+Shift+Q (or click "Quit Stream" button)

---

## Step 3.5: Optimize Settings

### What you're doing:
Making the stream look and feel better

### How to do it:

**1. In Moonlight on MacBook:**

- Click Settings (gear icon)
- Adjust:
  - Resolution: 1920x1080 or 1280x720 (lower = better performance)
  - FPS: 60 (or 30 if laggy)
  - Bitrate: Start at 20 Mbps, adjust if needed

**2. For remote gaming (via Tailscale):**

- Lower bitrate to 5-10 Mbps
- Lower resolution to 1280x720
- 30 FPS

**3. For local network gaming:**

- Keep higher settings (1080p, 60fps, 20+ Mbps)

---

## Step 3.6: Wake-on-LAN (Optional but Cool)

### What you're doing:
Let your MacBook wake up the sleeping HP Omen remotely

### How to do it:

**1. On HP Omen:**

- Go to Device Manager
- Network Adapters → find your WiFi/Ethernet adapter
- Right click → Properties
- "Power Management" tab
- Check "Allow this device to wake the computer"
- Click OK

**2. In BIOS (this varies by laptop):**

- Restart HP Omen
- Press F10 or DEL during boot
- Find "Wake on LAN" setting
- Enable it
- Save and exit

**3. Get the MAC address:**

```bash
# In Command Prompt on HP Omen:
ipconfig /all
# Look for "Physical Address" - write this down
```

**4. In Moonlight on MacBook:**

- Settings → add the MAC address
- Now when you try to connect, it can wake the PC!

---

# PHASE 4: GITHUB PORTFOLIO

## Step 4.1: Create the GitHub Repository

### What you're doing:
Starting your portfolio repository

### How to do it:

**1. Go to GitHub.com and sign in (or create account)**

**2. Click + in top right → New repository**

**3. Fill in:**
- Name: `homelab`
- Description: "Self-hosted infrastructure with Docker, Pi-hole, Tailscale, and game streaming"
- Public (so people can see it)
- Check "Add a README file"

**4. Click "Create repository"**

---

## Step 4.2: Clone and Set Up Locally

### What you're doing:
Getting the repository onto your MacBook so you can work on it

### How to do it:

**1. On GitHub, click the green "Code" button**

**2. Copy the URL (starts with https://github.com/...)**

**3. On MacBook Terminal:**

```bash
# Go to where you keep projects
cd ~/Documents

# Clone the repo
git clone https://github.com/YOUR-USERNAME/homelab.git

# Enter the directory
cd homelab

# Create folder structure
mkdir -p docs docker/portainer scripts network gaming
```

---

## Step 4.3: Create README.md

### What you're doing:
Making the main page that people see first

### How to do it:

**1. Open the README:**

```bash
nano README.md
```

**2. Paste this template and customize:**

```markdown
# 🏠 Homelab Infrastructure + Game Streaming

Self-hosted infrastructure demonstrating modern DevOps practices, containerization, network security, and distributed game streaming.

## 🎯 What This Is

A complete homelab setup that provides:
- Network-wide ad blocking (Pi-hole)
- Container management (Docker + Portainer)
- Secure remote access (Tailscale VPN)
- Game streaming from anywhere (Sunshine/Moonlight)

## 🏗️ Architecture

### Hardware
- **Mac Mini M4** - Primary server (Docker, Portainer)
- **Raspberry Pi** - Network services (Pi-hole DNS)
- **HP Omen 16** - Game streaming host (Sunshine)
- **MacBook M4 Air** - Client device (management + gaming)

### Software Stack
- Docker & Portainer (container orchestration)
- Pi-hole (DNS ad-blocking)
- Tailscale (zero-trust VPN mesh)
- Sunshine/Moonlight (game streaming)

## ✨ Features

- ✅ Network-wide ad blocking at DNS level
- ✅ Containerized service management
- ✅ Secure remote access from anywhere
- ✅ Stream full Steam library to MacBook
- ✅ No port forwarding required
- ✅ Auto-restart and monitoring

## 📊 What I Learned

### Technical Skills
- Docker containerization and networking
- DNS configuration and ad-blocking
- VPN architecture (WireGuard/Tailscale)
- Game streaming protocols (NVENC, H.264)
- Cross-platform integration (macOS/Linux/Windows)
- Remote system administration

### DevOps Practices
- Infrastructure as Code
- Service orchestration
- Network security best practices
- Documentation and knowledge sharing

## 🚀 Services

| Service | Purpose | Access | Status |
|---------|---------|--------|--------|
| Portainer | Container Management | mac-mini:9000 | ✅ Running |
| Pi-hole | DNS/Ad Blocking | pihole/admin | ✅ Running |
| Tailscale | Mesh VPN | All devices | ✅ Running |
| Sunshine | Game Streaming | hp-omen:47990 | ✅ Running |

## 📚 Documentation

- [Complete Setup Guide](docs/complete-setup-guide.md)
- [Tailscale Configuration](docs/tailscale-setup.md)
- [Game Streaming Guide](docs/game-streaming.md)
- [Lessons Learned](docs/lessons-learned.md)

## 🎮 Game Streaming

Stream games from my full Steam library to MacBook from anywhere:
- Local network: ~5ms latency, 1080p/60fps
- Remote (Tailscale): ~30-50ms latency, 720p/30fps
- Supports keyboard/mouse and controllers

## 🔮 Future Plans

- [ ] Jellyfin media server
- [ ] Prometheus + Grafana monitoring
- [ ] Automated backup solution
- [ ] Nextcloud file storage
- [ ] Home Assistant integration

## 📝 Acknowledgments

Setup and documentation developed with guidance from Claude (Anthropic).

## 📄 License

MIT - Feel free to use as inspiration for your own homelab!
```

**3. Save (Ctrl+X, Y, Enter)**

---

## Step 4.4: Add Your Guides

### What you're doing:
Creating detailed documentation

### How to do it:

**1. Copy the complete setup guide:**

```bash
cp ~/path/to/homelab-setup-guide.md docs/complete-setup-guide.md
```

**2. Create a game streaming guide:**

```bash
nano docs/game-streaming.md
```

Add a focused guide just on Sunshine/Moonlight setup.

**3. Create lessons learned:**

```bash
nano docs/lessons-learned.md
```

Write about challenges you faced and how you solved them. For example:

```markdown
# Lessons Learned

## Challenge: Understanding Tailscale vs Traditional VPN
**Problem:** Initially confused about how Tailscale differs from OpenVPN/WireGuard.
**Solution:** Learned Tailscale uses WireGuard but adds orchestration and NAT traversal.
**Takeaway:** Modern mesh VPNs are superior for homelab - no port forwarding needed.

## Challenge: Game Streaming Latency
**Problem:** Initial streaming tests had noticeable input lag.
**Solution:** Adjusted Moonlight bitrate and enabled hardware encoding in Sunshine.
**Takeaway:** Bitrate isn't always better - matching to network capacity is key.
```

---

## Step 4.5: Add Docker Compose Files

### What you're doing:
Showing Infrastructure as Code skills

### How to do it:

```bash
nano docker/portainer/docker-compose.yml
```

```yaml
version: "3"

services:
  portainer:
    container_name: portainer
    image: portainer/portainer-ce:latest
    ports:
      - "9000:9000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    restart: unless-stopped

volumes:
  portainer_data:
```

---

## Step 4.6: Commit and Push

### What you're doing:
Uploading everything to GitHub

### How to do it:

```bash
# Add all files
git add .

# Commit with message
git commit -m "Complete homelab setup with game streaming"

# Push to GitHub
git push origin main
```

**If it asks for credentials:**
- Use your GitHub username
- For password, create a Personal Access Token at github.com/settings/tokens

---

## Step 4.7: Polish It Up

### What you're doing:
Making it look professional

### How to do it:

**1. Add badges to README:**

At the top of README.md:
```markdown
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Raspberry Pi](https://img.shields.io/badge/Raspberry%20Pi-A22846?style=for-the-badge&logo=Raspberry%20Pi&logoColor=white)
![Tailscale](https://img.shields.io/badge/Tailscale-000000?style=for-the-badge&logo=tailscale&logoColor=white)
```

**2. Add GitHub topics:**

On GitHub repo page:
- Click the gear icon next to "About"
- Add topics: `homelab`, `docker`, `self-hosted`, `devops`, `game-streaming`, `pihole`, `tailscale`

**3. Pin to your profile:**

- Go to your GitHub profile
- Click "Customize your pins"
- Select your homelab repo

---

# 📋 QUICK REFERENCE CHEAT SHEET

## IP Addresses (Fill these in as you go)

| Device | Local IP | Tailscale IP | Hostname |
|--------|----------|--------------|----------|
| Mac Mini | 192.168.1.___ | 100.___.___.__ | mac-mini |
| Raspberry Pi | 192.168.1.___ | 100.___.___.__ | pihole |
| HP Omen 16 | 192.168.1.___ | 100.___.___.__ | hp-omen |
| MacBook | 192.168.1.___ | 100.___.___.__ | macbook |

## Passwords & Credentials

| Service | Username | Password | URL |
|---------|----------|----------|-----|
| Pi SSH | | | |
| Mac Mini SSH | | | |
| HP Omen SSH | | | |
| Pi-hole | admin | | http://pihole/admin |
| Portainer | | | http://mac-mini:9000 |
| Sunshine | | | https://hp-omen:47990 |

## Useful Commands

**Check Tailscale:**
```bash
tailscale status
```

**SSH to devices:**
```bash
ssh pi@pihole
ssh username@mac-mini
ssh username@hp-omen
```

**Restart services:**
```bash
# Restart Tailscale
sudo tailscale down && sudo tailscale up

# Restart Portainer
docker restart portainer

# Restart Pi-hole
pihole restartdns
```

## Access URLs

**Local network:**
- Portainer: http://192.168.1.50:9000
- Pi-hole: http://192.168.1.100/admin
- Sunshine: https://192.168.1.XXX:47990

**Via Tailscale (from anywhere):**
- Portainer: http://mac-mini:9000
- Pi-hole: http://pihole/admin
- Sunshine: https://hp-omen:47990

---

# ✅ FINAL CHECKLIST

## Phase 1: Core Infrastructure
- [ ] Raspberry Pi flashed and SSH accessible
- [ ] Pi-hole installed and working
- [ ] Mac Mini SSH enabled
- [ ] Docker installed on Mac Mini
- [ ] Portainer accessible
- [ ] Network using Pi-hole for DNS

## Phase 2: Remote Access
- [ ] Tailscale on Mac Mini
- [ ] Tailscale on MacBook
- [ ] Tailscale on Raspberry Pi
- [ ] Tailscale on HP Omen
- [ ] MagicDNS enabled
- [ ] Can access services via Tailscale

## Phase 3: Game Streaming
- [ ] Sunshine installed on HP Omen
- [ ] Moonlight installed on MacBook
- [ ] Devices paired
- [ ] Stream working locally
- [ ] Stream working remotely (via Tailscale)
- [ ] Settings optimized

## Phase 4: GitHub Portfolio
- [ ] Repository created
- [ ] README.md written
- [ ] Setup guides added
- [ ] Lessons learned documented
- [ ] Docker compose files added
- [ ] Everything pushed to GitHub
- [ ] Repo pinned to profile
- [ ] Topics/tags added

---

# 🎉 YOU'RE DONE!

You now have:
- A functional homelab with remote access
- Game streaming from anywhere
- An impressive GitHub portfolio
- Real DevOps skills you can talk about in interviews

## Next Steps

1. **Use it** - Actually use your setup daily
2. **Document issues** - When things break, write about fixes
3. **Expand** - Add new services over time
4. **Share** - Tweet about it, write blog posts
5. **Interview** - Talk about this in job applications!

## Resources

- **Homelab Reddit**: r/homelab
- **Self-Hosted Reddit**: r/selfhosted
- **Pi-hole Discourse**: discourse.pi-hole.net
- **Tailscale Docs**: tailscale.com/kb
- **Moonlight Discord**: discord.gg/moonlight

---

**Pro Tip:** Screenshot your dashboards and add them to your GitHub repo in an `images/` folder!
