# Docker Setup Guide

Complete guide to Docker and Portainer configuration for the homelab.

---

## Docker Installation

### Installing Docker Desktop on macOS

```bash
# Install via Homebrew
brew install --cask docker

# Launch Docker Desktop
open -a Docker

# Verify installation
docker --version
docker-compose --version
```

### Post-Installation Configuration

**Allocate Resources (Docker Desktop Settings):**

1. Open Docker Desktop
2. Settings → Resources
3. Recommended for Mac Mini M4:
   - CPUs: 4
   - Memory: 8GB
   - Swap: 2GB
   - Disk: 100GB

**Enable Docker CLI access:**

```bash
# Verify Docker CLI works
docker ps

# If command not found, restart terminal
source ~/.zprofile
```

---

## Portainer Deployment

### Method 1: Docker CLI (Recommended for Initial Setup)

```bash
# Create persistent volume
docker volume create portainer_data

# Deploy Portainer
docker run -d \
  --name portainer \
  --restart always \
  -p 9000:9000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest

# Verify deployment
docker ps | grep portainer
```

### Method 2: Docker Compose

Create `portainer-compose.yaml`:

```yaml
version: '3.8'

services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    ports:
      - "9000:9000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    restart: unless-stopped

volumes:
  portainer_data:
```

Deploy:

```bash
docker-compose -f portainer-compose.yaml up -d
```

### Initial Portainer Configuration

1. Navigate to `http://mac-mini-ip:9000`
2. Create admin user (username + strong password)
3. Click "Get Started"
4. Select "Local" environment
5. Dashboard loads

---

## Container Management via Portainer

### Creating Stacks (Recommended Method)

**Stacks = docker-compose files managed via web UI**

1. Portainer → **Stacks** → **Add stack**
2. Name your stack (e.g., `monitoring`)
3. Choose:
   - **Web editor** - Paste YAML directly
   - **Upload** - Upload docker-compose.yaml file
   - **Repository** - Pull from Git repo
4. Paste your docker-compose configuration
5. Click **Deploy the stack**

### Managing Individual Containers

**Via Portainer UI:**

1. **Containers** → Select container → Options:
   - **Logs** - View container output
   - **Inspect** - See detailed config
   - **Stats** - CPU/Memory usage
   - **Console** - Attach to container shell
   - **Restart** - Restart container
   - **Stop/Start** - Control container state
   - **Remove** - Delete container

**Via CLI:**

```bash
# View running containers
docker ps

# View all containers
docker ps -a

# View container logs
docker logs container-name

# Follow logs in real-time
docker logs -f container-name

# Restart container
docker restart container-name

# Stop container
docker stop container-name

# Start stopped container
docker start container-name

# Remove container (must be stopped first)
docker stop container-name
docker rm container-name
```

---

## Docker Compose Files

### Basic Structure

```yaml
version: '3.8'

services:
  service-name:
    image: image-name:tag
    container_name: friendly-name
    ports:
      - "host-port:container-port"
    volumes:
      - volume-name:/path/in/container
      - /host/path:/container/path
    environment:
      - ENV_VAR=value
    restart: unless-stopped

volumes:
  volume-name:
```

### Example: Multi-Service Stack

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

  dozzle:
    image: amir20/dozzle:latest
    container_name: dozzle
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    ports:
      - 8080:8080
    restart: unless-stopped

  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - WATCHTOWER_CLEANUP=true
      - WATCHTOWER_SCHEDULE=0 0 4 * * *
    restart: unless-stopped

volumes:
  uptime-kuma:
```

---

## Volume Management

### Understanding Docker Volumes

**Named Volumes** (Recommended):
- Managed by Docker
- Persist across container removals
- Easy to backup

```yaml
volumes:
  - volume-name:/container/path
```

**Bind Mounts**:
- Direct host path mapping
- Useful for config files
- Changes reflected immediately

```yaml
volumes:
  - /host/path:/container/path
```

### Volume Commands

```bash
# List volumes
docker volume ls

# Inspect volume
docker volume inspect volume-name

# Create volume
docker volume create volume-name

# Remove volume (container must be stopped)
docker volume rm volume-name

# Remove unused volumes
docker volume prune
```

### Backing Up Volumes

```bash
# Backup volume to tar file
docker run --rm \
  -v volume-name:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/volume-backup.tar.gz -C /data .

# Restore volume from backup
docker run --rm \
  -v volume-name:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/volume-backup.tar.gz -C /data
