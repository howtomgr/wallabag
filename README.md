# wallabag Installation Guide

wallabag is a free and open-source self-hosted read-it-later app. Wallabag saves web pages for later reading with article extraction and tagging, serving as a privacy-focused alternative to Pocket or Instapaper

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 1 core minimum
  - RAM: 512MB minimum
  - Storage: 1GB for articles
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80/443 (default wallabag port)
  - None
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install wallabag
sudo dnf install -y wallabag

# Enable and start service
sudo systemctl enable --now wallabag

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/443/tcp
sudo firewall-cmd --reload

# Verify installation
wallabag --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install wallabag
sudo apt install -y wallabag

# Enable and start service
sudo systemctl enable --now wallabag

# Configure firewall
sudo ufw allow 80/443

# Verify installation
wallabag --version
```

### Arch Linux

```bash
# Install wallabag
sudo pacman -S wallabag

# Enable and start service
sudo systemctl enable --now wallabag

# Verify installation
wallabag --version
```

### Alpine Linux

```bash
# Install wallabag
apk add --no-cache wallabag

# Enable and start service
rc-update add wallabag default
rc-service wallabag start

# Verify installation
wallabag --version
```

### openSUSE/SLES

```bash
# Install wallabag
sudo zypper install -y wallabag

# Enable and start service
sudo systemctl enable --now wallabag

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/443/tcp
sudo firewall-cmd --reload

# Verify installation
wallabag --version
```

### macOS

```bash
# Using Homebrew
brew install wallabag

# Start service
brew services start wallabag

# Verify installation
wallabag --version
```

### FreeBSD

```bash
# Using pkg
pkg install wallabag

# Enable in rc.conf
echo 'wallabag_enable="YES"' >> /etc/rc.conf

# Start service
service wallabag start

# Verify installation
wallabag --version
```

### Windows

```bash
# Using Chocolatey
choco install wallabag

# Or using Scoop
scoop install wallabag

# Verify installation
wallabag --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/wallabag

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
wallabag --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable wallabag

# Start service
sudo systemctl start wallabag

# Stop service
sudo systemctl stop wallabag

# Restart service
sudo systemctl restart wallabag

# Check status
sudo systemctl status wallabag

# View logs
sudo journalctl -u wallabag -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add wallabag default

# Start service
rc-service wallabag start

# Stop service
rc-service wallabag stop

# Restart service
rc-service wallabag restart

# Check status
rc-service wallabag status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'wallabag_enable="YES"' >> /etc/rc.conf

# Start service
service wallabag start

# Stop service
service wallabag stop

# Restart service
service wallabag restart

# Check status
service wallabag status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start wallabag
brew services stop wallabag
brew services restart wallabag

# Check status
brew services list | grep wallabag
```

### Windows Service Manager

```powershell
# Start service
net start wallabag

# Stop service
net stop wallabag

# Using PowerShell
Start-Service wallabag
Stop-Service wallabag
Restart-Service wallabag

# Check status
Get-Service wallabag
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream wallabag_backend {
    server 127.0.0.1:80/443;
}

server {
    listen 80;
    server_name wallabag.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name wallabag.example.com;

    ssl_certificate /etc/ssl/certs/wallabag.example.com.crt;
    ssl_certificate_key /etc/ssl/private/wallabag.example.com.key;

    location / {
        proxy_pass http://wallabag_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName wallabag.example.com
    Redirect permanent / https://wallabag.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName wallabag.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/wallabag.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/wallabag.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/443/
    ProxyPassReverse / http://127.0.0.1:80/443/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend wallabag_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/wallabag.pem
    redirect scheme https if !{ ssl_fc }
    default_backend wallabag_backend

backend wallabag_backend
    balance roundrobin
    server wallabag1 127.0.0.1:80/443 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R wallabag:wallabag /etc/wallabag
sudo chmod 750 /etc/wallabag

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/443/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status wallabag

# View logs
sudo journalctl -u wallabag -f

# Monitor resource usage
top -p $(pgrep wallabag)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/wallabag"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/wallabag-backup-$DATE.tar.gz" /etc/wallabag /var/lib/wallabag

echo "Backup completed: $BACKUP_DIR/wallabag-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop wallabag

# Restore from backup
tar -xzf /backup/wallabag/wallabag-backup-*.tar.gz -C /

# Start service
sudo systemctl start wallabag
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u wallabag -n 100
sudo tail -f /var/log/wallabag/wallabag.log

# Check configuration
wallabag --version

# Check permissions
ls -la /etc/wallabag
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80/443

# Test connectivity
telnet localhost 80/443

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep wallabag)

# Check disk I/O
iotop -p $(pgrep wallabag)

# Check connections
ss -an | grep 80/443
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  wallabag:
    image: wallabag:latest
    ports:
      - "80/443:80/443"
    volumes:
      - ./config:/etc/wallabag
      - ./data:/var/lib/wallabag
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update wallabag

# Debian/Ubuntu
sudo apt update && sudo apt upgrade wallabag

# Arch Linux
sudo pacman -Syu wallabag

# Alpine Linux
apk update && apk upgrade wallabag

# openSUSE
sudo zypper update wallabag

# FreeBSD
pkg update && pkg upgrade wallabag

# Always backup before updates
tar -czf /backup/wallabag-pre-update-$(date +%Y%m%d).tar.gz /etc/wallabag

# Restart after updates
sudo systemctl restart wallabag
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/wallabag

# Clean old logs
find /var/log/wallabag -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/wallabag
```

## Additional Resources

- Official Documentation: https://docs.wallabag.org/
- GitHub Repository: https://github.com/wallabag/wallabag
- Community Forum: https://forum.wallabag.org/
- Best Practices Guide: https://docs.wallabag.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
