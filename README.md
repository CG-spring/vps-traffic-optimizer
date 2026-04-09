# VPS 流量优化与节省指南

[![License](https://img.shields.io/badge/license-CC%20BY--NC--SA%204.0-blue.svg)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/CG-spring/vps-traffic-optimizer.svg?style=flat-square)](https://github.com/CG-spring/vps-traffic-optimizer/stargazers)

> VPS 流量优化完全指南 - 监控、节省、加速一站式解决方案
> 
> 持续更新中 | 最后更新: 2026-04-09

**中文** | **[English](README_EN.md)**

---

## 目录

- [为什么需要流量优化](#为什么需要流量优化)
- [流量监控工具](#流量监控工具)
- [节省流量技巧](#节省流量技巧)
- [CDN 加速方案](#cdn-加速方案)
- [自动化脚本](#自动化脚本)
- [常见问题](#常见问题)

---

## 为什么需要流量优化

### 流量成本分析

| VPS 厂商 | 流量限制 | 超额费用 | 推荐指数 |
|----------|----------|----------|----------|
| Vultr | 500GB-2TB | $0.01/GB | ⭐⭐⭐⭐ |
| DigitalOcean | 1TB-2TB | $0.01/GB | ⭐⭐⭐⭐ |
| BandwagonHost | 500GB-1TB | 停机 | ⭐⭐⭐ |
| [更多厂商对比](https://vpsvip.net) | - | - | - |

### 流量消耗来源

```
1. 网站服务 (Nginx/Apache) - 40%
2. 数据库同步 - 20%
3. 备份传输 - 15%
4. 代理服务 - 15%
5. 系统更新 - 10%
```

---

## 流量监控工具

### vnStat - 轻量级流量统计

```bash
# 安装
apt install vnstat -y

# 初始化数据库
vnstat --add -i eth0

# 查看实时流量
vnstat -l

# 查看月度统计
vnstat -m

# 查看日统计
vnstat -d
```

### nethogs - 按进程监控

```bash
# 安装
apt install nethogs -y

# 实时监控各进程流量
nethogs eth0
```

### iftop - 实时连接监控

```bash
# 安装
apt install iftop -y

# 查看连接流量
iftop -i eth0
```

### 一键监控脚本

```bash
#!/bin/bash
# 保存为: traffic-monitor.sh

echo "=== VPS 流量监控 ==="
echo ""
echo "今日流量:"
vnstat -d | grep "$(date +%Y-%m-%d)" | awk '{print "  下载: "$2" "$3"\n  上传: "$5" "$6"\n  总计: "$8" "$9}'
echo ""
echo "本月流量:"
vnstat -m | grep "$(date +%b '%y)" | awk '{print "  下载: "$2" "$3"\n  上传: "$5" "$6"\n  总计: "$8" "$9}'
echo ""
echo "当前活跃连接 TOP 10:"
netstat -an | grep ESTABLISHED | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr | head -10
echo ""
echo "流量消耗 TOP 5 进程:"
nethogs -t -d 3 -c 1 2>/dev/null | grep -E "^[0-9]" | head -5
```

---

## 节省流量技巧

### 1. 启用 Gzip 压缩

```nginx
# /etc/nginx/nginx.conf
gzip on;
gzip_vary on;
gzip_min_length 1024;
gzip_comp_level 6;
gzip_types text/plain text/css text/xml application/json application/javascript application/xml;
gzip_disable "MSIE [1-6]\.";

# 节省效果: 约 60-70% 文本流量
```

### 2. 启用浏览器缓存

```nginx
# 静态资源缓存
location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff2)$ {
    expires 30d;
    add_header Cache-Control "public, immutable";
}
```

### 3. 数据库优化

```bash
# MySQL 慢查询优化
mysql -e "SET GLOBAL slow_query_log = 1;"
mysql -e "SET GLOBAL long_query_time = 2;"

# 定期清理日志
mysql -e "PURGE BINARY LOGS BEFORE DATE_SUB(NOW(), INTERVAL 7 DAY);"
```

### 4. 备份优化

```bash
# 增量备份脚本
#!/bin/bash
BACKUP_DIR="/backup"
SOURCE_DIR="/var/www"

# 使用 rsync 增量同步
rsync -avz --delete --link-dest=$BACKUP_DIR/latest $SOURCE_DIR/ $BACKUP_DIR/$(date +%Y%m%d)

# 删除 7 天前的备份
find $BACKUP_DIR -type d -mtime +7 -exec rm -rf {} \;

# 节省效果: 80% 备份流量
```

### 5. 限制日志大小

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

## CDN 加速方案

### Cloudflare 免费方案

```
1. 注册 Cloudflare 账号
2. 添加域名，修改 NS 服务器
3. 开启 CDN 代理（橙色云朵）
4. 配置 SSL/TLS 为 Full

节省效果: 
- 静态资源流量节省 70-90%
- 全球加速访问
- 免费 DDoS 防护
```

### Cloudflare 配置优化

```yaml
# Page Rules 示例
规则 1: *example.com/*
- Cache Level: Cache Everything
- Edge Cache TTL: 1 month
- Browser Cache TTL: 1 year

规则 2: *example.com/api/*
- Cache Level: Bypass
```

### 其他 CDN 选择

| CDN | 免费额度 | 国内节点 | 推荐用途 |
|-----|----------|----------|----------|
| Cloudflare | 无限 | 无 | 国际站 |
| jsDelivr | 50GB/月 | 有 | 开源项目 |
| 又拍云 | 10GB/月 | 有 | 小型网站 |

---

## 自动化脚本

### 流量告警脚本

```bash
#!/bin/bash
# 保存为: traffic-alert.sh
# 需要: TELEGRAM_BOT_TOKEN 和 TELEGRAM_CHAT_ID

TELEGRAM_BOT_TOKEN="your_bot_token"
TELEGRAM_CHAT_ID="your_chat_id"
MONTHLY_LIMIT=500  # GB

# 获取本月流量 (GB)
used=$(vnstat -m | grep "$(date +%b '%y)" | awk '{print $4}' | sed 's/GiB//')

if (( $(echo "$used > $MONTHLY_LIMIT * 0.8" | bc -l) )); then
    message="⚠️ 流量告警%0A已使用: ${used}GB%0A限额: ${MONTHLY_LIMIT}GB%0A使用率: 80%+"
    curl -s "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage?chat_id=${TELEGRAM_CHAT_ID}&text=${message}"
fi
```

### 每日流量报告

```bash
# 添加到 crontab -e
0 8 * * * /root/traffic-report.sh

#!/bin/bash
# traffic-report.sh

today=$(vnstat -d | grep "$(date +%Y-%m-%d)" | awk '{print "下载 "$2$3" / 上传 "$5$6" / 总计 "$8$9}')
echo "$today" | mail -s "每日流量报告" your@email.com
```

---

## 常见问题

### Q: 流量突然暴增怎么办？

```bash
# 1. 查看实时连接
netstat -antp | grep ESTABLISHED | wc -l

# 2. 查看流量来源
iftop -i eth0 -n

# 3. 检查是否有攻击
tail -f /var/log/nginx/access.log | grep -E "(POST|DELETE)"

# 4. 临时限制
iptables -A INPUT -p tcp --dport 80 -m limit --limit 100/minute -j ACCEPT
```

### Q: 如何预估流量需求？

```
公式: 日均UV × 页面大小 × 访问深度 × 30天

示例:
- 日均 UV: 1000
- 页面大小: 500KB
- 访问深度: 3
- 月流量: 1000 × 0.5MB × 3 × 30 = 45GB
```

---

## 相关资源

- [VPS 评测与推荐](https://vpsvip.net) - 主机测评网站
- [Clash 教程](https://clash-for-windows.net) - 代理客户端
- [机场导航](https://nav.clashvip.net) - 机场推荐
- [用户社区](https://bbs.clashhub.net) - 技术交流

---

## License

CC BY-NC-SA 4.0 - 仅供学习交流，禁止商用