```

---

## Networking

### Default Bridge Network

All containers on same host can communicate via container names.

Example:
- Container A: `uptime-kuma`
- Container B: `portainer`
- Container A can reach B at: `http://portainer:9000`

### Creating Custom Networks

```bash
# Create network
docker network create homelab-network

# Run container on custom network
docker run -d \
  --network homelab-network \
  --name container-name \
  image-name
```

In docker-compose:

```yaml
version: '3.8'

services:
  service1:
    image: image1
    networks:
      - homelab

  service2:
    image: image2
    networks:
      - homelab

networks:
  homelab:
    driver: bridge
```

---

## Best Practices

### Resource Limits

Prevent containers from consuming all resources:

```yaml
services:
  service-name:
    image: image-name
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
```

### Health Checks

Monitor container health:

```yaml
services:
  service-name:
    image: image-name
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### Restart Policies

```yaml
restart: no           # Never restart
restart: always       # Always restart
restart: on-failure   # Restart on non-zero exit
restart: unless-stopped  # Restart unless manually stopped (RECOMMENDED)
```

### Logging Configuration

Prevent logs from filling disk:

```yaml
services:
  service-name:
    image: image-name
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

---

## Useful Services to Deploy

### Monitoring Stack

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

  dozzle:
    image: amir20/dozzle:latest
    container_name: dozzle
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    ports:
      - 8080:8080
    restart: unless-stopped

  glances:
    image: nicolargo/glances:latest
    container_name: glances
    pid: host
    ports:
      - 61208:61208
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      - GLANCES_OPT=-w
    restart: unless-stopped

volumes:
  uptime-kuma:
```

### Dashboard Stack

```yaml
version: '3.8'

services:
  homepage:
    image: ghcr.io/gethomepage/homepage:latest
    container_name: homepage
    ports:
      - 3000:3000
    volumes:
      - ./homepage:/app/config
      - /var/run/docker.sock:/var/run/docker.sock:ro
    restart: unless-stopped
```

---

## Troubleshooting

### Container Won't Start

```bash
# Check logs
docker logs container-name

# Inspect container config
docker inspect container-name

# Check if port is already in use
lsof -i :PORT_NUMBER
```

### Permission Issues

```bash
# Fix volume permissions
docker run --rm -v volume-name:/data alpine chown -R 1000:1000 /data
```

### Out of Disk Space

```bash
# Clean up unused resources
docker system prune -a

# Remove specific items
docker container prune  # Remove stopped containers
docker image prune -a   # Remove unused images
docker volume prune     # Remove unused volumes
docker network prune    # Remove unused networks
```

### Container Can't Reach Other Containers

```bash
# Verify network
docker network inspect bridge

# Check if containers are on same network
docker inspect container-name | grep NetworkMode
```

---

## Docker Maintenance

### Weekly Tasks

```bash
# Update all containers (if using Watchtower, this is automatic)
docker-compose pull
docker-compose up -d

# Clean up unused resources
docker system prune -f
```

### Monthly Tasks

```bash
# Backup volumes
for vol in $(docker volume ls -q); do
  docker run --rm -v $vol:/data -v $(pwd):/backup alpine tar czf /backup/$vol.tar.gz -C /data .
done

# Review container logs for errors
docker ps -q | xargs -I {} docker logs {} --since 720h 2>&1 | grep -i error
```

---

## Security Considerations

### Don't Expose Docker Socket Unnecessarily

The Docker socket (`/var/run/docker.sock`) gives **full control** of Docker.

**Only mount it for:**
- Portainer (container management)
- Uptime Kuma (container monitoring)
- Watchtower (auto-updates)
- Dozzle (log viewing)

**Never mount it for:**
- Untrusted containers
- Internet-facing services
- Third-party images you don't trust

### Use Official Images

Always prefer official images from:
- Docker Hub official images
- GitHub Container Registry (ghcr.io)
- Project's official registry

### Keep Images Updated

Use Watchtower or manually update:

```bash
docker-compose pull
docker-compose up -d
```

### Scan Images for Vulnerabilities

```bash
# Using Docker Scout (built-in)
docker scout cves image-name
```

---

## Next Steps

- Deploy additional services (Homepage, Dozzle, etc.)
- Set up automated backups
- Configure Tailscale for remote access
- Implement monitoring and alerting
- Document your custom configurations

See [Monitoring Setup](monitoring-setup.md) for configuring Uptime Kuma and external monitoring.
