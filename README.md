# VPS 娴侀噺浼樺寲涓庤妭鐪佹寚鍗?
[![License](https://img.shields.io/badge/license-CC%20BY--NC--SA%204.0-blue.svg)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/CG-spring/vps-traffic-optimizer.svg?style=flat-square)](https://github.com/CG-spring/vps-traffic-optimizer/stargazers)

> VPS 娴侀噺浼樺寲瀹屽叏鎸囧崡 - 鐩戞帶銆佽妭鐪併€佸姞閫熶竴绔欏紡瑙ｅ喅鏂规
> 
> 鎸佺画鏇存柊涓?| 鏈€鍚庢洿鏂? 2026-04-09

**涓枃** | **[English](README_EN.md)**

---

## 鐩綍

- [涓轰粈涔堥渶瑕佹祦閲忎紭鍖朷(#涓轰粈涔堥渶瑕佹祦閲忎紭鍖?
- [娴侀噺鐩戞帶宸ュ叿](#娴侀噺鐩戞帶宸ュ叿)
- [鑺傜渷娴侀噺鎶€宸(#鑺傜渷娴侀噺鎶€宸?
- [CDN 鍔犻€熸柟妗圿(#cdn-鍔犻€熸柟妗?
- [鑷姩鍖栬剼鏈琞(#鑷姩鍖栬剼鏈?
- [甯歌闂](#甯歌闂)

---

## 涓轰粈涔堥渶瑕佹祦閲忎紭鍖?
### 娴侀噺鎴愭湰鍒嗘瀽

| VPS 鍘傚晢 | 娴侀噺闄愬埗 | 瓒呴璐圭敤 | 鎺ㄨ崘鎸囨暟 |
|----------|----------|----------|----------|
| Vultr | 500GB-2TB | $0.01/GB | 猸愨瓙猸愨瓙 |
| DigitalOcean | 1TB-2TB | $0.01/GB | 猸愨瓙猸愨瓙 |
| BandwagonHost | 500GB-1TB | 鍋滄満 | 猸愨瓙猸?|
| [鏇村鍘傚晢瀵规瘮](https://vpsvip.net) | - | - | - |

### 娴侀噺娑堣€楁潵婧?
```
1. 缃戠珯鏈嶅姟 (Nginx/Apache) - 40%
2. 鏁版嵁搴撳悓姝?- 20%
3. 澶囦唤浼犺緭 - 15%
4. 浠ｇ悊鏈嶅姟 - 15%
5. 绯荤粺鏇存柊 - 10%
```

---

## 娴侀噺鐩戞帶宸ュ叿

### vnStat - 杞婚噺绾ф祦閲忕粺璁?
```bash
# 瀹夎
apt install vnstat -y

# 鍒濆鍖栨暟鎹簱
vnstat --add -i eth0

# 鏌ョ湅瀹炴椂娴侀噺
vnstat -l

# 鏌ョ湅鏈堝害缁熻
vnstat -m

# 鏌ョ湅鏃ョ粺璁?vnstat -d
```

### nethogs - 鎸夎繘绋嬬洃鎺?
```bash
# 瀹夎
apt install nethogs -y

# 瀹炴椂鐩戞帶鍚勮繘绋嬫祦閲?nethogs eth0
```

### iftop - 瀹炴椂杩炴帴鐩戞帶

```bash
# 瀹夎
apt install iftop -y

# 鏌ョ湅杩炴帴娴侀噺
iftop -i eth0
```

### 涓€閿洃鎺ц剼鏈?
```bash
#!/bin/bash
# 淇濆瓨涓? traffic-monitor.sh

echo "=== VPS 娴侀噺鐩戞帶 ==="
echo ""
echo "浠婃棩娴侀噺:"
vnstat -d | grep "$(date +%Y-%m-%d)" | awk '{print "  涓嬭浇: "$2" "$3"\n  涓婁紶: "$5" "$6"\n  鎬昏: "$8" "$9}'
echo ""
echo "鏈湀娴侀噺:"
vnstat -m | grep "$(date +%b '%y)" | awk '{print "  涓嬭浇: "$2" "$3"\n  涓婁紶: "$5" "$6"\n  鎬昏: "$8" "$9}'
echo ""
echo "褰撳墠娲昏穬杩炴帴 TOP 10:"
netstat -an | grep ESTABLISHED | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr | head -10
echo ""
echo "娴侀噺娑堣€?TOP 5 杩涚▼:"
nethogs -t -d 3 -c 1 2>/dev/null | grep -E "^[0-9]" | head -5
```

---

## 鑺傜渷娴侀噺鎶€宸?
### 1. 鍚敤 Gzip 鍘嬬缉

```nginx
# /etc/nginx/nginx.conf
gzip on;
gzip_vary on;
gzip_min_length 1024;
gzip_comp_level 6;
gzip_types text/plain text/css text/xml application/json application/javascript application/xml;
gzip_disable "MSIE [1-6]\.";

# 鑺傜渷鏁堟灉: 绾?60-70% 鏂囨湰娴侀噺
```

### 2. 鍚敤娴忚鍣ㄧ紦瀛?
```nginx
# 闈欐€佽祫婧愮紦瀛?location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff2)$ {
    expires 30d;
    add_header Cache-Control "public, immutable";
}
```

### 3. 鏁版嵁搴撲紭鍖?
```bash
# MySQL 鎱㈡煡璇紭鍖?mysql -e "SET GLOBAL slow_query_log = 1;"
mysql -e "SET GLOBAL long_query_time = 2;"

# 瀹氭湡娓呯悊鏃ュ織
mysql -e "PURGE BINARY LOGS BEFORE DATE_SUB(NOW(), INTERVAL 7 DAY);"
```

### 4. 澶囦唤浼樺寲

```bash
# 澧為噺澶囦唤鑴氭湰
#!/bin/bash
BACKUP_DIR="/backup"
SOURCE_DIR="/var/www"

# 浣跨敤 rsync 澧為噺鍚屾
rsync -avz --delete --link-dest=$BACKUP_DIR/latest $SOURCE_DIR/ $BACKUP_DIR/$(date +%Y%m%d)

# 鍒犻櫎 7 澶╁墠鐨勫浠?find $BACKUP_DIR -type d -mtime +7 -exec rm -rf {} \;

# 鑺傜渷鏁堟灉: 80% 澶囦唤娴侀噺
```

### 5. 闄愬埗鏃ュ織澶у皬

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

## CDN 鍔犻€熸柟妗?
### Cloudflare 鍏嶈垂鏂规

```
1. 娉ㄥ唽 Cloudflare 璐﹀彿
2. 娣诲姞鍩熷悕锛屼慨鏀?NS 鏈嶅姟鍣?3. 寮€鍚?CDN 浠ｇ悊锛堟鑹蹭簯鏈碉級
4. 閰嶇疆 SSL/TLS 涓?Full

鑺傜渷鏁堟灉: 
- 闈欐€佽祫婧愭祦閲忚妭鐪?70-90%
- 鍏ㄧ悆鍔犻€熻闂?- 鍏嶈垂 DDoS 闃叉姢
```

### Cloudflare 閰嶇疆浼樺寲

```yaml
# Page Rules 绀轰緥
瑙勫垯 1: *example.com/*
- Cache Level: Cache Everything
- Edge Cache TTL: 1 month
- Browser Cache TTL: 1 year

瑙勫垯 2: *example.com/api/*
- Cache Level: Bypass
```

### 鍏朵粬 CDN 閫夋嫨

| CDN | 鍏嶈垂棰濆害 | 鍥藉唴鑺傜偣 | 鎺ㄨ崘鐢ㄩ€?|
|-----|----------|----------|----------|
| Cloudflare | 鏃犻檺 | 鏃?| 鍥介檯绔?|
| jsDelivr | 50GB/鏈?| 鏈?| 寮€婧愰」鐩?|
| 鍙堟媿浜?| 10GB/鏈?| 鏈?| 灏忓瀷缃戠珯 |

---

## 鑷姩鍖栬剼鏈?
### 娴侀噺鍛婅鑴氭湰

```bash
#!/bin/bash
# 淇濆瓨涓? traffic-alert.sh
# 闇€瑕? TELEGRAM_BOT_TOKEN 鍜?TELEGRAM_CHAT_ID

TELEGRAM_BOT_TOKEN="your_bot_token"
TELEGRAM_CHAT_ID="your_chat_id"
MONTHLY_LIMIT=500  # GB

# 鑾峰彇鏈湀娴侀噺 (GB)
used=$(vnstat -m | grep "$(date +%b '%y)" | awk '{print $4}' | sed 's/GiB//')

if (( $(echo "$used > $MONTHLY_LIMIT * 0.8" | bc -l) )); then
    message="鈿狅笍 娴侀噺鍛婅%0A宸蹭娇鐢? ${used}GB%0A闄愰: ${MONTHLY_LIMIT}GB%0A浣跨敤鐜? 80%+"
    curl -s "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage?chat_id=${TELEGRAM_CHAT_ID}&text=${message}"
fi
```

### 姣忔棩娴侀噺鎶ュ憡

```bash
# 娣诲姞鍒?crontab -e
0 8 * * * /root/traffic-report.sh

#!/bin/bash
# traffic-report.sh

today=$(vnstat -d | grep "$(date +%Y-%m-%d)" | awk '{print "涓嬭浇 "$2$3" / 涓婁紶 "$5$6" / 鎬昏 "$8$9}')
echo "$today" | mail -s "姣忔棩娴侀噺鎶ュ憡" your@email.com
```

---

## 甯歌闂

### Q: 娴侀噺绐佺劧鏆村鎬庝箞鍔烇紵

```bash
# 1. 鏌ョ湅瀹炴椂杩炴帴
netstat -antp | grep ESTABLISHED | wc -l

# 2. 鏌ョ湅娴侀噺鏉ユ簮
iftop -i eth0 -n

# 3. 妫€鏌ユ槸鍚︽湁鏀诲嚮
tail -f /var/log/nginx/access.log | grep -E "(POST|DELETE)"

# 4. 涓存椂闄愬埗
iptables -A INPUT -p tcp --dport 80 -m limit --limit 100/minute -j ACCEPT
```

### Q: 濡備綍棰勪及娴侀噺闇€姹傦紵

```
鍏紡: 鏃ュ潎UV 脳 椤甸潰澶у皬 脳 璁块棶娣卞害 脳 30澶?
绀轰緥:
- 鏃ュ潎 UV: 1000
- 椤甸潰澶у皬: 500KB
- 璁块棶娣卞害: 3
- 鏈堟祦閲? 1000 脳 0.5MB 脳 3 脳 30 = 45GB
```

---

## 鐩稿叧璧勬簮

- [VPS 璇勬祴涓庢帹鑽怾(https://vpsvip.net) - 涓绘満娴嬭瘎缃戠珯
- [Clash 鏁欑▼](https://clash-for-windows.net) - 浠ｇ悊瀹㈡埛绔?- [鏈哄満瀵艰埅](https://nav.clashvip.net) - 鏈哄満鎺ㄨ崘
- [鐢ㄦ埛绀惧尯](https://bbs.clashhub.net) - 鎶€鏈氦娴?
---

## License

CC BY-NC-SA 4.0 - 浠呬緵瀛︿範浜ゆ祦锛岀姝㈠晢鐢?