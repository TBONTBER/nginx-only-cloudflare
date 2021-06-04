# 介绍/Introduction
此项目目的是保护我们的nginx、网站应用不被其它IP探测，防止服务端主机的IP地址被加入黑名单之中。Bash脚本可以被设定成定时任务用于实时更新最新Cloudflare的IP列表。

This project aims to protect your nginx or web server from being detected. Bash script can be scheduled to create an automated up-to-date Cloudflare ip list file.

## Nginx 配置/Configuration
在“/etc/nginx/nginx.conf”的http{....}中增加下面配置。

Open "/etc/nginx/nginx.conf" file and just add the following lines to your nginx.conf inside http{....} block.

```nginx
include /etc/nginx/cloudflare;
```

“/etc/nginx/cloudflare”由Bash脚本生成，Bash脚本可以手动执行，也可以设置为定时任务。

The bash script, generating "/etc/nginx/cloudflare", may run manually or can be scheduled to refresh the ip list of CloudFlare automatically.
```sh
#!/bin/bash

CLOUDFLARE_FILE_PATH=/etc/nginx/cloudflare

echo "#Cloudflare" > $CLOUDFLARE_FILE_PATH;
echo "" >> $CLOUDFLARE_FILE_PATH;

echo "# - IPv4" >> $CLOUDFLARE_FILE_PATH;
for i in `curl https://www.cloudflare.com/ips-v4`; do
        echo "allow $i;" >> $CLOUDFLARE_FILE_PATH;
done

echo "" >> $CLOUDFLARE_FILE_PATH;
echo "# - IPv6" >> $CLOUDFLARE_FILE_PATH;
for i in `curl https://www.cloudflare.com/ips-v6`; do
        echo "all $i;" >> $CLOUDFLARE_FILE_PATH;
done

echo "" >> $CLOUDFLARE_FILE_PATH;
echo "deny all;" >> $CLOUDFLARE_FILE_PATH;

#test configuration and reload nginx
nginx -t && systemctl reload nginx
```

## 定时任务/Crontab
你可以把“cloudflare-sync-ips.sh”放在任何位置，比如“/opt/scripts/”目录下，然后把下面内容添加到定时任务当中。

Change the location of "cloudflare-sync-ips.sh" anywhere you want, e.g. "/opt/scripts/", and add the following lines to your crontab. 
```sh
# Auto sync ip addresses of Cloudflare and reload nginx
30 2 * * * /opt/scripts/cloudflare-sync-ips.sh >/dev/null 2>&1
```

### 许可证/License

[Apache 2.0](http://www.apache.org/licenses/LICENSE-2.0)

### 感谢/Thanks

https://github.com/ergin/nginx-cloudflare-real-ip.git
