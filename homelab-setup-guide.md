# Complete Homelab Network Infrastructure Setup Guide with Remote Access

This guide will walk you through setting up a complete network infrastructure using:
- **MacBook** (your control device, in bed or on the go)
- **Mac Mini M4** (running Portainer and potentially other services)
- **Raspberry Pi** (running Pi-hole or AdGuard Home for network-wide ad blocking)
- **Router/Modem** (your network gateway)
- **Tailscale** (for secure remote access from anywhere)

## Prerequisites

- All devices connected to the same network with internet access
- You know your Mac Mini and MacBook usernames and passwords
- A microSD card for the Raspberry Pi (at least 8GB)
- An SD card reader for your MacBook (if needed)

---

## Part 1: Prepare the Raspberry Pi

### Step 1.1: Flash Raspberry Pi OS (from your MacBook)

**On your MacBook:**

1. Download and install **Raspberry Pi Imager**: https://www.raspberrypi.com/software/
2. Insert your microSD card into your MacBook
3. Open Raspberry Pi Imager
4. Click "Choose Device" → select your Raspberry Pi model
5. Click "Choose OS" → select "Raspberry Pi OS Lite (64-bit)" (no desktop needed)
6. Click "Choose Storage" → select your microSD card
7. Click "Next"
8. Click "Edit Settings" when prompted

**In the settings:**
- **General tab:**
  - Set hostname: `pihole` (or whatever you want)
  - Enable "Set username and password" → create a username and password (write these down!)
  - Configure wireless LAN if using Wi-Fi (enter your WiFi name and password)
  - Set locale settings (timezone and keyboard layout)
  
- **Services tab:**
  - Enable SSH
  - Select "Use password authentication"

9. Click "Save"
10. Click "Yes" to apply settings
11. Click "Yes" to confirm you want to erase the card
12. Wait for the imaging process to complete (5-10 minutes)
13. When done, eject the microSD card and insert it into your Raspberry Pi
14. Power on the Raspberry Pi

Wait about 2-3 minutes for the Pi to boot up completely.

### Step 1.2: Find your Raspberry Pi's IP address

**On your MacBook, open Terminal:**

```bash
# Scan your network to find the Pi
arp -a | grep -i "b8:27\|dc:a6\|e4:5f"
```

Alternatively, check your router's admin page to see connected devices and find the Raspberry Pi.

Write down this IP address (something like `192.168.1.100`).

### Step 1.3: SSH into the Raspberry Pi

**From your MacBook Terminal:**

```bash
ssh username@[Raspberry-Pi-IP]
```

Replace `username` with what you set during imaging and `[Raspberry-Pi-IP]` with the IP you found.

Type "yes" if asked about authenticity, then enter your Raspberry Pi password.

You're now connected to your Pi remotely!

### Step 1.4: Update the Raspberry Pi

**In your SSH session to the Pi:**

```bash
# Update package list and upgrade all packages
sudo apt update && sudo apt upgrade -y

# Reboot to ensure everything is fresh
sudo reboot
```

Wait 1 minute, then SSH back in:

```bash
ssh username@[Raspberry-Pi-IP]
```

---

## Part 2: Choose and Install Pi-hole OR AdGuard Home

You only need ONE of these. Both do the same thing (DNS-based ad blocking). Choose based on preference:

- **Pi-hole**: More established, larger community, simpler interface
- **AdGuard Home**: More modern UI, built-in HTTPS/DoH support, slightly more features

### Option A: Install Pi-hole

**In your SSH session to the Raspberry Pi:**

```bash
# Download and run Pi-hole installer
curl -sSL https://install.pi-hole.net | bash
```

