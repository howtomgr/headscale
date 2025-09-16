# headscale Installation Guide

headscale is a free and open-source open source Tailscale control server. Headscale provides self-hosted implementation of the Tailscale control server, enabling fully self-hosted mesh VPN

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
  - RAM: 256MB minimum
  - Storage: 100MB for installation
  - Network: HTTPS for clients
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8080 (default headscale port)
  - gRPC on 50443
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

# Install headscale
sudo dnf install -y headscale

# Enable and start service
sudo systemctl enable --now headscale

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
headscale version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install headscale
sudo apt install -y headscale

# Enable and start service
sudo systemctl enable --now headscale

# Configure firewall
sudo ufw allow 8080

# Verify installation
headscale version
```

### Arch Linux

```bash
# Install headscale
sudo pacman -S headscale

# Enable and start service
sudo systemctl enable --now headscale

# Verify installation
headscale version
```

### Alpine Linux

```bash
# Install headscale
apk add --no-cache headscale

# Enable and start service
rc-update add headscale default
rc-service headscale start

# Verify installation
headscale version
```

### openSUSE/SLES

```bash
# Install headscale
sudo zypper install -y headscale

# Enable and start service
sudo systemctl enable --now headscale

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
headscale version
```

### macOS

```bash
# Using Homebrew
brew install headscale

# Start service
brew services start headscale

# Verify installation
headscale version
```

### FreeBSD

```bash
# Using pkg
pkg install headscale

# Enable in rc.conf
echo 'headscale_enable="YES"' >> /etc/rc.conf

# Start service
service headscale start

# Verify installation
headscale version
```

### Windows

```bash
# Using Chocolatey
choco install headscale

# Or using Scoop
scoop install headscale

# Verify installation
headscale version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/headscale

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
headscale version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable headscale

# Start service
sudo systemctl start headscale

# Stop service
sudo systemctl stop headscale

# Restart service
sudo systemctl restart headscale

# Check status
sudo systemctl status headscale

# View logs
sudo journalctl -u headscale -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add headscale default

# Start service
rc-service headscale start

# Stop service
rc-service headscale stop

# Restart service
rc-service headscale restart

# Check status
rc-service headscale status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'headscale_enable="YES"' >> /etc/rc.conf

# Start service
service headscale start

# Stop service
service headscale stop

# Restart service
service headscale restart

# Check status
service headscale status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start headscale
brew services stop headscale
brew services restart headscale

# Check status
brew services list | grep headscale
```

### Windows Service Manager

```powershell
# Start service
net start headscale

# Stop service
net stop headscale

# Using PowerShell
Start-Service headscale
Stop-Service headscale
Restart-Service headscale

# Check status
Get-Service headscale
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream headscale_backend {
    server 127.0.0.1:8080;
}

server {
    listen 80;
    server_name headscale.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name headscale.example.com;

    ssl_certificate /etc/ssl/certs/headscale.example.com.crt;
    ssl_certificate_key /etc/ssl/private/headscale.example.com.key;

    location / {
        proxy_pass http://headscale_backend;
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
    ServerName headscale.example.com
    Redirect permanent / https://headscale.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName headscale.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/headscale.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/headscale.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend headscale_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/headscale.pem
    redirect scheme https if !{ ssl_fc }
    default_backend headscale_backend

backend headscale_backend
    balance roundrobin
    server headscale1 127.0.0.1:8080 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R headscale:headscale /etc/headscale
sudo chmod 750 /etc/headscale

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
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
sudo systemctl status headscale

# View logs
sudo journalctl -u headscale -f

# Monitor resource usage
top -p $(pgrep headscale)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/headscale"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/headscale-backup-$DATE.tar.gz" /etc/headscale /var/lib/headscale

echo "Backup completed: $BACKUP_DIR/headscale-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop headscale

# Restore from backup
tar -xzf /backup/headscale/headscale-backup-*.tar.gz -C /

# Start service
sudo systemctl start headscale
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u headscale -n 100
sudo tail -f /var/log/headscale/headscale.log

# Check configuration
headscale version

# Check permissions
ls -la /etc/headscale
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8080

# Test connectivity
telnet localhost 8080

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep headscale)

# Check disk I/O
iotop -p $(pgrep headscale)

# Check connections
ss -an | grep 8080
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  headscale:
    image: headscale:latest
    ports:
      - "8080:8080"
    volumes:
      - ./config:/etc/headscale
      - ./data:/var/lib/headscale
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update headscale

# Debian/Ubuntu
sudo apt update && sudo apt upgrade headscale

# Arch Linux
sudo pacman -Syu headscale

# Alpine Linux
apk update && apk upgrade headscale

# openSUSE
sudo zypper update headscale

# FreeBSD
pkg update && pkg upgrade headscale

# Always backup before updates
tar -czf /backup/headscale-pre-update-$(date +%Y%m%d).tar.gz /etc/headscale

# Restart after updates
sudo systemctl restart headscale
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/headscale

# Clean old logs
find /var/log/headscale -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/headscale
```

## Additional Resources

- Official Documentation: https://docs.headscale.org/
- GitHub Repository: https://github.com/headscale/headscale
- Community Forum: https://forum.headscale.org/
- Best Practices Guide: https://docs.headscale.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
