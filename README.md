# Poste.io Installation Guide

Poste.io is a free and open-source Mail Server Stack. Complete mail server with webmail and admin

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
  - CPU: 2 cores minimum (4+ cores recommended)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for installation
  - Network: 443 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 443 (default posteio port)
- **Dependencies**:
  - docker
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

# Install posteio
sudo dnf install -y posteio docker

# Enable and start service
sudo systemctl enable --now poste

# Configure firewall
sudo firewall-cmd --permanent --add-service=posteio
sudo firewall-cmd --reload

# Verify installation
posteio --version || systemctl status poste
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install posteio
sudo apt install -y posteio docker

# Enable and start service
sudo systemctl enable --now poste

# Configure firewall
sudo ufw allow 443

# Verify installation
posteio --version || systemctl status poste
```

### Arch Linux

```bash
# Install posteio
sudo pacman -S posteio

# Enable and start service
sudo systemctl enable --now poste

# Verify installation
posteio --version || systemctl status poste
```

### Alpine Linux

```bash
# Install posteio
apk add --no-cache posteio

# Enable and start service
rc-update add poste default
rc-service poste start

# Verify installation
posteio --version || rc-service poste status
```

### openSUSE/SLES

```bash
# Install posteio
sudo zypper install -y posteio docker

# Enable and start service
sudo systemctl enable --now poste

# Configure firewall
sudo firewall-cmd --permanent --add-service=posteio
sudo firewall-cmd --reload

# Verify installation
posteio --version || systemctl status poste
```

### macOS

```bash
# Using Homebrew
brew install posteio

# Start service
brew services start posteio

# Verify installation
posteio --version
```

### FreeBSD

```bash
# Using pkg
pkg install posteio

# Enable in rc.conf
echo 'poste_enable="YES"' >> /etc/rc.conf

# Start service
service poste start

# Verify installation
posteio --version || service poste status
```

### Windows

```powershell
# Using Chocolatey
choco install posteio

# Or using Scoop
scoop install posteio

# Verify installation
posteio --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /data

# Set up basic configuration
sudo tee /data/posteio.conf << 'EOF'
# Poste.io Configuration
max_connections=1000
EOF

# Test configuration
sudo posteio -t || sudo poste configtest

# Reload service
sudo systemctl reload poste
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R posteio:posteio /data
sudo chmod 750 /data

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable poste

# Start service
sudo systemctl start poste

# Stop service
sudo systemctl stop poste

# Restart service
sudo systemctl restart poste

# Reload configuration
sudo systemctl reload poste

# Check status
sudo systemctl status poste

# View logs
sudo journalctl -u poste -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add poste default

# Start service
rc-service poste start

# Stop service
rc-service poste stop

# Restart service
rc-service poste restart

# Check status
rc-service poste status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'poste_enable="YES"' >> /etc/rc.conf

# Start service
service poste start

# Stop service
service poste stop

# Restart service
service poste restart

# Check status
service poste status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start posteio
brew services stop posteio
brew services restart posteio

# Check status
brew services list | grep posteio
```

### Windows Service Manager

```powershell
# Start service
net start poste

# Stop service
net stop poste

# Using PowerShell
Start-Service poste
Stop-Service poste
Restart-Service poste

# Check status
Get-Service poste
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /data/posteio.conf << 'EOF'
max_connections=1000
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart poste
```

### Clustering and High Availability

```bash
# Configure clustering (if supported)
# See official documentation for cluster setup

# Basic load balancing setup example
# Configure multiple instances on different ports
```

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream posteio_backend {
    server 127.0.0.1:443;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name posteio.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name posteio.example.com;

    ssl_certificate /etc/ssl/certs/posteio.example.com.crt;
    ssl_certificate_key /etc/ssl/private/posteio.example.com.key;

    location / {
        proxy_pass http://posteio_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName posteio.example.com
    Redirect permanent / https://posteio.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName posteio.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/posteio.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/posteio.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:443/
    ProxyPassReverse / http://127.0.0.1:443/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:443/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend posteio_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/posteio.pem
    redirect scheme https if !{ ssl_fc }
    default_backend posteio_backend

backend posteio_backend
    balance roundrobin
    option httpchk GET /health
    server posteio1 127.0.0.1:443 check
    server posteio2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R posteio:posteio /data
sudo chmod 750 /data

# Configure firewall
sudo firewall-cmd --permanent --add-service=posteio
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/posteio.conf << 'EOF'
[posteio]
enabled = true
port = 443
filter = posteio
logpath = /var/log/poste/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/posteio.key \
    -out /etc/ssl/certs/posteio.crt

# Configure SSL in posteio
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE posteio_db;
CREATE USER posteio_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE posteio_db TO posteio_user;
EOF

# Configure posteio to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE posteio_db;
CREATE USER 'posteio_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON posteio_db.* TO 'posteio_user'@'localhost';
FLUSH PRIVILEGES;
EOF
```