During installation:
1. Press Enter through the initial screens
2. Select your network interface (usually `eth0` for ethernet or `wlan0` for WiFi)
3. Select an upstream DNS provider (Google, Cloudflare, OpenDNS—doesn't matter much, choose Cloudflare)
4. Keep default blocklists: Yes
5. Install web admin interface: Yes
6. Install web server (lighttpd): Yes
7. Enable logging: Yes (your choice, but Yes is useful for learning)
8. Privacy mode: Show everything (or your preference)

**At the end, it will show you a password—WRITE THIS DOWN!**

You can change it later with:
```bash
pihole -a -p
```

**Access Pi-hole web interface:**

From your MacBook browser: `http://[Raspberry-Pi-IP]/admin`

Login with the password shown during installation.

---

### Option B: Install AdGuard Home

**In your SSH session to the Raspberry Pi:**

```bash
# Download and run AdGuard Home installer
curl -s -S -L https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh | sh -s -- -v
```

The installer will show you:
- Admin panel URL: `http://[Raspberry-Pi-IP]:3000`

**Complete setup:**

1. From your MacBook browser, go to `http://[Raspberry-Pi-IP]:3000`
2. Click "Get Started"
3. Keep default admin interface settings (port 3000, all interfaces)
4. Keep default DNS settings (port 53)
5. Create an admin username and password (WRITE THESE DOWN!)
6. Click through the remaining setup screens
7. Click "Open Dashboard"

Login with your credentials.

---

## Part 3: Set up the Mac Mini with Docker and Portainer

### Step 3.1: Enable Remote Login on Mac Mini

**You need to do this once on the Mac Mini itself:**

1. Go to System Settings → General → Sharing
2. Turn on "Remote Login"
3. Make sure your user account is allowed to connect
4. Note the IP address shown (or find it in System Settings → Network)

Write down the Mac Mini's IP address.

### Step 3.2: SSH into Mac Mini from MacBook

**On your MacBook, open a new Terminal window (or tab):**

```bash
ssh yourusername@[Mac-Mini-IP]
```

Replace `yourusername` with your Mac Mini username and `[Mac-Mini-IP]` with its IP.

Enter your Mac Mini password when prompted.

### Step 3.3: Install Homebrew and Docker

**In your SSH session to the Mac Mini:**

```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Follow any instructions Homebrew gives about adding to PATH (usually something like):

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

Now install Docker:

```bash
# Install Docker Desktop
brew install --cask docker

# Start Docker Desktop
open -a Docker

# Wait for Docker to start (30 seconds)
sleep 30
```

### Step 3.4: Install Portainer

```bash
# Create volume for Portainer data
docker volume create portainer_data

# Run Portainer
docker run -d -p 9000:9000 --name portainer --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

### Step 3.5: Access Portainer

**From your MacBook browser:**

Go to: `http://[Mac-Mini-IP]:9000`

1. Create an admin account (username and password—SAVE THESE!)
2. Click "Get Started"
3. Select the local Docker environment

You now have Portainer running! You can deploy other services here later (media servers, file storage, monitoring, etc.).

---

## Part 4: Install Tailscale for Remote Access

This is where the magic happens—Tailscale creates a secure mesh VPN between all your devices so you can access your homelab from anywhere in the world.

### Step 4.1: Install Tailscale on Mac Mini

**In your SSH session to the Mac Mini:**

```bash
# Install Tailscale
brew install tailscale

# Start Tailscale
sudo tailscale up
```

You'll see a URL like: `https://login.tailscale.com/a/xxxxxx`

**Copy this URL and paste it into your MacBook browser.** Sign in with your preferred account (Google, Microsoft, GitHub, etc.) to authenticate the Mac Mini.

Once authenticated, you'll see "Success" in the browser.

**Back in the Mac Mini SSH session, get the Tailscale IP:**

```bash
tailscale ip -4
```

You'll see an IP like `100.x.x.x`—this is your Mac Mini's Tailscale IP. **Write this down!**

### Step 4.2: Install Tailscale on MacBook

**On your MacBook (in a regular Terminal, not SSH):**

```bash
# Install Tailscale
brew install tailscale

# Start Tailscale
sudo tailscale up
```

Again, you'll get a URL. Open it in your browser and authenticate (you'll be automatically signed into the same Tailscale network).

**Verify it's working:**

```bash
tailscale status
```

You should see both your MacBook and Mac Mini listed!

### Step 4.3: Install Tailscale on Raspberry Pi (Optional but Recommended)

**In your SSH session to the Raspberry Pi:**

```bash
# Install Tailscale
curl -fsSL https://tailscale.com/install.sh | sh

# Start Tailscale
sudo tailscale up
```

Open the authentication URL in your MacBook browser and sign in.

**Get the Pi's Tailscale IP:**

```bash
tailscale ip -4
```

Write this down too!

### Step 4.4: Test Remote Access

**On your MacBook:**

Now instead of using local IPs (`192.168.x.x`), you can use Tailscale IPs (`100.x.x.x`) to access your devices from anywhere!

Try accessing Portainer using the Tailscale IP:

```bash
# In your MacBook browser
http://100.x.x.x:9000
```

Replace `100.x.x.x` with your Mac Mini's Tailscale IP.

**It works!** Now even when you're on cellular data, at a coffee shop, or traveling, you can access your Mac Mini's Portainer dashboard.

Same for Pi-hole/AdGuard:

```bash
# In your MacBook browser
http://100.y.y.y/admin  # Pi-hole
# or
http://100.y.y.y:3000   # AdGuard Home
```

Replace `100.y.y.y` with your Raspberry Pi's Tailscale IP.

### Step 4.5: Enable MagicDNS (Optional but Convenient)

