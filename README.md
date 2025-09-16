# envoy Installation Guide

envoy is a free and open-source edge and service proxy. Envoy provides high-performance edge/service proxy

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
  - RAM: 1GB minimum
  - Storage: 1GB for config
  - Network: HTTP/gRPC
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 10000 (default envoy port)
  - Admin on 9901
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

# Install envoy
sudo dnf install -y envoy

# Enable and start service
sudo systemctl enable --now envoy

# Configure firewall
sudo firewall-cmd --permanent --add-port=10000/tcp
sudo firewall-cmd --reload

# Verify installation
envoy --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install envoy
sudo apt install -y envoy

# Enable and start service
sudo systemctl enable --now envoy

# Configure firewall
sudo ufw allow 10000

# Verify installation
envoy --version
```

### Arch Linux

```bash
# Install envoy
sudo pacman -S envoy

# Enable and start service
sudo systemctl enable --now envoy

# Verify installation
envoy --version
```

### Alpine Linux

```bash
# Install envoy
apk add --no-cache envoy

# Enable and start service
rc-update add envoy default
rc-service envoy start

# Verify installation
envoy --version
```

### openSUSE/SLES

```bash
# Install envoy
sudo zypper install -y envoy

# Enable and start service
sudo systemctl enable --now envoy

# Configure firewall
sudo firewall-cmd --permanent --add-port=10000/tcp
sudo firewall-cmd --reload

# Verify installation
envoy --version
```

### macOS

```bash
# Using Homebrew
brew install envoy

# Start service
brew services start envoy

# Verify installation
envoy --version
```

### FreeBSD

```bash
# Using pkg
pkg install envoy

# Enable in rc.conf
echo 'envoy_enable="YES"' >> /etc/rc.conf

# Start service
service envoy start

# Verify installation
envoy --version
```

### Windows

```bash
# Using Chocolatey
choco install envoy

# Or using Scoop
scoop install envoy

# Verify installation
envoy --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/envoy

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
envoy --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable envoy

# Start service
sudo systemctl start envoy

# Stop service
sudo systemctl stop envoy

# Restart service
sudo systemctl restart envoy

# Check status
sudo systemctl status envoy

# View logs
sudo journalctl -u envoy -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add envoy default

# Start service
rc-service envoy start

# Stop service
rc-service envoy stop

# Restart service
rc-service envoy restart

# Check status
rc-service envoy status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'envoy_enable="YES"' >> /etc/rc.conf

# Start service
service envoy start

# Stop service
service envoy stop

# Restart service
service envoy restart

# Check status
service envoy status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start envoy
brew services stop envoy
brew services restart envoy

# Check status
brew services list | grep envoy
```

### Windows Service Manager

```powershell
# Start service
net start envoy

# Stop service
net stop envoy

# Using PowerShell
Start-Service envoy
Stop-Service envoy
Restart-Service envoy

# Check status
Get-Service envoy
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream envoy_backend {
    server 127.0.0.1:10000;
}

server {
    listen 80;
    server_name envoy.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name envoy.example.com;

    ssl_certificate /etc/ssl/certs/envoy.example.com.crt;
    ssl_certificate_key /etc/ssl/private/envoy.example.com.key;

    location / {
        proxy_pass http://envoy_backend;
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
    ServerName envoy.example.com
    Redirect permanent / https://envoy.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName envoy.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/envoy.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/envoy.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:10000/
    ProxyPassReverse / http://127.0.0.1:10000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend envoy_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/envoy.pem
    redirect scheme https if !{ ssl_fc }
    default_backend envoy_backend

backend envoy_backend
    balance roundrobin
    server envoy1 127.0.0.1:10000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R envoy:envoy /etc/envoy
sudo chmod 750 /etc/envoy

# Configure firewall
sudo firewall-cmd --permanent --add-port=10000/tcp
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
sudo systemctl status envoy

# View logs
sudo journalctl -u envoy -f

# Monitor resource usage
top -p $(pgrep envoy)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/envoy"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/envoy-backup-$DATE.tar.gz" /etc/envoy /var/lib/envoy

echo "Backup completed: $BACKUP_DIR/envoy-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop envoy

# Restore from backup
tar -xzf /backup/envoy/envoy-backup-*.tar.gz -C /

# Start service
sudo systemctl start envoy
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u envoy -n 100
sudo tail -f /var/log/envoy/envoy.log

# Check configuration
envoy --version

# Check permissions
ls -la /etc/envoy
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 10000

# Test connectivity
telnet localhost 10000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep envoy)

# Check disk I/O
iotop -p $(pgrep envoy)

# Check connections
ss -an | grep 10000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  envoy:
    image: envoy:latest
    ports:
      - "10000:10000"
    volumes:
      - ./config:/etc/envoy
      - ./data:/var/lib/envoy
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update envoy

# Debian/Ubuntu
sudo apt update && sudo apt upgrade envoy

# Arch Linux
sudo pacman -Syu envoy

# Alpine Linux
apk update && apk upgrade envoy

# openSUSE
sudo zypper update envoy

# FreeBSD
pkg update && pkg upgrade envoy

# Always backup before updates
tar -czf /backup/envoy-pre-update-$(date +%Y%m%d).tar.gz /etc/envoy

# Restart after updates
sudo systemctl restart envoy
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/envoy

# Clean old logs
find /var/log/envoy -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/envoy
```

## Additional Resources

- Official Documentation: https://docs.envoy.org/
- GitHub Repository: https://github.com/envoy/envoy
- Community Forum: https://forum.envoy.org/
- Best Practices Guide: https://docs.envoy.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
