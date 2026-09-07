# VPS 流量优化与节省指南

[![License](https://img.shields.io/badge/license-CC%20BY--NC--SA%204.0-blue.svg)](LICENSE)
[![Stars](https://img.shields.io/github/stars/CG-spring/vps-traffic-optimizer?style=flat-square)](https://github.com/CG-spring/vps-traffic-optimizer/stargazers)

> 完整的 VPS 流量监控、优化与省钱指南——从零到精通

**中文** | **[English](README_EN.md)**

---

## 目录

- [为什么要做流量优化](#为什么要做流量优化)
- [流量监控工具](#流量监控工具)
- [流量节省技巧](#流量节省技巧)
- [CDN 加速](#cdn-加速)
- [自动化脚本](#自动化脚本)
- [成本分析](#成本分析)
- [相关资源](#相关资源)
- [许可证](#许可证)

---

## 为什么要做流量优化

很多 VPS 套餐按流量计费或超出套餐后高额扣费，流量优化能直接省钱、避免被关机。

### 流量成本对比

| 服务商 | 带宽/流量 | 超出计费 | 推荐 |
|--------|-----------|----------|------|
| Vultr | 500GB–2TB | $0.01/GB | 4/5 |
| DigitalOcean | 1TB–2TB | $0.01/GB | 4/5 |
| BandwagonHost | 500GB–1TB | 直接关机 | 3/5 |
| Linode | 1TB–4TB | $0.02/GB | 4/5 |

### 流量构成拆解

```
网站（Nginx）      40%
数据库同步          20%
备份传输            15%
代理服务            15%
系统更新            10%
```

> 找准「流量大户」才能对症下药：通常网站静态资源与备份传输是优化重点。

---

## 流量监控工具

不做监控就无从优化。下面三款工具覆盖「总量 / 进程 / 连接」三个层面。

### 1. vnStat —— 轻量统计

```bash
# 安装
apt install vnstat -y

# 初始化数据库
vnstat --add -i eth0

# 启动服务
systemctl enable vnstat
systemctl start vnstat

# 实时监测
vnstat -l

# 月度统计
vnstat -m

# 每日统计
vnstat -d
```

### 2. nethogs —— 按进程统计

```bash
# 安装
apt install nethogs -y

# 按网卡监测
nethogs eth0

# 指定进程
nethogs -p eth0
```

### 3. iftop —— 实时连接

```bash
# 安装
apt install iftop -y

# 监测连接
iftop -i eth0

# 显示端口
iftop -P -i eth0
```

### 4. bmon —— 带宽监测

```bash
apt install bmon -y
bmon
```

---

## 流量节省技巧

### 1. 开启 Gzip 压缩

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

# 效果：文本内容可节省 60–70%
```

### 2. 浏览器缓存

```nginx
# 静态资源缓存 30 天
location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff2|svg)$ {
    expires 30d;
    add_header Cache-Control "public, immutable";
}

# HTML 不缓存
location ~* \.html$ {
    expires -1;
    add_header Cache-Control "no-store, no-cache";
}
```

### 3. 图片优化

```bash
# 安装 ImageMagick
apt install imagemagick -y

# 压缩网站全部图片
find /var/www -type f -name "*.jpg" -exec convert {} -quality 85 {} \;
find /var/www -type f -name "*.png" -exec convert {} -quality 85 {} \;
```

### 4. 数据库优化

```sql
-- 开启慢查询日志
SET GLOBAL slow_query_log = 1;
SET GLOBAL long_query_time = 2;

-- 清理二进制日志
PURGE BINARY LOGS BEFORE DATE_SUB(NOW(), INTERVAL 7 DAY);

-- 优化表
OPTIMIZE TABLE your_table;
```

### 5. rsync 增量备份

```bash
#!/bin/bash
# 备份脚本——可节省约 80% 带宽
BACKUP_DIR="/backup"
SOURCE_DIR="/var/www"

rsync -avz --delete --link-dest=$BACKUP_DIR/latest \
    $SOURCE_DIR/ $BACKUP_DIR/$(date +%Y%m%d)

# 仅保留最近 7 天
find $BACKUP_DIR -type d -mtime +7 -exec rm -rf {} \;

# 相比每次全量备份，节省约 80% 流量
```

### 6. 日志轮转

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

## CDN 加速

把静态资源交给 CDN 边缘节点，源站流量可大幅下降。

### Cloudflare 免费版

```
配置步骤：
1. 在 cloudflare.com 注册
2. 添加你的域名
3. 更新 NS 解析
4. 开启代理（橙色云朵）
5. 将 SSL/TLS 设为 Full

收益：
- 静态资源：节省 70–90% 流量
- 全球加速
- 免费 DDoS 防护
- 支持 HTTP/2
```

### Cloudflare Page Rules

```yaml
规则 1：*example.com/*
  缓存级别：Cache Everything
  边缘缓存 TTL：1 个月
  浏览器缓存 TTL：1 年

规则 2：*example.com/api/*
  缓存级别：Bypass

规则 3：*example.com/*.jpg
  缓存级别：Cache Everything
  边缘缓存 TTL：1 周
```

### 其他 CDN 选项

| CDN | 免费额度 | 国内节点 | 适合 |
|-----|----------|----------|------|
| Cloudflare | 不限量 | 无 | 国际站点 |
| jsDelivr | 50GB/月 | 有 | 开源项目 |
| BunnyCDN | 100GB/月 | 可选 | 通用 |

---

## 自动化脚本

### 流量告警脚本

```bash
#!/bin/bash
# traffic-alert.sh（需配置 TELEGRAM_BOT_TOKEN 与 TELEGRAM_CHAT_ID）

BOT_TOKEN="your_bot_token"
CHAT_ID="your_chat_id"
MONTHLY_LIMIT=500  # GB

used=$(vnstat -m | grep "$(date +%b '%y)" | awk '{print $4}' | sed 's/GiB//')

if (( $(echo "$used > $MONTHLY_LIMIT * 0.8" | bc -l) )); then
    pct=$(echo "scale=1; $used/$MONTHLY_LIMIT*100" | bc -l)
    msg="流量提醒%0A已用: ${used}GB%0A限额: ${MONTHLY_LIMIT}GB%0A使用率: ${pct}%%"
    curl -s "https://api.telegram.org/bot${BOT_TOKEN}/sendMessage?chat_id=${CHAT_ID}&text=${msg}"
fi
```

### 每日流量报告

```bash
#!/bin/bash
# traffic-report.sh

echo "=== 每日流量报告 ==="
echo "今日:"
vnstat -d | grep "$(date +%Y-%m-%d)" | awk '{print "  下载: "$2" "$3"\n  上传: "$5" "$6"\n  合计: "$8" "$9}'
echo "本月:"
vnstat -m | grep "$(date +%b '%y)" | awk '{print "  下载: "$2" "$3"\n  上传: "$5" "$6"\n  合计: "$8" "$9}'
echo "Top 10 连接:"
netstat -an | grep ESTABLISHED | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr | head -10
```

### 定时任务

```bash
# crontab -e
0 */6 * * * /root/traffic-alert.sh    # 每 6 小时检查
0 8 * * * /root/traffic-report.sh      # 每天 8 点报告
*/5 * * * * /root/traffic-monitor.sh  # 每 5 分钟监测
```

---

## 成本分析

### 月度成本计算器

```
公式：（基础费用 + 超出流量费）/ 总流量

示例：
- 套餐：$5/月，含 500GB
- 实际用量：600GB
- 超出：100GB × $0.01 = $1
- 合计：$6
- 单 GB 成本：$6 / 600GB = $0.01/GB
```

### 流量估算公式

```
月流量 = 日均 UV × 页面大小 × 每次访问页数 × 30 天

示例：
- 日均 UV：1000
- 页面大小：500KB（已开启 gzip）
- 每次访问页数：3
- 月流量：1000 × 0.5MB × 3 × 30 = 45GB
```

---

## 相关资源

- [VPS 测评](https://vpsvip.net) - VPS 服务商对比
- [Clash 教程](https://clash-for-windows.net) - 代理客户端指南
- [机场导航](https://nav.clashvip.net) - VPN 节点推荐
- [社区论坛](https://bbs.clashhub.net) - 技术讨论

---

## 许可证

CC BY-NC-SA 4.0 - 仅限教育用途，禁止商用