Instead of remembering IP addresses, you can use hostnames!

**On your MacBook browser:**

1. Go to https://login.tailscale.com/admin/dns
2. Enable "MagicDNS"

Now you can access devices by name:
- `http://mac-mini:9000` for Portainer
- `http://pihole/admin` for Pi-hole
- SSH with `ssh username@mac-mini`

Much easier to remember!

### Step 4.6: Configure Tailscale to Start on Boot

**On Mac Mini (SSH session):**

```bash
# Enable Tailscale to start automatically
sudo tailscale set --operator=$USER
```

**On Raspberry Pi (SSH session):**

```bash
# Enable Tailscale service
sudo systemctl enable tailscaled
sudo systemctl start tailscaled
```

Now Tailscale will automatically connect when these devices boot up.

---

## Part 5: Configure Your Network to Use Pi-hole/AdGuard

You have two options: test on just your MacBook first, or enable for the whole network.

### Option A: Test on MacBook Only (Recommended First)

**On your MacBook:**

1. Open System Settings → Network
2. Click your connection (Wi-Fi or Ethernet)
3. Click "Details"
4. Go to the "DNS" tab
5. Click the + button and add your Raspberry Pi's **local** IP address (the `192.168.x.x` one, not Tailscale)
6. Move it to the top of the list if there are other DNS servers
7. Click OK

**Test it:**
- Visit some ad-heavy websites
- Check your Pi-hole/AdGuard dashboard—you should see queries appearing
- You should see fewer ads!

### Option B: Enable for Entire Network

**Once you've confirmed it works, configure your router:**

The exact steps depend on your router, but generally:

1. Log into your router's admin page (usually `192.168.1.1` or `192.168.0.1`)
2. Look for DHCP settings, DNS settings, or LAN settings
3. Set Primary DNS Server to your Raspberry Pi's **local** IP address (`192.168.x.x`)
4. Set Secondary DNS Server to something like `8.8.8.8` (Google) as a backup
5. Save settings
6. Restart your router (or wait for DHCP leases to renew)

All devices on your network will now use Pi-hole/AdGuard for DNS!

### Option C: Use Pi-hole/AdGuard When Away from Home (via Tailscale)

When you're away from home and want ad blocking:

**On your MacBook:**

1. Make sure Tailscale is connected: `tailscale status`
2. System Settings → Network → Details → DNS
3. Add your Raspberry Pi's **Tailscale** IP (`100.y.y.y`)
4. Move it to the top

Now you have ad blocking even on coffee shop WiFi or cellular!

**Pro tip:** You can use Tailscale's subnet routing to make this even more seamless, but that's advanced—this works great for now.

---

## Part 6: Make IP Addresses Static (Local Network Only)

For everything to work reliably on your local network, your Raspberry Pi and Mac Mini should have static local IP addresses.

### Option A: Set Static IPs on the Devices

**For Raspberry Pi:**

```bash
# Still in SSH session to the Pi
sudo nano /etc/dhcpcd.conf
```

Add to the bottom (adjust for your network):

```
interface eth0  # or wlan0 for WiFi
static ip_address=192.168.1.100/24
static routers=192.168.1.1
static domain_name_servers=1.1.1.1 8.8.8.8
```

Save (Ctrl+X, Y, Enter) and reboot:

```bash
sudo reboot
```

**For Mac Mini:**

1. System Settings → Network
2. Click your connection → Details
3. Go to TCP/IP tab
4. Change "Configure IPv4" from "Using DHCP" to "Manually"
5. Enter your current IP address, subnet mask (usually 255.255.255.0), and router address
6. Click OK

### Option B: Set DHCP Reservations in Router (Easier)

Log into your router and create DHCP reservations for both the Raspberry Pi and Mac Mini using their MAC addresses. This ensures they always get the same IP addresses.

---

## Part 7: Using Your Homelab On the Go

### When You're Away from Home

**Step 1: Make sure Tailscale is running on your MacBook**

```bash
# Check Tailscale status
tailscale status
```

You should see your Mac Mini and Raspberry Pi listed as online.

**Step 2: Access your services using Tailscale IPs or hostnames**

In your browser:
- Portainer: `http://mac-mini:9000` or `http://100.x.x.x:9000`
- Pi-hole: `http://pihole/admin` or `http://100.y.y.y/admin`
- AdGuard: `http://pihole:3000` or `http://100.y.y.y:3000`

**Step 3: SSH into your devices**

```bash
# SSH to Mac Mini from anywhere
ssh yourusername@mac-mini
# or
ssh yourusername@100.x.x.x

# SSH to Raspberry Pi from anywhere
ssh username@pihole
# or
ssh username@100.y.y.y
```

