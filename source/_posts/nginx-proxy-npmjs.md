---
title: Nginx反向代理registry.npmjs.org
date: 2025-01-14 10:59:25
tags: nginx
---

# 1. 反向代理上游仓库
nginx配置如下：

```
location / {
    proxy_pass https://registry.npmjs.org/;
    proxy_set_header Accept-Encoding "";
    proxy_set_header Host registry.npmjs.org;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_cache_bypass $http_upgrade;
    proxy_redirect off;
}
```

# 2. 设置pnpm的仓库源
修改仓库源的地址
```sh
$ pnpm config set registry https://your.domian.com
$ pnpm config get registry
https://your.domian.com
```
测试一下
```sh
$ pnpm info react
react@19.0.0 | MIT | deps: none | versions: 2143
React is a JavaScript library for building user interfaces.
https://react.dev/

keywords: react

dist
.tarball: https://registry.npmjs.org/react/-/react-19.0.0.tgz
.shasum: 6e1969251b9f108870aa4bff37a0ce9ddfaaabdd
.integrity: sha512-V8AVnmPIICiWpGfm6GLzCR/W5FXLchHop40W4nXBmdlEceh16rCN8O8LNWm5bh5XUX91fh7KpA+W0TgMKmgTpQ==
.unpackedSize: 237.5 kB

maintainers:
- gnoff <jcs.gnoff@gmail.com>
- fb <opensource+npm@fb.com>
- sophiebits <npm@sophiebits.com>
- react-bot <react-core@meta.com>

dist-tags:
beta: 19.0.0-beta-26f2496093-20240514               latest: 19.0.0
canary: 19.1.0-canary-cabd8a0e-20250113             next: 19.1.0-canary-cabd8a0e-20250113
experimental: 0.0.0-experimental-cabd8a0e-20250113  rc: 19.0.0-rc.1

published a month ago by react-bot <react-core@meta.com>
```

此时pnpm能够获取到包信息，但由于实际安装时是从`.tarball`中获取包的实际下载地址，还需要对结果进一步处理

# 3. 修改info的响应信息
nginx需要启用ngx_http_sub_module模块，使用`nginx -V`查看是否启用--with-http_sub_module
```sh
$ nginx -V
...
configure arguments: --with-http_sub_module
```

修改配置如下：

```
location / {
    proxy_pass https://registry.npmjs.org/;
    # 清空Accept-Encoding避免响应gzip，sub_filter读取到字节流无法生效
    proxy_set_header Accept-Encoding "";
    proxy_set_header Host registry.npmjs.org;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_cache_bypass $http_upgrade;
    proxy_redirect off;
    # 更改所有匹配的结果
    sub_filter_once off;
    # pnpm install时返回的content-type类型为application/vnd.npm.install-v1+json
    sub_filter_types application/vnd.npm.install-v1+json application/json;
    # 将info返回结果中的tarball地址更改为你的地址
    sub_filter 'https://registry.npmjs.org/' 'https://your.domian.com/';
}
```

再次测试下，可以看到`.tarball`的地址被替换为`https://your.domian.com/`
```sh
$ pnpm info react
react@19.0.0 | MIT | deps: none | versions: 2143
React is a JavaScript library for building user interfaces.
https://react.dev/

keywords: react

dist
.tarball: https://your.domian.com/react/-/react-19.0.0.tgz
```
