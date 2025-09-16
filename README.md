# lxd Installation Guide

lxd is a free and open-source next generation system container manager. LXD offers a user experience similar to virtual machines but using Linux containers, providing an alternative to traditional VMs

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
  - RAM: 512MB minimum (2GB+ recommended)
  - Storage: 3GB for installation
  - Network: REST API over HTTPS
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8443 (default lxd port)
  - Clustering ports if used
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

# Install lxd
sudo dnf install -y lxd

# Enable and start service
sudo systemctl enable --now lxd

# Configure firewall
sudo firewall-cmd --permanent --add-port=8443/tcp
sudo firewall-cmd --reload

# Verify installation
lxd --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install lxd
sudo apt install -y lxd

# Enable and start service
sudo systemctl enable --now lxd

# Configure firewall
sudo ufw allow 8443

# Verify installation
lxd --version
```

### Arch Linux

```bash
# Install lxd
sudo pacman -S lxd

# Enable and start service
sudo systemctl enable --now lxd

# Verify installation
lxd --version
```

### Alpine Linux

```bash
# Install lxd
apk add --no-cache lxd

# Enable and start service
rc-update add lxd default
rc-service lxd start

# Verify installation
lxd --version
```

### openSUSE/SLES

```bash
# Install lxd
sudo zypper install -y lxd

# Enable and start service
sudo systemctl enable --now lxd

# Configure firewall
sudo firewall-cmd --permanent --add-port=8443/tcp
sudo firewall-cmd --reload

# Verify installation
lxd --version
```

### macOS

```bash
# Using Homebrew
brew install lxd

# Start service
brew services start lxd

# Verify installation
lxd --version
```

### FreeBSD

```bash
# Using pkg
pkg install lxd

# Enable in rc.conf
echo 'lxd_enable="YES"' >> /etc/rc.conf

# Start service
service lxd start

# Verify installation
lxd --version
```

### Windows

```bash
# Using Chocolatey
choco install lxd

# Or using Scoop
scoop install lxd

# Verify installation
lxd --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/lxd

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
lxd --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable lxd

# Start service
sudo systemctl start lxd

# Stop service
sudo systemctl stop lxd

# Restart service
sudo systemctl restart lxd

# Check status
sudo systemctl status lxd

# View logs
sudo journalctl -u lxd -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add lxd default

# Start service
rc-service lxd start

# Stop service
rc-service lxd stop

# Restart service
rc-service lxd restart

# Check status
rc-service lxd status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'lxd_enable="YES"' >> /etc/rc.conf

# Start service
service lxd start

# Stop service
service lxd stop

# Restart service
service lxd restart

# Check status
service lxd status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start lxd
brew services stop lxd
brew services restart lxd

# Check status
brew services list | grep lxd
```

### Windows Service Manager

```powershell
# Start service
net start lxd

# Stop service
net stop lxd

# Using PowerShell
Start-Service lxd
Stop-Service lxd
Restart-Service lxd

# Check status
Get-Service lxd
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream lxd_backend {
    server 127.0.0.1:8443;
}

server {
    listen 80;
    server_name lxd.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name lxd.example.com;

    ssl_certificate /etc/ssl/certs/lxd.example.com.crt;
    ssl_certificate_key /etc/ssl/private/lxd.example.com.key;

    location / {
        proxy_pass http://lxd_backend;
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
    ServerName lxd.example.com
    Redirect permanent / https://lxd.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName lxd.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/lxd.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/lxd.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8443/
    ProxyPassReverse / http://127.0.0.1:8443/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend lxd_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/lxd.pem
    redirect scheme https if !{ ssl_fc }
    default_backend lxd_backend

backend lxd_backend
    balance roundrobin
    server lxd1 127.0.0.1:8443 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R lxd:lxd /etc/lxd
sudo chmod 750 /etc/lxd

# Configure firewall
sudo firewall-cmd --permanent --add-port=8443/tcp
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
sudo systemctl status lxd

# View logs
sudo journalctl -u lxd -f

# Monitor resource usage
top -p $(pgrep lxd)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/lxd"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/lxd-backup-$DATE.tar.gz" /etc/lxd /var/lib/lxd

echo "Backup completed: $BACKUP_DIR/lxd-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop lxd

# Restore from backup
tar -xzf /backup/lxd/lxd-backup-*.tar.gz -C /

# Start service
sudo systemctl start lxd
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u lxd -n 100
sudo tail -f /var/log/lxd/lxd.log

# Check configuration
lxd --version

# Check permissions
ls -la /etc/lxd
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8443

# Test connectivity
telnet localhost 8443

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep lxd)

# Check disk I/O
iotop -p $(pgrep lxd)

# Check connections
ss -an | grep 8443
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  lxd:
    image: lxd:latest
    ports:
      - "8443:8443"
    volumes:
      - ./config:/etc/lxd
      - ./data:/var/lib/lxd
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update lxd

# Debian/Ubuntu
sudo apt update && sudo apt upgrade lxd

# Arch Linux
sudo pacman -Syu lxd

# Alpine Linux
apk update && apk upgrade lxd

# openSUSE
sudo zypper update lxd

# FreeBSD
pkg update && pkg upgrade lxd

# Always backup before updates
tar -czf /backup/lxd-pre-update-$(date +%Y%m%d).tar.gz /etc/lxd

# Restart after updates
sudo systemctl restart lxd
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/lxd

# Clean old logs
find /var/log/lxd -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/lxd
```

## Additional Resources

- Official Documentation: https://docs.lxd.org/
- GitHub Repository: https://github.com/lxd/lxd
- Community Forum: https://forum.lxd.org/
- Best Practices Guide: https://docs.lxd.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
