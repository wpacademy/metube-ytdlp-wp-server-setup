# Metube VPS Setup Guide with Docker, UFW, and Nginx

This guide provides step-by-step instructions for deploying [Metube](https://github.com/alexta69/metube) (a self-hosted YouTube downloader) on a VPS using Docker, securing it with UFW firewall, and exposing it through Nginx with a custom domain name.

## Prerequisites

- A VPS running Ubuntu 20.04 or newer (or similar Debian-based distribution)
- Root or sudo access to the server
- A domain name pointed to your VPS IP address
- Basic command-line knowledge

## Table of Contents

1. [Initial Server Setup](#initial-server-setup)
2. [Install Docker and Docker Compose](#install-docker-and-docker-compose)
3. [Configure UFW Firewall](#configure-ufw-firewall)
4. [Install and Configure Nginx](#install-and-configure-nginx)
5. [Deploy Metube with Docker](#deploy-metube-with-docker)
6. [Configure Nginx Reverse Proxy](#configure-nginx-reverse-proxy)
7. [Set Up SSL with Let's Encrypt](#set-up-ssl-with-lets-encrypt)
8. [Maintenance and Updates](#maintenance-and-updates)
9. [Troubleshooting](#troubleshooting)

---

## Initial Server Setup

### 1. Update System Packages

```bash
sudo apt update
sudo apt upgrade -y
```

### 2. Create a Non-Root User (Optional but Recommended)

```bash
# Create new user
sudo adduser metube

# Add user to sudo group
sudo usermod -aG sudo metube

# Switch to new user
su - metube
```

---

## Install Docker and Docker Compose

### 1. Install Docker

```bash
# Install prerequisites
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Add Docker repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

### 2. Add Your User to Docker Group

```bash
sudo usermod -aG docker $USER

# Log out and log back in, or run:
newgrp docker

# Verify Docker installation
docker --version
docker run hello-world
```

### 3. Install Docker Compose

```bash
# Download Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Make it executable
sudo chmod +x /usr/local/bin/docker-compose

# Verify installation
docker-compose --version
```

### 4. Enable Docker to Start on Boot

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

---

## Configure UFW Firewall

### 1. Install UFW (if not already installed)

```bash
sudo apt install -y ufw
```

### 2. Configure Default Policies

```bash
# Deny all incoming traffic by default
sudo ufw default deny incoming

# Allow all outgoing traffic
sudo ufw default allow outgoing
```

### 3. Allow Required Ports

```bash
# Allow SSH (IMPORTANT: Do this before enabling UFW!)
sudo ufw allow ssh
# Or specify SSH port if you changed it (e.g., port 2222):
# sudo ufw allow 2222/tcp

# Allow HTTP
sudo ufw allow 80/tcp

# Allow HTTPS
sudo ufw allow 443/tcp
```

### 4. Enable UFW

```bash
# Enable the firewall
sudo ufw enable

# Check status
sudo ufw status verbose
```

Expected output:
```
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
```

**Note:** Port 8081 (Metube's default) is NOT exposed to the internet. Nginx will handle external requests on ports 80/443 and proxy them to Metube.

---

## Install and Configure Nginx

### 1. Install Nginx

```bash
sudo apt install -y nginx
```

### 2. Enable Nginx to Start on Boot

```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

### 3. Verify Nginx is Running

```bash
sudo systemctl status nginx
```

Visit your server's IP address in a browser. You should see the default Nginx welcome page.

---

## Deploy Metube with Docker

### 1. Create Directory Structure

```bash
# Create directory for Metube
mkdir -p ~/metube
cd ~/metube

# Create downloads directory
mkdir -p downloads
```

### 2. Create Docker Compose File

```bash
nano docker-compose.yml
```

Add the following configuration:

```yaml
version: '3'

services:
  metube:
    image: ghcr.io/alexta69/metube
    container_name: metube
    restart: unless-stopped
    ports:
      - "127.0.0.1:8081:8081"  # Only bind to localhost
    volumes:
      - ./downloads:/downloads
    environment:
      # Download settings
      - DOWNLOAD_DIR=/downloads
      - TEMP_DIR=/downloads
      - STATE_DIR=/downloads/.metube
      
      # Download mode: sequential, concurrent, or limited
      - DOWNLOAD_MODE=limited
      - MAX_CONCURRENT_DOWNLOADS=3
      
      # File naming
      - OUTPUT_TEMPLATE=%(title)s.%(ext)s
      - OUTPUT_TEMPLATE_PLAYLIST=%(playlist_title)s/%(title)s.%(ext)s
      
      # UI Settings
      - DEFAULT_THEME=auto
      - CUSTOM_DIRS=true
      - CREATE_CUSTOM_DIRS=true
      
      # User/Group settings (match your user ID)
      - UID=1000
      - GID=1000
      - UMASK=022
      
      # Logging
      - LOGLEVEL=INFO
```

**Important:** `127.0.0.1:8081:8081` ensures Metube only listens on localhost and isn't accessible directly from the internet.

### 3. Adjust UID and GID

Find your user ID and group ID:

```bash
id
```

Update the `UID` and `GID` values in the docker-compose.yml file to match your user.

### 4. Start Metube

```bash
# Start in detached mode
docker-compose up -d

# Check logs
docker-compose logs -f

# Verify it's running
docker-compose ps
```

### 5. Test Local Access

```bash
curl http://localhost:8081
```

You should see HTML output from Metube.

---

## Configure Nginx Reverse Proxy

### 1. Create Nginx Configuration File

Replace `yourdomain.com` with your actual domain name:

```bash
sudo nano /etc/nginx/sites-available/metube
```

Add the following configuration:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name yourdomain.com www.yourdomain.com;

    # Increase max upload size for large video URLs/metadata
    client_max_body_size 50M;

    location / {
        proxy_pass http://127.0.0.1:8081;
        proxy_http_version 1.1;
        
        # WebSocket support (required for real-time updates)
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # Standard proxy headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts for long downloads
        proxy_connect_timeout 7d;
        proxy_send_timeout 7d;
        proxy_read_timeout 7d;
    }
}
```

### 2. Enable the Site

```bash
# Create symbolic link to enable the site
sudo ln -s /etc/nginx/sites-available/metube /etc/nginx/sites-enabled/

# Remove default site (optional)
sudo rm /etc/nginx/sites-enabled/default
```

### 3. Test Nginx Configuration

```bash
sudo nginx -t
```

You should see:
```
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### 4. Reload Nginx

```bash
sudo systemctl reload nginx
```

### 5. Test Access

Visit `http://yourdomain.com` in your browser. You should see the Metube interface.

---

## Set Up SSL with Let's Encrypt

### 1. Install Certbot

```bash
sudo apt install -y certbot python3-certbot-nginx
```

### 2. Obtain SSL Certificate

Replace `yourdomain.com` with your domain:

```bash
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

Follow the prompts:
- Enter your email address
- Agree to terms of service
- Choose whether to redirect HTTP to HTTPS (recommended: Yes)

### 3. Verify Auto-Renewal

```bash
# Test renewal process
sudo certbot renew --dry-run

# Check certbot timer status
sudo systemctl status certbot.timer
```

### 4. Updated Nginx Configuration

After Certbot runs, your `/etc/nginx/sites-available/metube` file will be automatically updated with SSL configuration. It should look similar to this:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name yourdomain.com www.yourdomain.com;
    
    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name yourdomain.com www.yourdomain.com;

    # SSL certificates (managed by Certbot)
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    client_max_body_size 50M;

    location / {
        proxy_pass http://127.0.0.1:8081;
        proxy_http_version 1.1;
        
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        proxy_connect_timeout 7d;
        proxy_send_timeout 7d;
        proxy_read_timeout 7d;
    }
}
```

### 5. Test HTTPS Access

Visit `https://yourdomain.com` - you should see a secure connection (padlock icon).

---

## Maintenance and Updates

### Updating Metube

Metube receives nightly builds with the latest yt-dlp version. Update regularly:

```bash
cd ~/metube

# Pull latest image
docker-compose pull

# Restart with new image
docker-compose up -d

# Remove old images
docker image prune -a
```

### Automatic Updates with Watchtower

Install Watchtower to automatically update Docker containers:

```bash
docker run -d \
  --name watchtower \
  --restart unless-stopped \
  -v /var/run/docker.sock:/var/run/docker.sock \
  containrrr/watchtower \
  --cleanup \
  --interval 86400
```

This checks for updates daily and automatically updates containers.

### Backup Downloads

```bash
# Create backup directory
mkdir -p ~/backups

# Backup downloads
tar -czf ~/backups/metube-downloads-$(date +%Y%m%d).tar.gz ~/metube/downloads/

# Keep only last 7 days of backups
find ~/backups -name "metube-downloads-*.tar.gz" -mtime +7 -delete
```

### View Logs

```bash
# Metube logs
cd ~/metube
docker-compose logs -f

# Nginx access logs
sudo tail -f /var/log/nginx/access.log

# Nginx error logs
sudo tail -f /var/log/nginx/error.log
```

---

## Troubleshooting

### Metube Container Won't Start

```bash
# Check container status
docker-compose ps

# View detailed logs
docker-compose logs

# Check if port is already in use
sudo netstat -tulpn | grep 8081

# Restart container
docker-compose restart
```

### Cannot Access Metube Through Domain

```bash
# Check Nginx status
sudo systemctl status nginx

# Test Nginx configuration
sudo nginx -t

# Check if domain resolves correctly
nslookup yourdomain.com

# Check UFW rules
sudo ufw status verbose

# Test local access
curl http://localhost:8081
```

### Permission Issues with Downloads

```bash
# Check directory ownership
ls -la ~/metube/downloads

# Fix ownership (use your UID/GID from 'id' command)
sudo chown -R 1000:1000 ~/metube/downloads

# Check Docker container user
docker-compose exec metube id
```

### WebSocket Connection Errors

Ensure these headers are in your Nginx configuration:
```nginx
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
```

### SSL Certificate Issues

```bash
# Renew certificates manually
sudo certbot renew

# Force renewal
sudo certbot renew --force-renewal

# Check certificate expiry
sudo certbot certificates
```

### High Memory Usage

Limit concurrent downloads in `docker-compose.yml`:
```yaml
environment:
  - DOWNLOAD_MODE=limited
  - MAX_CONCURRENT_DOWNLOADS=2
```

### Downloads Failing

```bash
# Update yt-dlp by updating Metube
cd ~/metube
docker-compose pull
docker-compose up -d

# Check yt-dlp directly in container
docker-compose exec metube sh
cd /downloads
yt-dlp --version
```

---

## Advanced Configuration

### Custom Download Directory Structure

Edit `docker-compose.yml`:

```yaml
environment:
  - OUTPUT_TEMPLATE=%(uploader)s/%(upload_date)s - %(title)s.%(ext)s
  - CUSTOM_DIRS=true
```

### Enable Cookies for Private Videos

```bash
# Create cookies directory
mkdir -p ~/metube/cookies

# Get cookies.txt from browser using extension
# Place it in ~/metube/cookies/

# Update docker-compose.yml
volumes:
  - ./downloads:/downloads
  - ./cookies:/cookies
environment:
  - YTDL_OPTIONS={"cookiefile":"/cookies/cookies.txt"}
```

### Add Basic Authentication

Edit `/etc/nginx/sites-available/metube`:

```bash
# Install apache2-utils
sudo apt install -y apache2-utils

# Create password file
sudo htpasswd -c /etc/nginx/.htpasswd yourusername
```

Add to Nginx location block:
```nginx
location / {
    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/.htpasswd;
    
    proxy_pass http://127.0.0.1:8081;
    # ... rest of configuration
}
```

Reload Nginx:
```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## Security Best Practices

1. **Keep everything updated:**
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Configure SSH key authentication** and disable password authentication

3. **Use strong passwords** for any authentication

4. **Regular backups** of your downloads directory

5. **Monitor logs** regularly for suspicious activity

6. **Consider fail2ban** for additional protection:
   ```bash
   sudo apt install -y fail2ban
   ```

7. **Limit rate** in Nginx to prevent abuse:
   ```nginx
   limit_req_zone $binary_remote_addr zone=metube:10m rate=10r/s;
   
   location / {
       limit_req zone=metube burst=20;
       # ... rest of configuration
   }
   ```

---

## Useful Commands Reference

```bash
# Docker Compose
docker-compose up -d              # Start containers
docker-compose down               # Stop containers
docker-compose restart            # Restart containers
docker-compose logs -f            # View logs
docker-compose pull               # Update images
docker-compose ps                 # List containers

# Nginx
sudo systemctl start nginx        # Start Nginx
sudo systemctl stop nginx         # Stop Nginx
sudo systemctl restart nginx      # Restart Nginx
sudo systemctl reload nginx       # Reload config
sudo nginx -t                     # Test configuration

# UFW
sudo ufw status                   # Check firewall status
sudo ufw allow 80/tcp             # Allow HTTP
sudo ufw deny from 1.2.3.4        # Block specific IP
sudo ufw delete allow 80/tcp      # Remove rule

# SSL/Certbot
sudo certbot renew                # Renew certificates
sudo certbot certificates         # List certificates
sudo certbot delete               # Delete certificate
```

---

## Conclusion

You now have a fully functional Metube instance running on your VPS with:
- ✅ Docker containerization for easy management
- ✅ UFW firewall protection
- ✅ Nginx reverse proxy with custom domain
- ✅ SSL/HTTPS encryption
- ✅ Automatic updates support

Access your Metube instance at `https://yourdomain.com` and start downloading videos!

For more information and advanced configurations, visit:
- [Metube GitHub Repository](https://github.com/alexta69/metube)
- [Metube Wiki](https://github.com/alexta69/metube/wiki)
- [yt-dlp Documentation](https://github.com/yt-dlp/yt-dlp)

---

## Support and Resources

- **Issues:** Report bugs on [GitHub Issues](https://github.com/alexta69/metube/issues)
- **Documentation:** [Metube README](https://github.com/alexta69/metube/blob/master/README.md)
- **yt-dlp:** [Supported Sites](https://github.com/yt-dlp/yt-dlp/blob/master/supportedsites.md)
