# smf Installation Guide

smf is a free and open-source Simple Machines Forum. SMF provides powerful, custom-made forum software

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
  - Storage: 1GB for data
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default smf port)
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

# Install smf
sudo dnf install -y smf

# Enable and start service
sudo systemctl enable --now smf

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
smf --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install smf
sudo apt install -y smf

# Enable and start service
sudo systemctl enable --now smf

# Configure firewall
sudo ufw allow 80

# Verify installation
smf --version
```

### Arch Linux

```bash
# Install smf
sudo pacman -S smf

# Enable and start service
sudo systemctl enable --now smf

# Verify installation
smf --version
```

### Alpine Linux

```bash
# Install smf
apk add --no-cache smf

# Enable and start service
rc-update add smf default
rc-service smf start

# Verify installation
smf --version
```

### openSUSE/SLES

```bash
# Install smf
sudo zypper install -y smf

# Enable and start service
sudo systemctl enable --now smf

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
smf --version
```

### macOS

```bash
# Using Homebrew
brew install smf

# Start service
brew services start smf

# Verify installation
smf --version
```

### FreeBSD

```bash
# Using pkg
pkg install smf

# Enable in rc.conf
echo 'smf_enable="YES"' >> /etc/rc.conf

# Start service
service smf start

# Verify installation
smf --version
```

### Windows

```bash
# Using Chocolatey
choco install smf

# Or using Scoop
scoop install smf

# Verify installation
smf --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/smf

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
smf --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable smf

# Start service
sudo systemctl start smf

# Stop service
sudo systemctl stop smf

# Restart service
sudo systemctl restart smf

# Check status
sudo systemctl status smf

# View logs
sudo journalctl -u smf -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add smf default

# Start service
rc-service smf start

# Stop service
rc-service smf stop

# Restart service
rc-service smf restart

# Check status
rc-service smf status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'smf_enable="YES"' >> /etc/rc.conf

# Start service
service smf start

# Stop service
service smf stop

# Restart service
service smf restart

# Check status
service smf status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start smf
brew services stop smf
brew services restart smf

# Check status
brew services list | grep smf
```

### Windows Service Manager

```powershell
# Start service
net start smf

# Stop service
net stop smf

# Using PowerShell
Start-Service smf
Stop-Service smf
Restart-Service smf

# Check status
Get-Service smf
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream smf_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name smf.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name smf.example.com;

    ssl_certificate /etc/ssl/certs/smf.example.com.crt;
    ssl_certificate_key /etc/ssl/private/smf.example.com.key;

    location / {
        proxy_pass http://smf_backend;
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
    ServerName smf.example.com
    Redirect permanent / https://smf.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName smf.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/smf.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/smf.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend smf_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/smf.pem
    redirect scheme https if !{ ssl_fc }
    default_backend smf_backend

backend smf_backend
    balance roundrobin
    server smf1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R smf:smf /etc/smf
sudo chmod 750 /etc/smf

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
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
sudo systemctl status smf

# View logs
sudo journalctl -u smf -f

# Monitor resource usage
top -p $(pgrep smf)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/smf"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/smf-backup-$DATE.tar.gz" /etc/smf /var/lib/smf

echo "Backup completed: $BACKUP_DIR/smf-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop smf

# Restore from backup
tar -xzf /backup/smf/smf-backup-*.tar.gz -C /

# Start service
sudo systemctl start smf
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u smf -n 100
sudo tail -f /var/log/smf/smf.log

# Check configuration
smf --version

# Check permissions
ls -la /etc/smf
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80

# Test connectivity
telnet localhost 80

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep smf)

# Check disk I/O
iotop -p $(pgrep smf)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  smf:
    image: smf:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/smf
      - ./data:/var/lib/smf
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update smf

# Debian/Ubuntu
sudo apt update && sudo apt upgrade smf

# Arch Linux
sudo pacman -Syu smf

# Alpine Linux
apk update && apk upgrade smf

# openSUSE
sudo zypper update smf

# FreeBSD
pkg update && pkg upgrade smf

# Always backup before updates
tar -czf /backup/smf-pre-update-$(date +%Y%m%d).tar.gz /etc/smf

# Restart after updates
sudo systemctl restart smf
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/smf

# Clean old logs
find /var/log/smf -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/smf
```

## Additional Resources

- Official Documentation: https://docs.smf.org/
- GitHub Repository: https://github.com/smf/smf
- Community Forum: https://forum.smf.org/
- Best Practices Guide: https://docs.smf.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
