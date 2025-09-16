# cacti Installation Guide

cacti is a free and open-source network graphing. Cacti provides complete RRDtool-based graphing solution

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
  - RAM: 1GB minimum
  - Storage: 5GB for RRDs
  - Network: SNMP/HTTP
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default cacti port)
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

# Install cacti
sudo dnf install -y cacti

# Enable and start service
sudo systemctl enable --now cacti

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
cacti --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install cacti
sudo apt install -y cacti

# Enable and start service
sudo systemctl enable --now cacti

# Configure firewall
sudo ufw allow 80

# Verify installation
cacti --version
```

### Arch Linux

```bash
# Install cacti
sudo pacman -S cacti

# Enable and start service
sudo systemctl enable --now cacti

# Verify installation
cacti --version
```

### Alpine Linux

```bash
# Install cacti
apk add --no-cache cacti

# Enable and start service
rc-update add cacti default
rc-service cacti start

# Verify installation
cacti --version
```

### openSUSE/SLES

```bash
# Install cacti
sudo zypper install -y cacti

# Enable and start service
sudo systemctl enable --now cacti

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
cacti --version
```

### macOS

```bash
# Using Homebrew
brew install cacti

# Start service
brew services start cacti

# Verify installation
cacti --version
```

### FreeBSD

```bash
# Using pkg
pkg install cacti

# Enable in rc.conf
echo 'cacti_enable="YES"' >> /etc/rc.conf

# Start service
service cacti start

# Verify installation
cacti --version
```

### Windows

```bash
# Using Chocolatey
choco install cacti

# Or using Scoop
scoop install cacti

# Verify installation
cacti --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/cacti

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
cacti --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable cacti

# Start service
sudo systemctl start cacti

# Stop service
sudo systemctl stop cacti

# Restart service
sudo systemctl restart cacti

# Check status
sudo systemctl status cacti

# View logs
sudo journalctl -u cacti -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add cacti default

# Start service
rc-service cacti start

# Stop service
rc-service cacti stop

# Restart service
rc-service cacti restart

# Check status
rc-service cacti status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'cacti_enable="YES"' >> /etc/rc.conf

# Start service
service cacti start

# Stop service
service cacti stop

# Restart service
service cacti restart

# Check status
service cacti status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start cacti
brew services stop cacti
brew services restart cacti

# Check status
brew services list | grep cacti
```

### Windows Service Manager

```powershell
# Start service
net start cacti

# Stop service
net stop cacti

# Using PowerShell
Start-Service cacti
Stop-Service cacti
Restart-Service cacti

# Check status
Get-Service cacti
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream cacti_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name cacti.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name cacti.example.com;

    ssl_certificate /etc/ssl/certs/cacti.example.com.crt;
    ssl_certificate_key /etc/ssl/private/cacti.example.com.key;

    location / {
        proxy_pass http://cacti_backend;
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
    ServerName cacti.example.com
    Redirect permanent / https://cacti.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName cacti.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/cacti.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/cacti.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend cacti_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/cacti.pem
    redirect scheme https if !{ ssl_fc }
    default_backend cacti_backend

backend cacti_backend
    balance roundrobin
    server cacti1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R cacti:cacti /etc/cacti
sudo chmod 750 /etc/cacti

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
sudo systemctl status cacti

# View logs
sudo journalctl -u cacti -f

# Monitor resource usage
top -p $(pgrep cacti)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/cacti"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/cacti-backup-$DATE.tar.gz" /etc/cacti /var/lib/cacti

echo "Backup completed: $BACKUP_DIR/cacti-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop cacti

# Restore from backup
tar -xzf /backup/cacti/cacti-backup-*.tar.gz -C /

# Start service
sudo systemctl start cacti
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u cacti -n 100
sudo tail -f /var/log/cacti/cacti.log

# Check configuration
cacti --version

# Check permissions
ls -la /etc/cacti
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
top -p $(pgrep cacti)

# Check disk I/O
iotop -p $(pgrep cacti)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  cacti:
    image: cacti:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/cacti
      - ./data:/var/lib/cacti
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update cacti

# Debian/Ubuntu
sudo apt update && sudo apt upgrade cacti

# Arch Linux
sudo pacman -Syu cacti

# Alpine Linux
apk update && apk upgrade cacti

# openSUSE
sudo zypper update cacti

# FreeBSD
pkg update && pkg upgrade cacti

# Always backup before updates
tar -czf /backup/cacti-pre-update-$(date +%Y%m%d).tar.gz /etc/cacti

# Restart after updates
sudo systemctl restart cacti
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/cacti

# Clean old logs
find /var/log/cacti -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/cacti
```

## Additional Resources

- Official Documentation: https://docs.cacti.org/
- GitHub Repository: https://github.com/cacti/cacti
- Community Forum: https://forum.cacti.org/
- Best Practices Guide: https://docs.cacti.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
