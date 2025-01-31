# Squid Proxy + Socks5

###### Video : لینک ویدیو یوتیوب
```

```

###### خرید دامنه از نیم چیپ: 
```
https://namecheap.pxf.io/BX7m6W
```
###### خرید دامنه سایت ایرانی: 
```
https://dashboard.azaronline.com/order/?aff=790&p=domain
```
###### خرید سرور از دیجیتال اوشن : 
```
https://m.do.co/c/0fb522deafa4
```
###### خرید سرور از سایت ایرانی : 
```
https://dashboard.azaronline.com/order/?aff=790&p=vps
```

**If you think this project is helpful to you, you may wish to give a** 🌟

**Feel Free To Donation:** ❤️

>TRC20: ```TGTyqv2MH7dZztMvaP5PKuS9Bma8RY5Pk8```

>ETH: ```0x5b5202a54e5ce4fb25f0d886254eeb07bb088614```

___
---
***
#### Update & Upgrade server:
```
sudo apt update && sudo apt upgrade -y
```

#### Install Squid:
```
sudo apt install squid -y
```

#### Configuration:
```
sudo nano /etc/squid/squid.conf
```

##### Then Paste the following Commands:
```
http_port 3128
acl localnet src 0.0.0.0/0
http_access allow localnet
forwarded_for delete
request_header_access X-Forwarded-For deny all
via off

```

##### Save and Close Nano.

#### Activate Log:
```
access_log /var/log/squid/access.log squid
```

#### Restart Squid:
```
sudo systemctl restart squid
```

#### Install Dante for Socks5:
```
sudo apt install dante-server -y
```

#### Configuration:
```
sudo nano /etc/danted.conf
```

#### Paste the following Commands:
```
method: username
user.privileged: root
user.unprivileged: nobody
accesslog: /var/log/dante.log

internal: 0.0.0.0 port = 1080
external: eth0

client pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
}

socks pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    log: connect error
}


route {
    from: 0.0.0.0/0 to: 0.0.0.0/0 via 127.0.0.1 port = 3128
}

```

#### Restart Dante:
```
sudo systemctl restart danted
```

#### How to add User
```
sudo useradd -M USERNAME
sudo passwd USERNAME
```

##### -M causes the user to be created without a home directory.

##### Ex:
```
sudo useradd -M Vira
sudo passwd Vira
```

#### Create T.Monitor:
```
sudo nano /usr/local/bin/traffic_monitor.sh
```

#### Paste the following Commands:
```
#!/bin/bash

LOG_FILE="/var/log/squid/access.log"
USERS_DIR="/etc/proxy_users"

# اطمینان از اینکه دایرکتوری کاربران ایجاد شده
mkdir -p $USERS_DIR

# خواندن لیست کاربران از /etc/passwd
for user in $(awk -F: '{print $1}' /etc/passwd); do
    # بررسی آیا یوزر جزو پروکسی هست یا نه
    if id -u $user >/dev/null 2>&1; then
        USER_FILE="$USERS_DIR/$user"

        # اگر فایل یوزر وجود نداشت، ایجادش کن و تاریخ شروع رو ثبت کن
        if [ ! -f "$USER_FILE" ]; then
            echo "START_DATE=$(date +%s)" > "$USER_FILE"
            echo "USED_MB=0" >> "$USER_FILE"
        fi

        # خواندن مقدار قبلی حجم مصرف شده
        USED_MB=$(grep "USED_MB" "$USER_FILE" | cut -d '=' -f2)

        # محاسبه مصرف جدید
        NEW_USAGE=$(grep $user $LOG_FILE | awk '{sum+=$5} END {print sum/1024/1024}')
        if [ -z "$NEW_USAGE" ]; then
            NEW_USAGE=0
        fi

        # اضافه کردن مصرف جدید به مصرف قبلی
        TOTAL_USAGE=$(echo "$USED_MB + $NEW_USAGE" | bc)
        echo "USED_MB=$TOTAL_USAGE" > "$USER_FILE"

        # بررسی اگر مصرف از ۵۰ گیگ بیشتر شد، کاربر رو حذف کن
        LIMIT_MB=51200  # ۵۰ گیگ
        if (( $(echo "$TOTAL_USAGE >= $LIMIT_MB" | bc -l) )); then
            sudo userdel -r $user
            rm "$USER_FILE"
            echo "User $user has been removed due to exceeding quota." >> /var/log/proxy_limit.log
        fi

        # بررسی زمان انقضا
        START_DATE=$(grep "START_DATE" "$USER_FILE" | cut -d '=' -f2)
        CURRENT_DATE=$(date +%s)
        EXPIRE_DAYS=30
        EXPIRE_TIME=$((START_DATE + EXPIRE_DAYS*24*3600))

        if [ "$CURRENT_DATE" -ge "$EXPIRE_TIME" ]; then
            sudo userdel -r $user
            rm "$USER_FILE"
            echo "User $user has been removed due to expiration." >> /var/log/proxy_limit.log
        fi
    fi
done
```

#### Run Script:
```
sudo chmod +x /usr/local/bin/traffic_monitor.sh
```

#### And let it run every hour:
```
(crontab -l ; echo "0 * * * * /usr/local/bin/traffic_monitor.sh") | crontab -
```

#### For more mischief You can Edit Squid log :
```
access_log /var/log/squid/access.log squid
logformat combined %>a %ui %un [%tl] "%rm %ru HTTP/%rv" %Hs %<st "%{Referer}>h" "%{User-Agent}>h"
```

#### Live Log:
```
sudo tail -f /var/log/squid/access.log
```

#### Filter Log:
```
cat /var/log/squid/access.log | grep "IP-USER"
```

