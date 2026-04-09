# VPS Traffic Optimizer Guide

[![License](https://img.shields.io/badge/license-CC%20BY--NC--SA%204.0-blue.svg)](LICENSE)

> Complete guide for VPS traffic monitoring, optimization and cost saving

## Traffic Monitoring

### vnStat Installation

```bash
apt install vnstat -y
vnstat --add -i eth0
vnstat -l
```

### nethogs Process Monitor

```bash
apt install nethogs -y
nethogs eth0
```

## Traffic Saving

### Enable Gzip

```nginx
gzip on;
gzip_comp_level 6;
gzip_types text/plain application/json application/javascript;
```

### Browser Cache

```nginx
location ~* \.(jpg|png|css|js)$ {
    expires 30d;
}
```

## CDN

Cloudflare free plan provides unlimited bandwidth with 70-90% traffic savings.

## Resources

- https://vpsvip.net
- https://nav.clashvip.net

## License

CC BY-NC-SA 4.0
