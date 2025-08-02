---
title: Rebuild Proxmox Server
date: 2025-04-28 00:00:00 +0900
categories: [proxmox]
tags: [proxmox]
# TAG names should always be lowercase
---
Update Date: 2025-04-28
## TL;DR
因為Proxmox 主機不小心被我搞爛了，因此趁著這次rebuild，希望可以記錄一下所有基礎建設。

## Proxmox
因為溫度很重要，所以最好即時監測溫度：[link](https://github.com/Meliox/PVE-mods)

## 100 - OpenVPN & Nginx
### OpenVPN
因為人在國外，最重要的還是VPN。沒有VPN的日子，無法看動畫瘋跟有字幕的Netflix。
我參考了[BlueMonkey 4n6](https://www.youtube.com/watch?v=nsy9acOKnPo&ab_channel=BlueMonkey4n6)來重新建立OpenVPN。

### Nginx
Nginx 相對比較麻煩，因為還要寫.conf。
這裡附上目前的conf:
```
server {
        listen 80;
        server_name {server_name, e.g. prox.aaa.com};

        access_log /var/log/nginx/proxmox.access.log;
        error_log  /var/log/nginx/proxmox.error.log;

        location / {
        proxy_pass         https://192.168.0.46:8006;
        proxy_http_version 1.1;
        proxy_set_header   Host                 $host;
        proxy_set_header   X-Real-IP            $remote_addr;
        proxy_set_header   X-Forwarded-For      $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto    $scheme;

        # WebSocket-specific headers (avoid undefined, 1006)
        proxy_set_header Upgrade                $http_upgrade;
        proxy_set_header Connection             "Upgrade";


        proxy_buffering    off;
        proxy_read_timeout 360s;
        }
}

```