## Performance Optimization

### System Tuning

```bash
# Kernel parameters
sudo tee -a /etc/sysctl.conf << EOF
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.core.netdev_max_backlog = 5000
vm.swappiness = 10
EOF

sudo sysctl -p

# Poste.io specific tuning
max_connections=1000
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
posteio soft nofile 65535
posteio hard nofile 65535
posteio soft nproc 32768
posteio hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'posteio'
    static_configs:
      - targets: ['localhost:443']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet poste; then
    echo "Poste.io is running"
    exit 0
else
    echo "Poste.io is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/posteio << 'EOF'
/var/log/poste/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 posteio posteio
    postrotate
        systemctl reload poste > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Poste.io backup script
BACKUP_DIR="/backup/posteio"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop poste

# Backup configuration
tar -czf "$BACKUP_DIR/posteio-config-$DATE.tar.gz" /data

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/posteio-data-$DATE.tar.gz" /var/lib/posteio

# Start service
systemctl start poste

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop poste

# Restore configuration
sudo tar -xzf /backup/posteio/posteio-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/posteio/posteio-data-*.tar.gz -C /

# Set permissions
sudo chown -R posteio:posteio /data
sudo chown -R posteio:posteio /var/lib/posteio

# Start service
sudo systemctl start poste
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u poste -n 100
sudo tail -f /var/log/poste/*.log

# Check configuration
sudo posteio -t || sudo poste configtest

# Check permissions
ls -la /data
ls -la /var/lib/posteio
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 443
sudo netstat -tlnp | grep 443

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 443
nc -zv localhost 443
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep poste)
htop -p $(pgrep poste)

# Check connections
ss -ant | grep :443 | wc -l

# Monitor I/O
iotop -p $(pgrep poste)
```

### Debug Mode

```bash
# Run in debug mode
sudo posteio -d
# or
sudo poste debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  posteio:
    image: posteio:latest
    container_name: posteio
    ports:
      - "443:443"
    volumes:
      - ./config:/data
      - ./data:/var/lib/posteio
    environment:
      - posteio_CONFIG=/data/posteio.conf
    restart: unless-stopped
    networks:
      - posteio_net

networks:
  posteio_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: posteio
spec:
  replicas: 3
  selector:
    matchLabels:
      app: posteio
  template:
    metadata:
      labels:
        app: posteio
    spec:
      containers:
      - name: posteio
        image: posteio:latest
        ports:
        - containerPort: 443
        volumeMounts:
        - name: config
          mountPath: /data
      volumes:
      - name: config
        configMap:
          name: posteio-config
---
apiVersion: v1
kind: Service
metadata:
  name: posteio
spec:
  selector:
    app: posteio
  ports:
  - port: 443
    targetPort: 443
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure Poste.io
  hosts: all
  become: yes
  tasks:
    - name: Install posteio
      package:
        name: posteio
        state: present
    
    - name: Configure posteio
      template:
        src: posteio.conf.j2
        dest: /data/posteio.conf
        owner: posteio
        group: posteio
        mode: '0640'
      notify: restart posteio
    
    - name: Start and enable posteio
      systemd:
        name: poste
        state: started
        enabled: yes
  
  handlers:
    - name: restart posteio
      systemd:
        name: poste
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update posteio

# Debian/Ubuntu
sudo apt update && sudo apt upgrade posteio

# Arch Linux
sudo pacman -Syu posteio

# Alpine Linux
apk update && apk upgrade posteio

# openSUSE
sudo zypper update posteio

# FreeBSD
pkg update && pkg upgrade posteio

# Always backup before updates
tar -czf /backup/posteio-pre-update-$(date +%Y%m%d).tar.gz /data

# Restart after updates
sudo systemctl restart poste
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/poste -name "*.log" -mtime +30 -delete

# Verify integrity
sudo posteio --verify || sudo poste check

# Update databases (if applicable)
sudo posteio-update-db

# Optimize performance
sudo posteio-optimize

# Check for security updates
sudo posteio --security-check
```

## Additional Resources

- Official Documentation: https://docs.posteio.org/
- GitHub Repository: https://github.com/posteio/posteio
- Community Forum: https://forum.posteio.org/
- Wiki: https://wiki.posteio.org/
- Comparison vs mailcow, Mailu, iRedMail, Zimbra: https://docs.posteio.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
