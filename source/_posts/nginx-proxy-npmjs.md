---
title: Nginx反向代理registry.npmjs.org
date: 2025-01-14 10:59:25
tags: nginx
---

 <!-- more -->
```
server {
    listen     443 ssl;
    listen [::]:443 ssl;
    server_name  _;
    location / {
        proxy_pass https://registry.npmjs.org/;
        proxy_set_header Accept-Encoding "";
        proxy_set_header Host registry.npmjs.org;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        proxy_redirect off;
        sub_filter_once off;
        sub_filter_types application/vnd.npm.install-v1+json application/json;
        sub_filter 'registry.npmjs.org' 'npm.zzflying.cn';
    }
}
```