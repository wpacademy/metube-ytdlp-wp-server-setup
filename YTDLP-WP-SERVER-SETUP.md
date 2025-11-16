# YT-DLP WordPress Plugin - Complete Server Setup Guide

A comprehensive guide for setting up the YT-DLP WordPress plugin on various hosting platforms including VPS, cPanel, shared hosting, and cloud platforms.

---

## Table of Contents

1. [Overview](#overview)
2. [Server Requirements](#server-requirements)
3. [VPS Setup (Ubuntu/Debian)](#vps-setup-ubuntudebian)
4. [VPS Setup (CentOS/RHEL)](#vps-setup-centosrhel)
5. [cPanel/WHM Setup](#cpanelwhm-setup)
6. [Shared Hosting Setup](#shared-hosting-setup)
7. [Cloud Platform Guides](#cloud-platform-guides)
8. [Windows Server Setup](#windows-server-setup)
9. [PHP Configuration](#php-configuration)
10. [Security Hardening](#security-hardening)
11. [Performance Optimization](#performance-optimization)
12. [Troubleshooting](#troubleshooting)
13. [Maintenance](#maintenance)

---

## Overview

### What This Guide Covers

This guide will help you set up all required components for the YT-DLP WordPress plugin:

- **yt-dlp** - Video download utility
- **FFmpeg** - Audio/video processing library
- **PHP Configuration** - Required settings and extensions
- **File Permissions** - Proper security settings
- **WordPress Configuration** - Plugin-specific settings

### Prerequisites

- Root or sudo access (VPS)
- SSH access (VPS) or cPanel access (shared hosting)
- WordPress 5.8+ installed
- PHP 7.4+ available
- At least 2GB RAM (recommended)
- 10GB+ free disk space (for temporary downloads)

---

## Server Requirements

### Minimum Requirements

| Component | Requirement |
|-----------|-------------|
| PHP Version | 7.4 or higher |
| WordPress | 5.8 or higher |
| RAM | 1GB minimum, 2GB+ recommended |
| Disk Space | 10GB+ free space |
| CPU | 1 core minimum, 2+ recommended |
| Shell Access | Required for yt-dlp execution |

### Required PHP Extensions

```
- curl
- json
- mbstring
- openssl
- zip
- gd or imagick (for thumbnail handling)
```

### Required PHP Functions (Must NOT be disabled)

```
- shell_exec
- exec
- proc_open
- escapeshellarg
- escapeshellcmd
```

### System Binaries Required

```
- yt-dlp (mandatory)
- ffmpeg (mandatory for format conversion)
- python3 (recommended for yt-dlp)
- wget or curl (for downloading yt-dlp)
```

---

## VPS Setup (Ubuntu/Debian)

### Step 1: System Update

```bash
# Update system packages
sudo apt-get update
sudo apt-get upgrade -y

# Install essential build tools
sudo apt-get install -y build-essential curl wget git unzip
```

### Step 2: Install Python3 and pip

```bash
# Install Python3 (usually pre-installed)
sudo apt-get install -y python3 python3-pip python3-dev

# Verify installation
python3 --version
pip3 --version
```

### Step 3: Install yt-dlp

**Method 1: Direct Binary Download (Recommended)**

```bash
# Download latest yt-dlp binary
sudo wget https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp -O /usr/local/bin/yt-dlp

# Make it executable
sudo chmod a+rx /usr/local/bin/yt-dlp

# Create symbolic link (optional)
sudo ln -s /usr/local/bin/yt-dlp /usr/bin/yt-dlp

# Verify installation
yt-dlp --version
```

**Method 2: Using pip (Alternative)**

```bash
# Install via pip
sudo pip3 install yt-dlp

# Verify installation
yt-dlp --version

# Find installation path
which yt-dlp
# Usually: /usr/local/bin/yt-dlp or /usr/bin/yt-dlp
```

**Method 3: Using Package Manager (Alternative)**

```bash
# Add repository (if not available)
sudo add-apt-repository ppa:tomtomtom/yt-dlp
sudo apt-get update

# Install
sudo apt-get install yt-dlp

# Verify
yt-dlp --version
```

### Step 4: Install FFmpeg

```bash
# Install FFmpeg from official repositories
sudo apt-get install -y ffmpeg

# Verify installation
ffmpeg -version

# Check installation path
which ffmpeg
# Usually: /usr/bin/ffmpeg
```

**Installing Latest FFmpeg (Optional)**

```bash
# Add PPA for latest version
sudo add-apt-repository ppa:jonathonf/ffmpeg-4
sudo apt-get update
sudo apt-get install -y ffmpeg

# Verify
ffmpeg -version
```

### Step 5: Install and Configure PHP

```bash
# Install PHP 8.1 (adjust version as needed)
sudo apt-get install -y php8.1 php8.1-cli php8.1-fpm php8.1-mysql \
    php8.1-curl php8.1-json php8.1-mbstring php8.1-xml \
    php8.1-zip php8.1-gd php8.1-imagick

# Verify PHP version
php -v

# Check installed extensions
php -m
```

### Step 6: Configure PHP for WordPress

```bash
# Edit PHP configuration
sudo nano /etc/php/8.1/fpm/php.ini

# Find and modify these values:
```

Add/modify the following in `php.ini`:

```ini
; Increase memory limit
memory_limit = 512M

; Increase maximum execution time
max_execution_time = 600

; Increase maximum input time
max_input_time = 600

; Increase upload file size
upload_max_filesize = 500M
post_max_size = 500M

; Ensure these functions are NOT disabled
disable_functions = 

; Enable error logging
log_errors = On
error_log = /var/log/php_errors.log
```

**Restart PHP-FPM:**

```bash
sudo systemctl restart php8.1-fpm
```

### Step 7: Configure WordPress Directory Permissions

```bash
# Navigate to WordPress root
cd /var/www/html  # or your WordPress installation path

# Set proper ownership
sudo chown -R www-data:www-data wp-content/uploads/

# Set directory permissions
sudo find wp-content/uploads/ -type d -exec chmod 755 {} \;

# Set file permissions
sudo find wp-content/uploads/ -type f -exec chmod 644 {} \;
```

### Step 8: Install WordPress Plugin

```bash
# Navigate to plugins directory
cd /var/www/html/wp-content/plugins/

# Create plugin directory
sudo mkdir -p yt-dlp-downloader

# Upload plugin files (via SFTP or wget)
# Example using wget from GitHub:
sudo wget https://github.com/your-repo/plugin.zip
sudo unzip plugin.zip -d yt-dlp-downloader/

# Set proper permissions
sudo chown -R www-data:www-data yt-dlp-downloader/
sudo chmod -R 755 yt-dlp-downloader/
```

### Step 9: Configure Plugin Settings

1. Log in to WordPress Admin
2. Go to **Plugins** → Activate "YT-DLP Video Downloader"
3. Go to **YT-DLP WP** → **Settings**
4. Configure paths:
   - **YT-DLP Path**: `/usr/local/bin/yt-dlp`
   - **FFmpeg Path**: `/usr/bin/ffmpeg`
5. Save settings

### Step 10: Test Installation

```bash
# Test yt-dlp directly
yt-dlp --version

# Test yt-dlp with a video
yt-dlp --skip-download --print-json "https://www.youtube.com/watch?v=dQw4w9WgXcQ"

# Test FFmpeg
ffmpeg -version

# Test from PHP
php -r "echo shell_exec('yt-dlp --version');"
```

### Step 11: Create Automatic Updates (Optional)

```bash
# Create update script
sudo nano /usr/local/bin/update-ytdlp.sh
```

Add this content:

```bash
#!/bin/bash
# Auto-update yt-dlp

wget https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp -O /usr/local/bin/yt-dlp
chmod a+rx /usr/local/bin/yt-dlp
echo "yt-dlp updated successfully on $(date)" >> /var/log/ytdlp-updates.log
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/update-ytdlp.sh
```

Add to crontab (weekly updates):

```bash
sudo crontab -e
```

Add this line:

```
0 3 * * 0 /usr/local/bin/update-ytdlp.sh
```

---

## VPS Setup (CentOS/RHEL)

### Step 1: System Update

```bash
# Update system
sudo yum update -y

# Install EPEL repository
sudo yum install -y epel-release

# Install development tools
sudo yum groupinstall -y "Development Tools"
sudo yum install -y curl wget git unzip
```

### Step 2: Install Python3

```bash
# Install Python 3
sudo yum install -y python3 python3-pip python3-devel

# Verify
python3 --version
pip3 --version
```

### Step 3: Install yt-dlp

```bash
# Download binary
sudo wget https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp -O /usr/local/bin/yt-dlp

# Make executable
sudo chmod a+rx /usr/local/bin/yt-dlp

# Verify
yt-dlp --version
```

### Step 4: Install FFmpeg

**Enable RPM Fusion Repository:**

```bash
# Install RPM Fusion Free
sudo yum install -y https://download1.rpmfusion.org/free/el/rpmfusion-free-release-$(rpm -E %rhel).noarch.rpm

# Install FFmpeg
sudo yum install -y ffmpeg ffmpeg-devel

# Verify
ffmpeg -version
```

**Alternative: Compile from Source**

```bash
# Install dependencies
sudo yum install -y autoconf automake bzip2 cmake freetype-devel \
    gcc gcc-c++ git libtool make mercurial pkgconfig zlib-devel

# Download and compile (this takes time)
cd /tmp
git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg
cd ffmpeg
./configure --enable-gpl --enable-nonfree
make
sudo make install

# Verify
ffmpeg -version
```

### Step 5: Install PHP

```bash
# Install Remi repository
sudo yum install -y https://rpms.remirepo.net/enterprise/remi-release-$(rpm -E %rhel).rpm

# Enable PHP 8.1 module
sudo yum module enable php:remi-8.1 -y

# Install PHP and extensions
sudo yum install -y php php-cli php-fpm php-mysqlnd php-curl \
    php-json php-mbstring php-xml php-zip php-gd php-imagick

# Verify
php -v
```

### Step 6: Configure PHP

```bash
# Edit PHP configuration
sudo nano /etc/php.ini
```

Apply same php.ini changes as Ubuntu section above.

```bash
# Restart PHP-FPM
sudo systemctl restart php-fpm
sudo systemctl enable php-fpm
```

### Step 7-11: Same as Ubuntu

Follow steps 7-11 from the Ubuntu/Debian section above.

---

## cPanel/WHM Setup

### Prerequisites

- Root/WHM access or cPanel account access
- SSH access (root or jailed shell)
- cPanel version 11.80 or higher

### Step 1: Access SSH Terminal

**Option A: WHM Terminal**
1. Log in to WHM
2. Go to **Server Configuration** → **Terminal**

**Option B: SSH Client**
```bash
ssh username@your-server.com
```

### Step 2: Check Current Environment

```bash
# Check available Python versions
python3 --version

# Check PHP version
php -v

# Check shell_exec availability
php -r "var_dump(function_exists('shell_exec'));"
```

### Step 3: Install yt-dlp

**Method 1: In User's Home Directory (Recommended for Shared Hosting)**

```bash
# Navigate to home directory
cd ~

# Create a bin directory
mkdir -p bin

# Download yt-dlp
wget https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp -O ~/bin/yt-dlp

# Make executable
chmod +x ~/bin/yt-dlp

# Get full path (save this for plugin settings)
realpath ~/bin/yt-dlp
# Example output: /home/username/bin/yt-dlp

# Test
~/bin/yt-dlp --version
```

**Method 2: System-Wide Installation (Requires Root)**

```bash
# As root user
sudo wget https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp -O /usr/local/bin/yt-dlp
sudo chmod a+rx /usr/local/bin/yt-dlp

# Verify
yt-dlp --version
```

### Step 4: Install FFmpeg

**Option A: Using cPanel EasyApache4 (WHM Access Required)**

1. Log in to WHM
2. Go to **Software** → **EasyApache 4**
3. Click **Customize**
4. Search for "ffmpeg"
5. Enable "ea-ffmpeg" package
6. Click **Review** and **Provision**

**Option B: Manual Installation in Home Directory**

```bash
# Download static build
cd ~/bin
wget https://johnvansickle.com/ffmpeg/releases/ffmpeg-release-amd64-static.tar.xz

# Extract
tar xvf ffmpeg-release-amd64-static.tar.xz

# Copy binaries
cp ffmpeg-*-static/ffmpeg .
cp ffmpeg-*-static/ffprobe .

# Clean up
rm -rf ffmpeg-*-static*

# Make executable
chmod +x ffmpeg ffprobe

# Get full path
realpath ~/bin/ffmpeg
# Example: /home/username/bin/ffmpeg

# Test
~/bin/ffmpeg -version
```

**Option C: Request from Hosting Provider**

Contact your hosting provider and request FFmpeg installation.

### Step 5: Configure PHP Settings via cPanel

**Method 1: Using MultiPHP INI Editor**

1. Log in to cPanel
2. Go to **Software** → **MultiPHP INI Editor**
3. Select your domain
4. Modify these values:

```ini
memory_limit = 512M
max_execution_time = 600
max_input_time = 600
upload_max_filesize = 500M
post_max_size = 500M
```

5. Click **Apply**

**Method 2: Using .htaccess (If MultiPHP Editor not available)**

Create/edit `.htaccess` in your WordPress root:

```apache
# Add these PHP settings
php_value memory_limit 512M
php_value max_execution_time 600
php_value upload_max_filesize 500M
php_value post_max_size 500M
```

**Method 3: Using php.ini (If Available)**

If your hosting allows custom php.ini:

```bash
cd ~/public_html  # or your WordPress directory
nano php.ini
```

Add the same values from the Ubuntu php.ini section.

### Step 6: Verify PHP Functions

```bash
# Create a test file
cd ~/public_html
nano test-shell.php
```

Add this content:

```php
<?php
// Test shell_exec
echo "Testing shell_exec: ";
$result = shell_exec('echo "Hello from shell"');
echo $result ? "WORKS ✓" : "DISABLED ✗";
echo "\n\n";

// Test yt-dlp
echo "Testing yt-dlp: ";
$ytdlp = shell_exec('~/bin/yt-dlp --version 2>&1');
echo $ytdlp ? "WORKS ✓\nVersion: " . $ytdlp : "FAILED ✗";
echo "\n\n";

// Test FFmpeg
echo "Testing FFmpeg: ";
$ffmpeg = shell_exec('~/bin/ffmpeg -version 2>&1');
echo $ffmpeg ? "WORKS ✓" : "FAILED ✗";
```

Access via browser: `https://yourdomain.com/test-shell.php`

**Important**: Delete this file after testing!

```bash
rm ~/public_html/test-shell.php
```

### Step 7: Install Plugin via cPanel

**Method A: Using WordPress Admin (Recommended)**

1. Log in to WordPress Admin
2. Go to **Plugins** → **Add New** → **Upload Plugin**
3. Upload plugin ZIP file
4. Activate plugin

**Method B: Using cPanel File Manager**

1. Log in to cPanel
2. Open **File Manager**
3. Navigate to `public_html/wp-content/plugins/`
4. Upload plugin ZIP file
5. Extract the ZIP file
6. Go to WordPress Admin → **Plugins** → Activate

### Step 8: Configure Plugin Paths

1. Go to WordPress Admin → **YT-DLP WP** → **Settings**
2. Set paths based on your installation:
   - **YT-DLP Path**: `/home/username/bin/yt-dlp` (from Step 3)
   - **FFmpeg Path**: `/home/username/bin/ffmpeg` (from Step 4)
3. Save settings

### Step 9: Test in WordPress

1. Create a test page
2. Add shortcode: `[ytdlp_downloader]`
3. Try downloading a test video
4. Check for errors

### Troubleshooting cPanel Setup

**Issue: "shell_exec is disabled"**

```bash
# Check disabled functions
php -r "echo ini_get('disable_functions');"

# If shell_exec is listed, contact hosting provider
```

**Issue: "Permission denied" errors**

```bash
# Fix permissions
chmod +x ~/bin/yt-dlp
chmod +x ~/bin/ffmpeg

# Fix uploads directory
cd ~/public_html/wp-content
chmod -R 755 uploads/
```

**Issue: Path not found**

Always use absolute paths in plugin settings:
```
Correct: /home/username/bin/yt-dlp
Wrong: ~/bin/yt-dlp
Wrong: bin/yt-dlp
```

---

## Shared Hosting Setup

### Overview

Shared hosting has more limitations:
- Limited or no SSH access
- Restricted PHP functions
- No root access
- Limited resources

### Compatibility Check

**Before proceeding, verify these with your host:**

1. ✓ PHP 7.4+ available
2. ✓ shell_exec() NOT disabled
3. ✓ Ability to upload custom binaries
4. ✓ At least 512MB PHP memory limit
5. ✓ At least 300 seconds execution time

**If ANY of these are not available, this plugin will NOT work.**

### Step 1: Download Required Binaries Locally

On your local computer, download static builds:

**yt-dlp:**
- Download from: https://github.com/yt-dlp/yt-dlp/releases/latest
- Choose: `yt-dlp` (not the .tar.gz)

**FFmpeg:**
- Download from: https://johnvansickle.com/ffmpeg/releases/ffmpeg-release-amd64-static.tar.xz
- Extract and find the `ffmpeg` binary inside

### Step 2: Upload via FTP/File Manager

**Using FTP (FileZilla, etc.):**

```
1. Connect to your hosting via FTP
2. Navigate to: /public_html/ or your WordPress root
3. Create folder: bin/
4. Upload: yt-dlp to bin/
5. Upload: ffmpeg to bin/
6. Set permissions: 755 (executable) for both files
```

**Using cPanel File Manager:**

1. Log in to cPanel
2. Open File Manager
3. Navigate to public_html/
4. Create folder: `bin`
5. Upload both files
6. Right-click each file → Permissions → 755

### Step 3: Get Absolute Paths

Create a temporary PHP file to find paths:

```php
<?php
// Save as path-finder.php in public_html/
echo "Document root: " . $_SERVER['DOCUMENT_ROOT'] . "\n";
echo "Current directory: " . __DIR__ . "\n";
echo "\nyt-dlp path: " . realpath(__DIR__ . '/bin/yt-dlp') . "\n";
echo "ffmpeg path: " . realpath(__DIR__ . '/bin/ffmpeg') . "\n";
```

Access: `https://yourdomain.com/path-finder.php`

Copy the paths shown. Delete the file after.

### Step 4: Install WordPress Plugin

Use WordPress Admin:
1. **Plugins** → **Add New** → **Upload Plugin**
2. Upload and activate

### Step 5: Configure Plugin

1. **YT-DLP WP** → **Settings**
2. Enter the paths from Step 3:
   - YT-DLP Path: (e.g., `/home/username/public_html/bin/yt-dlp`)
   - FFmpeg Path: (e.g., `/home/username/public_html/bin/ffmpeg`)
3. Set conservative limits:
   - Max File Size: 100MB (to avoid timeouts)
   - Timeout: 300 seconds
4. Enable logging for debugging
5. Save

### Step 6: Optimize for Shared Hosting

Add this to your WordPress `wp-config.php`:

```php
// Increase WordPress memory limit
define('WP_MEMORY_LIMIT', '512M');
define('WP_MAX_MEMORY_LIMIT', '512M');

// Set timezone
date_default_timezone_set('UTC');
```

### Step 7: Test Carefully

Start with short videos:
1. YouTube short videos (< 1 minute)
2. Low quality options
3. MP3 audio only

### Limitations on Shared Hosting

**You may experience:**
- Slower download speeds
- Timeouts on large files
- Occasional failures during peak times
- Limited concurrent downloads
- Host may block after heavy usage

**Not Recommended For:**
- High-traffic public sites
- Long videos (>10 minutes)
- High-resolution downloads (4K)
- Multiple simultaneous users

**Alternative Solutions:**
- Upgrade to VPS hosting
- Use cloud-based solutions
- Implement queue system
- Add caching layer

---

## Cloud Platform Guides

### AWS EC2 Setup

**Launch Instance:**

1. Choose Ubuntu 22.04 LTS AMI
2. Instance type: t2.medium or larger
3. Configure security group:
   - Allow SSH (22)
   - Allow HTTP (80)
   - Allow HTTPS (443)
4. Create and download key pair

**Connect and Setup:**

```bash
# Connect to instance
ssh -i your-key.pem ubuntu@ec2-xx-xxx-xxx-xxx.compute.amazonaws.com

# Follow VPS Setup (Ubuntu/Debian) section above
```

**Additional AWS-Specific Steps:**

```bash
# Install AWS CLI (optional)
sudo apt-get install -y awscli

# Configure swap for better performance
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

### Google Cloud Platform (GCP)

**Create VM Instance:**

1. Go to Compute Engine → VM Instances
2. Click "Create Instance"
3. Choose: Ubuntu 22.04 LTS
4. Machine type: e2-medium or larger
5. Allow HTTP/HTTPS traffic
6. Create

**Connect:**

```bash
# Use browser SSH or gcloud CLI
gcloud compute ssh instance-name
```

**Setup:**
Follow VPS Setup (Ubuntu/Debian) section.

### DigitalOcean

**Create Droplet:**

1. Choose Ubuntu 22.04 LTS
2. Plan: Basic - $12/month or higher
3. Add SSH key
4. Create

**Connect:**

```bash
ssh root@your-droplet-ip
```

**Quick Setup Script:**

```bash
# Download and run automated setup script
wget https://raw.githubusercontent.com/your-repo/setup.sh
chmod +x setup.sh
sudo ./setup.sh
```

### Microsoft Azure

**Create Virtual Machine:**

1. Resource → Virtual Machines → Create
2. Image: Ubuntu Server 22.04 LTS
3. Size: Standard_B2s or larger
4. Configure networking
5. Review and create

**Connect:**

```bash
ssh azureuser@your-vm-ip
```

**Setup:**
Follow VPS Setup (Ubuntu/Debian) section.

### Linode

**Create Linode:**

1. Select: Ubuntu 22.04 LTS
2. Plan: Nanode 1GB or higher
3. Select region
4. Set root password
5. Create

**Connect:**

```bash
ssh root@your-linode-ip
```

**Setup:**
Follow VPS Setup (Ubuntu/Debian) section.

---

## Windows Server Setup

### Prerequisites

- Windows Server 2016 or higher
- Administrator access
- IIS with PHP installed
- WordPress running

### Step 1: Install Python

**Download and Install:**

1. Go to https://www.python.org/downloads/windows/
2. Download latest Python 3.x installer
3. Run installer
4. **IMPORTANT**: Check "Add Python to PATH"
5. Click "Install Now"

**Verify:**

```cmd
python --version
pip --version
```

### Step 2: Install yt-dlp

**Method 1: Using pip**

```cmd
pip install yt-dlp

# Find installation path
where yt-dlp
# Example: C:\Python3\Scripts\yt-dlp.exe
```

**Method 2: Download Binary**

1. Download from: https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp.exe
2. Save to: `C:\Program Files\yt-dlp\yt-dlp.exe`
3. Note the path

### Step 3: Install FFmpeg

**Download FFmpeg:**

1. Go to: https://www.gyan.dev/ffmpeg/builds/
2. Download: ffmpeg-release-essentials.zip
3. Extract to: `C:\Program Files\ffmpeg\`
4. Add to PATH:
   ```
   - Right-click "This PC" → Properties
   - Advanced System Settings
   - Environment Variables
   - Edit "Path"
   - Add: C:\Program Files\ffmpeg\bin
   ```

**Verify:**

```cmd
ffmpeg -version
```

### Step 4: Configure PHP

**Edit php.ini:**

Location: `C:\Program Files\PHP\php.ini`

```ini
memory_limit = 512M
max_execution_time = 600
upload_max_filesize = 500M
post_max_size = 500M

; Ensure these are NOT in disable_functions
; disable_functions = 
```

**Restart IIS:**

```cmd
iisreset
```

### Step 5: Set File Permissions

**Configure uploads folder:**

1. Navigate to: `C:\inetpub\wwwroot\wp-content\uploads\`
2. Right-click → Properties → Security
3. Edit permissions:
   - Add "IIS_IUSRS" group
   - Grant: Modify, Read & Execute, List folder contents, Read, Write
4. Apply to all subfolders

### Step 6: Install Plugin

1. Download plugin ZIP
2. WordPress Admin → Plugins → Add New → Upload
3. Activate plugin

### Step 7: Configure Plugin Paths

**Windows path format:**

```
YT-DLP Path: C:\Program Files\yt-dlp\yt-dlp.exe
FFmpeg Path: C:\Program Files\ffmpeg\bin\ffmpeg.exe

OR (if using pip):
YT-DLP Path: C:\Python3\Scripts\yt-dlp.exe
```

**Important**: Use backslashes `\` not forward slashes `/`

### Step 8: Test

Create test file: `C:\inetpub\wwwroot\test-windows.php`

```php
<?php
$ytdlp = shell_exec('C:\Python3\Scripts\yt-dlp.exe --version');
echo "yt-dlp: " . $ytdlp . "\n";

$ffmpeg = shell_exec('C:\Program Files\ffmpeg\bin\ffmpeg.exe -version');
echo "ffmpeg: " . (strlen($ffmpeg) > 0 ? "WORKS" : "FAILED");
```

### Windows-Specific Issues

**Issue: "shell_exec not working"**

Check PHP configuration:
```ini
; Ensure these are empty
disable_functions = 
```

**Issue: "Permission denied"**

Grant IIS_IUSRS permissions to:
- PHP executable directory
- yt-dlp installation directory
- WordPress uploads directory

**Issue: Antivirus blocking**

Add exclusions for:
- yt-dlp.exe
- ffmpeg.exe
- WordPress uploads folder

---

## PHP Configuration

### Required php.ini Settings

```ini
[PHP]
; Basic settings
max_execution_time = 600
max_input_time = 600
memory_limit = 512M

; Upload settings
upload_max_filesize = 500M
post_max_size = 500M
max_file_uploads = 20

; Error handling
display_errors = Off
log_errors = On
error_log = /var/log/php_errors.log

; Security
allow_url_fopen = On
allow_url_include = Off

; DO NOT disable these functions
disable_functions = 

; Session settings
session.save_path = "/tmp"
session.gc_maxlifetime = 3600
```

### PHP Extensions Required

**Verify installed extensions:**

```bash
php -m
```

**Required extensions:**

```
- curl
- json
- mbstring
- openssl
- zip
- gd or imagick
- mysqli or pdo_mysql
```

**Install missing extensions (Ubuntu):**

```bash
sudo apt-get install php8.1-curl php8.1-json php8.1-mbstring \
    php8.1-xml php8.1-zip php8.1-gd php8.1-imagick
```

### PHP-FPM Configuration (If Applicable)

Edit: `/etc/php/8.1/fpm/pool.d/www.conf`

```ini
[www]
user = www-data
group = www-data
listen = /run/php/php8.1-fpm.sock
listen.owner = www-data
listen.group = www-data
listen.mode = 0660

; Process management
pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
pm.max_requests = 500

; Timeout settings
request_terminate_timeout = 600s
```

Restart:

```bash
sudo systemctl restart php8.1-fpm
```

### Apache Configuration

**Enable required modules:**

```bash
sudo a2enmod rewrite
sudo a2enmod headers
sudo a2enmod expires
sudo systemctl restart apache2
```

**Add to .htaccess or VirtualHost:**

```apache
<IfModule mod_php8.c>
    php_value memory_limit 512M
    php_value max_execution_time 600
    php_value upload_max_filesize 500M
    php_value post_max_size 500M
</IfModule>
```

### Nginx Configuration

**Edit site config:**

```nginx
server {
    listen 80;
    server_name yourdomain.com;
    root /var/www/html;
    index index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        
        # Timeout settings
        fastcgi_read_timeout 600s;
        fastcgi_send_timeout 600s;
    }

    # Client body size for uploads
    client_max_body_size 500M;
    client_body_timeout 600s;
}
```

Restart:

```bash
sudo systemctl restart nginx
```

---

## Security Hardening

### File Permissions

**WordPress directory structure:**

```bash
# Base directory
chmod 755 /var/www/html

# wp-config.php
chmod 600 /var/www/html/wp-config.php

# wp-content
chmod 755 /var/www/html/wp-content

# Plugins
chmod 755 /var/www/html/wp-content/plugins
find /var/www/html/wp-content/plugins -type f -exec chmod 644 {} \;
find /var/www/html/wp-content/plugins -type d -exec chmod 755 {} \;

# Uploads
chmod 755 /var/www/html/wp-content/uploads
find /var/www/html/wp-content/uploads -type f -exec chmod 644 {} \;
find /var/www/html/wp-content/uploads -type d -exec chmod 755 {} \;
```

### Restrict Download Directory Access

**Create .htaccess in download folder:**

```bash
sudo nano /var/www/html/wp-content/uploads/yt-dlp-downloads/.htaccess
```

Add:

```apache
# Deny direct access
Options -Indexes

# Protect against scripts
<FilesMatch "\.(php|php3|php4|php5|phtml)$">
    Order Deny,Allow
    Deny from all
</FilesMatch>

# Only allow specific file types to be served
<FilesMatch "\.(mp4|webm|mkv|mp3|m4a|wav|flac)$">
    Order Allow,Deny
    Allow from all
</FilesMatch>
```

### Firewall Configuration

**Ubuntu/Debian (UFW):**

```bash
# Install UFW
sudo apt-get install ufw

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH
sudo ufw allow 22/tcp

# Allow HTTP/HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable
sudo ufw enable

# Check status
sudo ufw status
```

**CentOS/RHEL (firewalld):**

```bash
# Start firewalld
sudo systemctl start firewalld
sudo systemctl enable firewalld

# Allow services
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-service=ssh

# Reload
sudo firewall-cmd --reload
```

### Rate Limiting (Nginx)

**Limit download requests:**

```nginx
http {
    # Define rate limit zone
    limit_req_zone $binary_remote_addr zone=downloads:10m rate=5r/m;
    
    server {
        location ~ /wp-content/uploads/yt-dlp-downloads/ {
            limit_req zone=downloads burst=10 nodelay;
        }
    }
}
```

### Fail2ban Setup (Optional)

```bash
# Install
sudo apt-get install fail2ban

# Create WordPress filter
sudo nano /etc/fail2ban/filter.d/wordpress-hard.conf
```

Add:

```ini
[Definition]
failregex = ^<HOST> .* "POST .*wp-login.php
            ^<HOST> .* "POST .*wp-admin
            ^<HOST> .* "POST .*xmlrpc.php
ignoreregex =
```

Configure jail:

```bash
sudo nano /etc/fail2ban/jail.local
```

Add:

```ini
[wordpress-hard]
enabled = true
filter = wordpress-hard
logpath = /var/log/nginx/access.log
maxretry = 3
port = http,https
bantime = 3600
```

Restart:

```bash
sudo systemctl restart fail2ban
```

### SSL/TLS Configuration

**Using Let's Encrypt (Free):**

```bash
# Install Certbot
sudo apt-get install certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Auto-renewal (should be automatic)
sudo certbot renew --dry-run
```

---

## Performance Optimization

### Optimize yt-dlp Performance

**Create config file:**

```bash
# User config
mkdir -p ~/.config/yt-dlp
nano ~/.config/yt-dlp/config
```

Add:

```
# Download settings
--no-check-certificates
--prefer-free-formats
--no-playlist

# Network optimization
--concurrent-fragments 5
--buffer-size 16K
--http-chunk-size 10M

# Caching
--cache-dir /tmp/yt-dlp-cache
```

### PHP OpCache

**Enable in php.ini:**

```ini
[opcache]
opcache.enable=1
opcache.memory_consumption=256
opcache.interned_strings_buffer=16
opcache.max_accelerated_files=10000
opcache.revalidate_freq=60
opcache.fast_shutdown=1
```

Restart PHP:

```bash
sudo systemctl restart php8.1-fpm
```

### WordPress Optimization

**Add to wp-config.php:**

```php
// Increase memory
define('WP_MEMORY_LIMIT', '512M');
define('WP_MAX_MEMORY_LIMIT', '512M');

// Disable revisions (saves DB space)
define('WP_POST_REVISIONS', 3);

// Increase autosave interval
define('AUTOSAVE_INTERVAL', 300);

// Disable post cron for large sites
define('DISABLE_WP_CRON', true);
```

### Caching Setup

**Install Redis (Optional):**

```bash
# Install Redis
sudo apt-get install redis-server php-redis

# Configure Redis
sudo nano /etc/redis/redis.conf
```

Set:

```
maxmemory 256mb
maxmemory-policy allkeys-lru
```

Restart:

```bash
sudo systemctl restart redis-server
```

**WordPress Redis Plugin:**

1. Install "Redis Object Cache" plugin
2. Add to wp-config.php:

```php
define('WP_REDIS_HOST', '127.0.0.1');
define('WP_REDIS_PORT', 6379);
define('WP_REDIS_TIMEOUT', 1);
define('WP_REDIS_READ_TIMEOUT', 1);
define('WP_REDIS_DATABASE', 0);
```

### Disk Space Management

**Automatic cleanup script:**

```bash
sudo nano /usr/local/bin/cleanup-ytdlp.sh
```

Add:

```bash
#!/bin/bash
# Clean up old downloads

DOWNLOAD_DIR="/var/www/html/wp-content/uploads/yt-dlp-downloads/temp"
MAX_AGE=1 # Hours

# Delete files older than MAX_AGE
find "$DOWNLOAD_DIR" -type f -mmin +$((MAX_AGE * 60)) -delete

# Log
echo "Cleaned up downloads on $(date)" >> /var/log/ytdlp-cleanup.log
```

Make executable:

```bash
sudo chmod +x /usr/local/bin/cleanup-ytdlp.sh
```

Add to crontab (runs every hour):

```bash
sudo crontab -e
```

Add:

```
0 * * * * /usr/local/bin/cleanup-ytdlp.sh
```

---

## Troubleshooting

### Common Errors and Solutions

#### Error: "yt-dlp not found"

**Diagnosis:**

```bash
# Check if installed
which yt-dlp
yt-dlp --version

# Check path
ls -l /usr/local/bin/yt-dlp
```

**Solutions:**

1. **Path incorrect in plugin settings**
   - Get correct path: `which yt-dlp`
   - Update in WordPress admin

2. **Not installed**
   - Follow installation steps above

3. **Wrong permissions**
   ```bash
   sudo chmod +x /usr/local/bin/yt-dlp
   ```

4. **PHP can't execute**
   ```bash
   # Test from PHP
   php -r "echo shell_exec('which yt-dlp');"
   ```

#### Error: "shell_exec is disabled"

**Diagnosis:**

```bash
php -r "echo ini_get('disable_functions');"
```

**Solutions:**

1. **Edit php.ini:**
   ```bash
   sudo nano /etc/php/8.1/fpm/php.ini
   ```
   
   Find `disable_functions` and remove `shell_exec`, `exec`, `proc_open`

2. **Restart PHP:**
   ```bash
   sudo systemctl restart php8.1-fpm
   ```

3. **Contact host (shared hosting):**
   - If on shared hosting and functions are disabled, ask host to enable
   - May need to upgrade to VPS

#### Error: "Download failed" or "ERROR 403"

**Common causes:**

1. **YouTube bot detection**
   
   **Solution: Use cookies file**
   
   ```bash
   # Export cookies from browser using extension
   # Examples: "Get cookies.txt" (Chrome), "cookies.txt" (Firefox)
   
   # Upload to server
   scp cookies.txt user@server:/path/to/cookies.txt
   
   # In WordPress plugin settings:
   Cookies File: /path/to/cookies.txt
   ```

2. **Outdated yt-dlp**
   
   **Solution: Update**
   
   ```bash
   sudo wget https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp -O /usr/local/bin/yt-dlp
   sudo chmod +x /usr/local/bin/yt-dlp
   ```

3. **Network issues**
   
   **Solution: Test connection**
   
   ```bash
   yt-dlp --verbose "VIDEO_URL" 2>&1 | grep -i error
   ```

#### Error: "Permission denied"

**Solution: Fix permissions**

```bash
# WordPress uploads directory
sudo chown -R www-data:www-data /var/www/html/wp-content/uploads/
sudo chmod -R 755 /var/www/html/wp-content/uploads/

# yt-dlp downloads directory
sudo mkdir -p /var/www/html/wp-content/uploads/yt-dlp-downloads/temp
sudo chown -R www-data:www-data /var/www/html/wp-content/uploads/yt-dlp-downloads/
sudo chmod -R 755 /var/www/html/wp-content/uploads/yt-dlp-downloads/
```

#### Error: "Timeout" or "Max execution time exceeded"

**Solutions:**

1. **Increase PHP timeout:**
   ```ini
   max_execution_time = 600
   ```

2. **Increase plugin timeout:**
   - WordPress Admin → YT-DLP WP → Settings
   - Download Timeout: 600

3. **Increase server timeout:**
   
   **Nginx:**
   ```nginx
   fastcgi_read_timeout 600s;
   ```
   
   **Apache:**
   ```apache
   TimeOut 600
   ```

4. **Use smaller files:**
   - Lower quality
   - Shorter videos
   - Audio only

#### Error: "FFmpeg not found"

**Diagnosis:**

```bash
which ffmpeg
ffmpeg -version
```

**Solutions:**

1. **Install FFmpeg:**
   ```bash
   sudo apt-get install ffmpeg
   ```

2. **Update path:**
   - Get path: `which ffmpeg`
   - Update in plugin settings

3. **Test from PHP:**
   ```bash
   php -r "echo shell_exec('ffmpeg -version');"
   ```

#### Error: "Format not available"

**Solutions:**

1. **Try different format:**
   - MP4 instead of MKV
   - MP3 instead of FLAC

2. **Use "Best Quality" option:**
   - Let yt-dlp choose best available

3. **Check video source:**
   - Some platforms have format restrictions
   - Try different video

#### Error: "File too large"

**Solutions:**

1. **Increase limit in plugin:**
   - YT-DLP WP → Settings
   - Max File Size: increase value

2. **Free up disk space:**
   ```bash
   df -h
   # If low, clean up old files
   ```

3. **Use lower quality:**
   - 720p instead of 1080p
   - Compress audio

### Debug Mode

**Enable WordPress debug:**

```php
// In wp-config.php
define('WP_DEBUG', true);
define('WP_DEBUG_LOG', true);
define('WP_DEBUG_DISPLAY', false);
@ini_set('display_errors', 0);
```

**Enable plugin logging:**

1. YT-DLP WP → Settings
2. Check "Enable Logging"
3. Save

**View logs:**

```bash
# WordPress debug log
tail -f /var/www/html/wp-content/debug.log

# PHP error log
tail -f /var/log/php_errors.log

# Nginx error log
tail -f /var/log/nginx/error.log

# Apache error log
tail -f /var/log/apache2/error.log
```

### Test Commands

**Test yt-dlp directly:**

```bash
# Basic test
yt-dlp --version

# Test with video
yt-dlp --skip-download --print-json "https://www.youtube.com/watch?v=dQw4w9WgXcQ"

# Test with cookies
yt-dlp --cookies /path/to/cookies.txt --skip-download --print-json "VIDEO_URL"
```

**Test from PHP:**

```php
<?php
// Save as test.php in WordPress root

// Test yt-dlp
$ytdlp_path = '/usr/local/bin/yt-dlp';
$cmd = escapeshellarg($ytdlp_path) . ' --version 2>&1';
$output = shell_exec($cmd);
echo "yt-dlp test:\n" . $output . "\n\n";

// Test FFmpeg
$ffmpeg_path = '/usr/bin/ffmpeg';
$cmd = escapeshellarg($ffmpeg_path) . ' -version 2>&1';
$output = shell_exec($cmd);
echo "FFmpeg test:\n" . $output . "\n\n";

// Test with real download
$url = 'https://www.youtube.com/watch?v=dQw4w9WgXcQ';
$cmd = escapeshellarg($ytdlp_path) . ' --skip-download --print-json ' . escapeshellarg($url) . ' 2>&1';
$output = shell_exec($cmd);
echo "Download test:\n" . $output . "\n";
```

Access via browser, then delete.

---

## Maintenance

### Regular Updates

**yt-dlp (Critical - update regularly):**

```bash
# Manual update
sudo wget https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp -O /usr/local/bin/yt-dlp
sudo chmod +x /usr/local/bin/yt-dlp

# Or via pip
sudo pip3 install --upgrade yt-dlp
```

**FFmpeg (update occasionally):**

```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get upgrade ffmpeg

# Check version
ffmpeg -version
```

**WordPress and Plugin:**

- Keep WordPress core updated
- Update plugin when new version available
- Test updates on staging site first

### Monitoring

**Disk space monitoring:**

```bash
# Check space
df -h

# Check temp downloads folder
du -sh /var/www/html/wp-content/uploads/yt-dlp-downloads/temp/

# Set up alert (example)
THRESHOLD=80
CURRENT=$(df -h / | grep / | awk '{print $5}' | sed 's/%//g')
if [ "$CURRENT" -gt "$THRESHOLD" ]; then
    echo "Disk usage is above $THRESHOLD%" | mail -s "Disk Alert" admin@example.com
fi
```

**Monitor logs:**

```bash
# Watch for errors
tail -f /var/log/nginx/error.log | grep ytdlp

# Count errors per day
grep -c "ytdlp" /var/log/nginx/error.log
```

### Cleanup Strategies

**Manual cleanup:**

```bash
# Remove old downloads
find /var/www/html/wp-content/uploads/yt-dlp-downloads/temp/ -type f -mtime +1 -delete

# Remove empty directories
find /var/www/html/wp-content/uploads/yt-dlp-downloads/temp/ -type d -empty -delete

# Check cache
rm -rf /tmp/yt-dlp-cache/*
```

**Automated cleanup (cron):**

```bash
sudo crontab -e
```

Add:

```
# Clean downloads every hour
0 * * * * find /var/www/html/wp-content/uploads/yt-dlp-downloads/temp/ -type f -mmin +60 -delete

# Clean cache daily at 3 AM
0 3 * * * rm -rf /tmp/yt-dlp-cache/*

# Check disk space daily
0 6 * * * df -h | mail -s "Daily Disk Report" admin@example.com
```

### Backup Important Files

**What to backup:**

```bash
# Plugin settings
wp-content/uploads/yt-dlp-downloads/.htaccess

# Cookies file (if using)
/path/to/cookies.txt

# WordPress config
wp-config.php

# Database (includes plugin settings)
# Use WordPress backup plugin or:
mysqldump -u username -p database_name > backup.sql
```

### Health Check Script

Create: `/usr/local/bin/ytdlp-health-check.sh`

```bash
#!/bin/bash

echo "=== YT-DLP Health Check $(date) ===" | tee -a /var/log/ytdlp-health.log

# Check yt-dlp
if command -v yt-dlp &> /dev/null; then
    echo "✓ yt-dlp installed" | tee -a /var/log/ytdlp-health.log
    yt-dlp --version | tee -a /var/log/ytdlp-health.log
else
    echo "✗ yt-dlp NOT found" | tee -a /var/log/ytdlp-health.log
fi

# Check FFmpeg
if command -v ffmpeg &> /dev/null; then
    echo "✓ FFmpeg installed" | tee -a /var/log/ytdlp-health.log
else
    echo "✗ FFmpeg NOT found" | tee -a /var/log/ytdlp-health.log
fi

# Check disk space
DISK_USAGE=$(df -h / | grep / | awk '{print $5}' | sed 's/%//g')
echo "Disk usage: $DISK_USAGE%" | tee -a /var/log/ytdlp-health.log
if [ "$DISK_USAGE" -gt 80 ]; then
    echo "⚠ WARNING: Disk usage above 80%" | tee -a /var/log/ytdlp-health.log
fi

# Check temp folder size
TEMP_SIZE=$(du -sh /var/www/html/wp-content/uploads/yt-dlp-downloads/temp/ 2>/dev/null | awk '{print $1}')
echo "Temp folder size: $TEMP_SIZE" | tee -a /var/log/ytdlp-health.log

# Check permissions
if [ -w /var/www/html/wp-content/uploads/ ]; then
    echo "✓ Upload directory writable" | tee -a /var/log/ytdlp-health.log
else
    echo "✗ Upload directory NOT writable" | tee -a /var/log/ytdlp-health.log
fi

echo "===================================" | tee -a /var/log/ytdlp-health.log
```

Make executable:

```bash
sudo chmod +x /usr/local/bin/ytdlp-health-check.sh
```

Run daily:

```bash
sudo crontab -e
```

Add:

```
0 9 * * * /usr/local/bin/ytdlp-health-check.sh
```

---

## Quick Reference: Platform-Specific Paths

### Ubuntu/Debian VPS

```
yt-dlp path: /usr/local/bin/yt-dlp
FFmpeg path: /usr/bin/ffmpeg
PHP config: /etc/php/8.1/fpm/php.ini
WordPress root: /var/www/html
Web user: www-data
```

### CentOS/RHEL VPS

```
yt-dlp path: /usr/local/bin/yt-dlp
FFmpeg path: /usr/bin/ffmpeg
PHP config: /etc/php.ini
WordPress root: /var/www/html
Web user: nginx or apache
```

### cPanel

```
yt-dlp path: /home/username/bin/yt-dlp
FFmpeg path: /home/username/bin/ffmpeg
PHP config: ~/public_html/php.ini or MultiPHP Editor
WordPress root: ~/public_html
Web user: username
```

### Windows Server

```
yt-dlp path: C:\Program Files\yt-dlp\yt-dlp.exe
FFmpeg path: C:\Program Files\ffmpeg\bin\ffmpeg.exe
PHP config: C:\Program Files\PHP\php.ini
WordPress root: C:\inetpub\wwwroot
Web user: IIS_IUSRS
```

---

## Support Resources

### Official Documentation

- **yt-dlp**: https://github.com/yt-dlp/yt-dlp
- **FFmpeg**: https://ffmpeg.org/documentation.html
- **WordPress**: https://wordpress.org/support/
- **PHP**: https://www.php.net/manual/

### Community Help

- **yt-dlp Issues**: https://github.com/yt-dlp/yt-dlp/issues
- **WordPress Forums**: https://wordpress.org/support/forums/
- **Stack Overflow**: Tag: wordpress, yt-dlp, ffmpeg

### Hosting Provider Support

Always check with your hosting provider for:
- Specific server configurations
- PHP function restrictions
- Shell access policies
- Resource limitations
- Installation assistance

---

## Checklist: Post-Installation Verification

Use this checklist after completing setup:

- [ ] yt-dlp installed and accessible
- [ ] yt-dlp version command works: `yt-dlp --version`
- [ ] FFmpeg installed and accessible
- [ ] FFmpeg version command works: `ffmpeg -version`
- [ ] PHP version is 7.4 or higher
- [ ] PHP shell_exec is NOT disabled
- [ ] PHP memory_limit is at least 512M
- [ ] PHP max_execution_time is at least 300
- [ ] PHP upload_max_filesize is at least 100M
- [ ] WordPress uploads directory is writable
- [ ] Plugin is activated in WordPress
- [ ] Plugin settings are configured with correct paths
- [ ] Test download works with a short video
- [ ] No errors in WordPress debug.log
- [ ] Automatic cleanup is configured
- [ ] Monitoring/logging is enabled
- [ ] Backup strategy is in place

---

## Conclusion

This guide covered setup for various hosting environments. Key takeaways:

1. **VPS hosting is recommended** for best performance and control
2. **Shared hosting has limitations** - test carefully and set conservative limits
3. **Keep yt-dlp updated** - YouTube changes frequently
4. **Monitor disk space** - temporary files can accumulate
5. **Enable logging** - helps troubleshoot issues quickly
6. **Test thoroughly** - start with short videos, increase gradually

For additional help, consult the official documentation links provided throughout this guide.

---

**Document Version**: 1.0  
**Last Updated**: November 2024  
**Compatible with**: YT-DLP WordPress Plugin v1.0.0