**That's it!** Your entire homelab is accessible as if you were at home.

### Quick Reference Card

Save these for when you're traveling:

**Tailscale Commands:**
```bash
# Check connection status
tailscale status

# Get your device IPs
tailscale ip -4

# Restart Tailscale if needed
sudo tailscale down
sudo tailscale up
```

**Access URLs (replace with your actual hostnames/IPs):**
- Portainer: `http://mac-mini:9000`
- Pi-hole: `http://pihole/admin`
- AdGuard: `http://pihole:3000`

**SSH Access:**
- Mac Mini: `ssh username@mac-mini`
- Raspberry Pi: `ssh username@pihole`

---

## Summary of What You've Built

- **Raspberry Pi**: Running Pi-hole or AdGuard Home for network-wide ad blocking and DNS management, accessible remotely via Tailscale
- **Mac Mini**: Running Docker and Portainer, ready for additional services, accessible remotely via Tailscale
- **MacBook**: Your control center for managing everything from bed or anywhere in the world
- **Tailscale**: Secure mesh VPN connecting all your devices, allowing remote access without port forwarding or exposing services to the internet
- **Network**: Protected from ads and tracking at the DNS level

## What You Can Add Later

Using Portainer on your Mac Mini, you can easily add:
- **Jellyfin/Plex** for media streaming (accessible via Tailscale on the go!)
- **Nextcloud** for file storage and sync
- **Home Assistant** for smart home automation
- **Grafana + Prometheus** for monitoring
- **Uptime Kuma** for service monitoring
- **Gitea** for Git hosting
- **Wireguard** (though Tailscale already handles VPN)

Just search for docker-compose files for any service and deploy them through Portainer!

---

## Troubleshooting

**Can't SSH into Raspberry Pi:**
- Check it's powered on and connected to network
- Verify IP address in router
- Try `ssh -v username@ip` for verbose debugging

**Pi-hole/AdGuard not blocking ads:**
- Verify DNS is set correctly
- Check the Pi-hole/AdGuard dashboard shows queries
- Try flushing DNS cache on your device: `sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder`

**Docker won't start on Mac Mini:**
- Check Docker Desktop is running: `docker ps`
- Try restarting Docker Desktop: `killall Docker && open -a Docker`

**Containers won't start:**
- Check logs in Portainer
- Verify ports aren't already in use
- Ensure Docker has enough resources (Docker Desktop → Settings → Resources)

**Can't access services via Tailscale:**
- Check Tailscale is running: `tailscale status`
- Verify devices show as online
- Try using IP addresses instead of hostnames
- Check firewall settings aren't blocking Tailscale

**Tailscale not connecting:**
- Try: `sudo tailscale down && sudo tailscale up`
- Check you're authenticated: go to https://login.tailscale.com/admin/machines
- Verify the device shows as "Connected"

---

## Security Notes

**Why Tailscale is secure:**
- Creates a private mesh VPN between only your devices
- Uses WireGuard protocol (modern, fast, secure)
- No ports exposed to the public internet
- End-to-end encrypted
- Zero-trust network access

**Important:** With Tailscale, you don't need to open ports on your router or expose services to the internet. Everything stays private within your Tailscale network.

---

## IP Address Tracking Sheet

Fill this in as you go through the setup:

| Device | Local IP | Tailscale IP | Hostname | Notes |
|--------|----------|--------------|----------|-------|
| Mac Mini | 192.168.1.___ | 100.___.___.___ | mac-mini | Portainer on :9000 |
| Raspberry Pi | 192.168.1.___ | 100.___.___.___ | pihole | Pi-hole on /admin or AdGuard on :3000 |
| MacBook | 192.168.1.___ | 100.___.___.___ | macbook | Your control device |
| Router | 192.168.1.___ | N/A | N/A | Gateway |

---

## Credentials Tracking Sheet

**KEEP THIS SECURE!**

| Service | Username | Password | URL | Notes |
|---------|----------|----------|-----|-------|
| Raspberry Pi SSH | | | | |
| Mac Mini SSH | | | | |
| Pi-hole Admin | admin | | http://pihole/admin | |
| AdGuard Admin | | | http://pihole:3000 | |
| Portainer | | | http://mac-mini:9000 | |
| Tailscale | | | https://login.tailscale.com | Uses OAuth (Google/MS/GitHub) |

---

You're all set! You now have a homelab you can manage from bed, from a coffee shop, from across the country—anywhere with an internet connection. Your Mac Mini and Raspberry Pi are doing the heavy lifting at home, and Tailscale makes it feel like you're always on your home network.

Enjoy your new superpower! 🚀
