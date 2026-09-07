# VPS Traffic Optimizer Guide

[![License](https://img.shields.io/badge/license-CC%20BY--NC--SA%204.0-blue.svg)](LICENSE)
[![Stars](https://img.shields.io/github/stars/CG-spring/vps-traffic-optimizer?style=flat-square)](https://github.com/CG-spring/vps-traffic-optimizer/stargazers)

> Complete guide for VPS traffic monitoring, optimization and cost saving. From zero to hero.

**English** | [Chinese](README_ZH.md)

---

## Table of Contents

- [Why Traffic Optimization](#why-traffic-optimization)
- [Traffic Monitoring Tools](#traffic-monitoring-tools)
- [Traffic Saving Techniques](#traffic-saving-techniques)
- [CDN Acceleration](#cdn-acceleration)
- [Automated Scripts](#automated-scripts)
- [Cost Analysis](#cost-analysis)

---

## Why Traffic Optimization

### Traffic Cost Analysis

| Provider | Bandwidth | Overage | Rec |
|----------|-----------|---------|-----|
| Vultr | 500GB-2TB | $0.01/GB | 4/5 |
| DigitalOcean | 1TB-2TB | $0.01/GB | 4/5 |
| BandwagonHost | 500GB-1TB | Shutdown | 3/5 |
| Linode | 1TB-4TB | $0.02/GB | 4/5 |

### Traffic Breakdown

```
Website (Nginx)     40%
Database Sync       20%
Backup Transfer     15%
Proxy Service       15%
System Updates      10%
```

---

## Traffic Monitoring Tools

### 1. vnStat - Lightweight Stats

```bash
# Install
apt install vnstat -y

# Initialize database
vnstat --add -i eth0

# Start service
systemctl enable vnstat
systemctl start vnstat

# Real-time monitoring
vnstat -l

# Monthly stats
vnstat -m

# Daily stats
vnstat -d
```

### 2. nethogs - Per-Process Monitor

```bash
# Install
apt install nethogs -y

# Monitor by interface
nethogs eth0

# Monitor specific process
nethogs -p eth0
```

### 3. iftop - Real-time Connection

```bash
# Install
apt install iftop -y

# Monitor connections
iftop -i eth0

# Show ports
iftop -P -i eth0
```

### 4. bmon - Bandwidth Monitor

```bash
apt install bmon -y
bmon
```

---

## Traffic Saving Techniques

### 1. Enable Gzip Compression

```nginx
# /etc/nginx/nginx.conf
gzip on;
gzip_vary on;
gzip_min_length 1024;
gzip_comp_level 6;
gzip_types text/plain text/css text/xml text/javascript
    application/json application/javascript application/xml+rss
    application/x-javascript;
gzip_disable "MSIE [1-6]\.";

# Savings: 60-70% on text content
```

### 2. Browser Caching

```nginx
# Static resources - 30 day cache
location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff2|svg)$ {
    expires 30d;
    add_header Cache-Control "public, immutable";
}

# HTML - no cache
location ~* \.html$ {
    expires -1;
    add_header Cache-Control "no-store, no-cache";
}
```

### 3. Image Optimization

```bash
# Install ImageMagick
apt install imagemagick -y

# Compress all images in /var/www
find /var/www -type f -name "*.jpg" -exec convert {} -quality 85 {} \;
find /var/www -type f -name "*.png" -exec convert {} -quality 85 {} \;
```

### 4. Database Optimization

```sql
-- Enable slow query log
SET GLOBAL slow_query_log = 1;
SET GLOBAL long_query_time = 2;

-- Clean binary logs
PURGE BINARY LOGS BEFORE DATE_SUB(NOW(), INTERVAL 7 DAY);

-- Optimize tables
OPTIMIZE TABLE your_table;
```

### 5. Incremental Backup with rsync

```bash
#!/bin/bash
# Backup script - saves 80% bandwidth
BACKUP_DIR="/backup"
SOURCE_DIR="/var/www"

rsync -avz --delete --link-dest=$BACKUP_DIR/latest \
    $SOURCE_DIR/ $BACKUP_DIR/$(date +%Y%m%d)

# Keep only last 7 days
find $BACKUP_DIR -type d -mtime +7 -exec rm -rf {} \;

# Savings: 80% vs full backup every time
```

### 6. Log Rotation

```bash
# /etc/logrotate.d/nginx
/var/log/nginx/*.log {
    daily
    missingok
    rotate 7
    compress
    delaycompress
    notifempty
    create 0640 www-data adm
    sharedscripts
    size 10M
}
```

---

## CDN Acceleration

### Cloudflare Free Plan

```
Setup Steps:
1. Register at cloudflare.com
2. Add your domain
3. Update nameservers
4. Enable proxy (orange cloud)
5. Set SSL/TLS to Full

Benefits:
- Static assets: 70-90% traffic savings
- Global acceleration
- Free DDoS protection
- HTTP/2 support
```

### Cloudflare Page Rules

```yaml
Rule 1: *example.com/*
  Cache Level: Cache Everything
  Edge Cache TTL: 1 month
  Browser Cache TTL: 1 year

Rule 2: *example.com/api/*
  Cache Level: Bypass

Rule 3: *example.com/*.jpg
  Cache Level: Cache Everything
  Edge Cache TTL: 1 week
```

### Other CDN Options

| CDN | Free Tier | China PoP | Best For |
|-----|-----------|-----------|---------|
| Cloudflare | Unlimited | No | International |
| jsDelivr | 50GB/mo | Yes | Open source |
| BunnyCDN | 100GB/mo | Optional | General use |

---

## Automated Scripts

### Traffic Alert Script

```bash
#!/bin/bash
# traffic-alert.sh
# Requires: TELEGRAM_BOT_TOKEN and TELEGRAM_CHAT_ID

BOT_TOKEN="your_bot_token"
CHAT_ID="your_chat_id"
MONTHLY_LIMIT=500  # GB

# Get monthly traffic (GB)
used=$(vnstat -m | grep "$(date +%b '%y)" | awk '{print $4}' | sed 's/GiB//')

if (( $(echo "$used > $MONTHLY_LIMIT * 0.8" | bc -l) )); then
    pct=$(echo "scale=1; $used/$MONTHLY_LIMIT*100" | bc -l)
    msg="Traffic Alert%0AUsed: ${used}GB%0ALimit: ${MONTHLY_LIMIT}GB%0AUsage: ${pct}%%"
    curl -s "https://api.telegram.org/bot${BOT_TOKEN}/sendMessage?chat_id=${CHAT_ID}&text=${msg}"
fi
```

### Daily Traffic Report

```bash
#!/bin/bash
# traffic-report.sh

echo "=== Daily Traffic Report ==="
echo ""
echo "Today:"
vnstat -d | grep "$(date +%Y-%m-%d)" | awk '{print "  Download: "$2" "$3"\n  Upload: "$5" "$6"\n  Total: "$8" "$9}'
echo ""
echo "This month:"
vnstat -m | grep "$(date +%b '%y)" | awk '{print "  Download: "$2" "$3"\n  Upload: "$5" "$6"\n  Total: "$8" "$9}'
echo ""
echo "Top 10 connections:"
netstat -an | grep ESTABLISHED | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr | head -10

# Send via email
echo "$report" | mail -s "VPS Traffic Report $(date +%Y-%m-%d)" admin@example.com
```

### Cron Jobs

```bash
# Add to crontab -e
0 */6 * * * /root/traffic-alert.sh    # Check every 6 hours
0 8 * * * /root/traffic-report.sh      # Daily report at 8am
*/5 * * * * /root/traffic-monitor.sh  # Monitor every 5 minutes
```

---

## Cost Analysis

### Monthly Cost Calculator

```
Formula: (Base Cost + Bandwidth Overage) / Total Traffic

Example:
- Plan: $5/month, 500GB included
- Actual usage: 600GB
- Overage: 100GB x $0.01 = $1
- Total: $6
- Cost per GB: $6 / 600GB = $0.01/GB
```

### Traffic Estimation Formula

```
Monthly Traffic = Daily UV x Page Size x Pages Per Visit x 30 days

Example:
- Daily UV: 1000
- Page Size: 500KB (with gzip)
- Pages Per Visit: 3
- Monthly: 1000 x 0.5MB x 3 x 30 = 45GB
```

---

## Related Resources

- [VPS Reviews](https://vpsvip.net) - VPS provider comparisons
- [Clash Tutorial](https://clash-for-windows.net) - Proxy client guide
- [Airport Navigation](https://nav.clashvip.net) - VPN service recommendations
- [Community Forum](https://bbs.clashhub.net) - Technical discussions

---

## License

CC BY-NC-SA 4.0 - Educational use only, no commercial use
