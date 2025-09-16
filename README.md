# nifi Installation Guide

nifi is a free and open-source data flow automation. NiFi provides easy-to-use data flow automation and management

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
  - CPU: 2+ cores
  - RAM: 8GB minimum
  - Storage: 20GB for repos
  - Network: HTTP/HTTPS
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8080 (default nifi port)
  - Cluster on various
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

# Install nifi
sudo dnf install -y nifi

# Enable and start service
sudo systemctl enable --now nifi

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
nifi --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install nifi
sudo apt install -y nifi

# Enable and start service
sudo systemctl enable --now nifi

# Configure firewall
sudo ufw allow 8080

# Verify installation
nifi --version
```

### Arch Linux

```bash
# Install nifi
sudo pacman -S nifi

# Enable and start service
sudo systemctl enable --now nifi

# Verify installation
nifi --version
```

### Alpine Linux

```bash
# Install nifi
apk add --no-cache nifi

# Enable and start service
rc-update add nifi default
rc-service nifi start

# Verify installation
nifi --version
```

### openSUSE/SLES

```bash
# Install nifi
sudo zypper install -y nifi

# Enable and start service
sudo systemctl enable --now nifi

# Configure firewall
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Verify installation
nifi --version
```

### macOS

```bash
# Using Homebrew
brew install nifi

# Start service
brew services start nifi

# Verify installation
nifi --version
```

### FreeBSD

```bash
# Using pkg
pkg install nifi

# Enable in rc.conf
echo 'nifi_enable="YES"' >> /etc/rc.conf

# Start service
service nifi start

# Verify installation
nifi --version
```

### Windows

```bash
# Using Chocolatey
choco install nifi

# Or using Scoop
scoop install nifi

# Verify installation
nifi --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/nifi

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
nifi --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable nifi

# Start service
sudo systemctl start nifi

# Stop service
sudo systemctl stop nifi

# Restart service
sudo systemctl restart nifi

# Check status
sudo systemctl status nifi

# View logs
sudo journalctl -u nifi -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add nifi default

# Start service
rc-service nifi start

# Stop service
rc-service nifi stop

# Restart service
rc-service nifi restart

# Check status
rc-service nifi status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'nifi_enable="YES"' >> /etc/rc.conf

# Start service
service nifi start

# Stop service
service nifi stop

# Restart service
service nifi restart

# Check status
service nifi status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start nifi
brew services stop nifi
brew services restart nifi

# Check status
brew services list | grep nifi
```

### Windows Service Manager

```powershell
# Start service
net start nifi

# Stop service
net stop nifi

# Using PowerShell
Start-Service nifi
Stop-Service nifi
Restart-Service nifi

# Check status
Get-Service nifi
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream nifi_backend {
    server 127.0.0.1:8080;
}

server {
    listen 80;
    server_name nifi.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name nifi.example.com;

    ssl_certificate /etc/ssl/certs/nifi.example.com.crt;
    ssl_certificate_key /etc/ssl/private/nifi.example.com.key;

    location / {
        proxy_pass http://nifi_backend;
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
    ServerName nifi.example.com
    Redirect permanent / https://nifi.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName nifi.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/nifi.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/nifi.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend nifi_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/nifi.pem
    redirect scheme https if !{ ssl_fc }
    default_backend nifi_backend

backend nifi_backend
    balance roundrobin
    server nifi1 127.0.0.1:8080 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R nifi:nifi /etc/nifi
sudo chmod 750 /etc/nifi

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
sudo systemctl status nifi

# View logs
sudo journalctl -u nifi -f

# Monitor resource usage
top -p $(pgrep nifi)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/nifi"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/nifi-backup-$DATE.tar.gz" /etc/nifi /var/lib/nifi

echo "Backup completed: $BACKUP_DIR/nifi-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop nifi

# Restore from backup
tar -xzf /backup/nifi/nifi-backup-*.tar.gz -C /

# Start service
sudo systemctl start nifi
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u nifi -n 100
sudo tail -f /var/log/nifi/nifi.log

# Check configuration
nifi --version

# Check permissions
ls -la /etc/nifi
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
top -p $(pgrep nifi)

# Check disk I/O
iotop -p $(pgrep nifi)

# Check connections
ss -an | grep 8080
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  nifi:
    image: nifi:latest
    ports:
      - "8080:8080"
    volumes:
      - ./config:/etc/nifi
      - ./data:/var/lib/nifi
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update nifi

# Debian/Ubuntu
sudo apt update && sudo apt upgrade nifi

# Arch Linux
sudo pacman -Syu nifi

# Alpine Linux
apk update && apk upgrade nifi

# openSUSE
sudo zypper update nifi

# FreeBSD
pkg update && pkg upgrade nifi

# Always backup before updates
tar -czf /backup/nifi-pre-update-$(date +%Y%m%d).tar.gz /etc/nifi

# Restart after updates
sudo systemctl restart nifi
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/nifi

# Clean old logs
find /var/log/nifi -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/nifi
```

## Additional Resources

- Official Documentation: https://docs.nifi.org/
- GitHub Repository: https://github.com/nifi/nifi
- Community Forum: https://forum.nifi.org/
- Best Practices Guide: https://docs.nifi.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
