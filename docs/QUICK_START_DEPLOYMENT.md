# Quick Start Deployment Guide

## Option 1: Docker Deployment (Fastest)

### Prerequisites
- Docker Engine 20.10+
- Docker Compose V2
- Domain name with DNS configured
- 4GB RAM minimum

### Step-by-Step

```bash
# 1. Create working directory
mkdir -p /opt/peertube && cd /opt/peertube

# 2. Download Docker Compose files
curl -O https://raw.githubusercontent.com/chocobozzz/PeerTube/master/support/docker/production/docker-compose.yml
curl -O https://raw.githubusercontent.com/Chocobozzz/PeerTube/master/support/docker/production/.env

# 3. Generate secrets
PEERTUBE_SECRET=$(openssl rand -hex 32)
POSTGRES_PASSWORD=$(openssl rand -hex 16)

# 4. Configure environment
cat > .env << EOF
POSTGRES_USER=peertube
POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
POSTGRES_DB=peertube
PEERTUBE_DB_USERNAME=peertube
PEERTUBE_DB_PASSWORD=${POSTGRES_PASSWORD}
PEERTUBE_DB_HOSTNAME=postgres
PEERTUBE_WEBSERVER_HOSTNAME=your-domain.com
PEERTUBE_WEBSERVER_PORT=443
PEERTUBE_WEBSERVER_HTTPS=true
PEERTUBE_SECRET=${PEERTUBE_SECRET}
PEERTUBE_ADMIN_EMAIL=admin@your-domain.com
EOF

# 5. Setup nginx config
mkdir -p docker-volume/nginx
curl https://raw.githubusercontent.com/Chocobozzz/PeerTube/master/support/nginx/peertube > docker-volume/nginx/peertube

# 6. Get SSL certificate (stop any service on port 80 first)
mkdir -p docker-volume/certbot
docker run -it --rm --name certbot \
  -p 80:80 \
  -v "$(pwd)/docker-volume/certbot/conf:/etc/letsencrypt" \
  certbot/certbot certonly --standalone -d your-domain.com

# 7. Start PeerTube
docker compose up -d

# 8. Get admin password
docker compose logs peertube | grep -A1 "root"
```

### Post-Installation

```bash
# Change admin password
docker compose exec -u peertube peertube npm run reset-password -- -u root

# View logs
docker compose logs -f peertube

# Restart services
docker compose restart

# Update PeerTube
docker compose pull
docker compose down -v
docker compose up -d
```

---

## Option 2: Manual Installation (Ubuntu/Debian)

### 1. Install Dependencies

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Node.js 20
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Install pnpm
sudo npm install -g pnpm

# Install other dependencies
sudo apt install -y \
  postgresql postgresql-contrib \
  redis-server \
  ffmpeg \
  python3 python3-pip \
  nginx \
  certbot python3-certbot-nginx \
  git wget unzip g++ make

# Verify versions
node --version    # Should be >= 20.x
pnpm --version    # Should be >= 10.x
ffmpeg -version   # Should be >= 4.3
```

### 2. Create PeerTube User

```bash
sudo useradd -m -d /var/www/peertube -s /usr/sbin/nologin peertube
sudo chmod 755 /var/www/peertube
```

### 3. Setup Database

```bash
# Create PostgreSQL user and database
sudo -u postgres createuser -P peertube
# Enter a strong password when prompted

sudo -u postgres createdb -O peertube -E UTF8 -T template0 peertube_prod

# Enable required extensions
sudo -u postgres psql -c "CREATE EXTENSION pg_trgm;" peertube_prod
sudo -u postgres psql -c "CREATE EXTENSION unaccent;" peertube_prod
```

### 4. Download PeerTube

```bash
# Get latest version
VERSION=$(curl -s https://api.github.com/repos/chocobozzz/peertube/releases/latest | grep tag_name | cut -d '"' -f 4)
echo "Installing PeerTube $VERSION"

# Create directories
cd /var/www/peertube
sudo -u peertube mkdir -p config storage versions

# Download and extract
cd /var/www/peertube/versions
sudo -u peertube wget -q "https://github.com/Chocobozzz/PeerTube/releases/download/${VERSION}/peertube-${VERSION}.zip"
sudo -u peertube unzip -q peertube-${VERSION}.zip
sudo -u peertube rm peertube-${VERSION}.zip

# Create symlink
cd /var/www/peertube
sudo -u peertube ln -s versions/peertube-${VERSION} peertube-latest

# Install dependencies
cd peertube-latest
sudo -H -u peertube pnpm install --production
```

### 5. Configure PeerTube

```bash
cd /var/www/peertube

# Copy default config
sudo -u peertube cp peertube-latest/config/default.yaml config/default.yaml
sudo -u peertube cp peertube-latest/config/production.yaml.example config/production.yaml

# Generate secret
PEERTUBE_SECRET=$(openssl rand -hex 32)

# Edit configuration
sudo -u peertube nano config/production.yaml
```

**Key settings in production.yaml:**

```yaml
webserver:
  https: true
  hostname: 'your-domain.com'
  port: 443

secrets:
  peertube: 'YOUR_GENERATED_SECRET'

database:
  hostname: 'localhost'
  port: 5432
  suffix: '_prod'
  username: 'peertube'
  password: 'YOUR_DB_PASSWORD'

redis:
  hostname: 'localhost'
  port: 6379

smtp:
  hostname: 'smtp.example.com'
  port: 587
  username: 'your-smtp-user'
  password: 'your-smtp-password'
  tls: false
  disable_starttls: false
  from_address: 'noreply@your-domain.com'

admin:
  email: 'admin@your-domain.com'
```

### 6. Setup Nginx

```bash
# Copy nginx config
sudo cp /var/www/peertube/peertube-latest/support/nginx/peertube /etc/nginx/sites-available/peertube

# Edit config
sudo sed -i 's/${WEBSERVER_HOST}/your-domain.com/g' /etc/nginx/sites-available/peertube
sudo sed -i 's/${PEERTUBE_HOST}/127.0.0.1:9000/g' /etc/nginx/sites-available/peertube

# Enable site
sudo ln -s /etc/nginx/sites-available/peertube /etc/nginx/sites-enabled/

# Get SSL certificate
sudo certbot --nginx -d your-domain.com

# Test and reload nginx
sudo nginx -t
sudo systemctl reload nginx
```

### 7. Setup Systemd Service

```bash
# Copy service file
sudo cp /var/www/peertube/peertube-latest/support/systemd/peertube.service /etc/systemd/system/

# Enable and start
sudo systemctl daemon-reload
sudo systemctl enable peertube
sudo systemctl start peertube

# Check status
sudo systemctl status peertube
sudo journalctl -fu peertube
```

### 8. Get Admin Credentials

```bash
# Find in logs
sudo journalctl -u peertube | grep -A1 "root"

# Or reset password
cd /var/www/peertube/peertube-latest
sudo -u peertube NODE_CONFIG_DIR=/var/www/peertube/config NODE_ENV=production npm run reset-password -- -u root
```

---

## Post-Installation Configuration

### 1. Access Admin Panel

1. Go to `https://your-domain.com`
2. Login with `root` and the generated password
3. Navigate to Administration → Configuration

### 2. Essential Settings

**Instance Information:**
- Instance name
- Short description
- Terms of service
- Contact email

**Signup:**
- Enable/disable registration
- Require email verification
- Require admin approval

**Transcoding:**
- Enable transcoding
- Select resolutions (720p, 1080p recommended)
- Enable HLS (recommended)

**Live Streaming:**
- Enable if needed
- Configure RTMP port (1935)

### 3. Create First Channel

1. Go to My Account → Video Channels
2. Create a channel for your content

---

## Firewall Configuration

```bash
# UFW (Ubuntu)
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 1935/tcp  # If using live streaming

# Or iptables
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 1935 -j ACCEPT
```

---

## Backup Strategy

### Database Backup

```bash
# Create backup
sudo -u postgres pg_dump -F c peertube_prod > /backup/peertube_$(date +%Y%m%d).dump

# Restore backup
sudo -u postgres pg_restore -c -C -d postgres /backup/peertube_20240101.dump
```

### Storage Backup

```bash
# Backup storage directory
rsync -av /var/www/peertube/storage/ /backup/peertube-storage/

# Or use object storage for automatic redundancy
```

### Automated Backup Script

```bash
#!/bin/bash
# /opt/scripts/backup-peertube.sh

BACKUP_DIR="/backup/peertube"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

# Database
sudo -u postgres pg_dump -F c peertube_prod > $BACKUP_DIR/db_$DATE.dump

# Config
cp -r /var/www/peertube/config $BACKUP_DIR/config_$DATE

# Keep only last 7 days
find $BACKUP_DIR -mtime +7 -delete
```

Add to crontab:
```bash
0 2 * * * /opt/scripts/backup-peertube.sh
```

---

## Monitoring

### Basic Health Check

```bash
# Check if PeerTube is running
curl -s https://your-domain.com/api/v1/config | jq .instance.name

# Check disk space
df -h /var/www/peertube/storage

# Check memory
free -h

# Check CPU
top -bn1 | head -5
```

### Log Monitoring

```bash
# Application logs
tail -f /var/www/peertube/storage/logs/peertube.log

# Nginx access logs
tail -f /var/log/nginx/access.log

# System logs
journalctl -fu peertube
```

---

## Troubleshooting

### PeerTube Won't Start

```bash
# Check logs
sudo journalctl -u peertube -n 100

# Common issues:
# - Database connection: verify PostgreSQL is running and credentials are correct
# - Redis connection: verify Redis is running
# - Port conflict: check if port 9000 is available
```

### Videos Not Transcoding

```bash
# Check FFmpeg
ffmpeg -version

# Check job queue
redis-cli LLEN bull:transcoding:waiting

# Check transcoding logs
grep -i transcode /var/www/peertube/storage/logs/peertube.log
```

### SSL Certificate Issues

```bash
# Renew certificate
sudo certbot renew

# Check certificate
sudo certbot certificates

# Test nginx config
sudo nginx -t
```

---

## Next Steps

1. **Customize branding** - Add logo, colors, custom CSS
2. **Configure email** - Set up SMTP for notifications
3. **Enable features** - Live streaming, transcription, etc.
4. **Install plugins** - Browse available plugins in admin panel
5. **Set up monitoring** - Configure alerts for disk space, errors
6. **Plan scaling** - Consider object storage, CDN, load balancing

For your gaming platform MVP, proceed to create:
1. Custom theme with your branding
2. Subscription plugin for payment integration
3. Gaming-specific video categories
