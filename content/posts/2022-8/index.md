---
title: "Nginx部署Hugo站点"
date: 2022-03-18T11:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Blog,Git,Hugo,Linux,Nginx]
series: [Deploy Blog on Own Server]
series_weight: 1
featuredImagePreview: "/2022-8/featuredImagePreview.svg"
summary: 将本地 Git 仓库提交到位于云服务器的 Remote 仓库, 通过 Git Hooks 在接收到 push 行为后, 运行脚本以达成构建 Hugo 博客、同步到多个 Git Repo 源等多个目的。
---

{{< image src="step.png" caption="流程" height="auto" width="100%">}}

## 安装软件

- Git: https://git-scm.com/downloads
- Hugo: https://github.com/gohugoio/hugo/releases/latest, 推荐使用 `extended` 版本
- Nginx: https://nginx.org/en/linux_packages.html
- acme.sh: [https://github.com/acmesh-official/acme.sh/wiki/说明](https://github.com/acmesh-official/acme.sh/wiki/说明)

## 配置仓库

### 服务器中的Remote仓库

> 以下是在云服务器中的操作。

{{< admonition type=question title="是否需要新建git用户" open=true >}}
由于SSH方式连接Git仓库, 参数格式是这样的: `ssh://<UserName>@<domain|ip>:[port]/xxx.git`, 所以: 
- 如果新建名为`git`的用户, 那么之后Git仓库地址是`git@<domain|ip>:[port]/xxx.git`, 符合习惯, 比如常用的`git@github.com`。
- 直接使用现有的普通用户, 比如`senzyo`, 之后Git仓库地址则是`ssh://senzyo@<domain|ip>:[port]/xxx.git`。
{{< /admonition >}}

1. 初始化裸仓库: 

    ```bash
    cd /home/senzyo/git-repos/blog
    git init --bare .git
    ```

2. 编辑Git Hooks: 

    ```bash
    vim .git/hooks/post-receive
    ```

    粘贴以下内容后保存退出: 

    ```bash
    #!/bin/bash
    
    BRANCH="master"
    GIT_DIR="/home/senzyo/project/blog/.git"
    TARGET="/home/senzyo/project/blog"
    
    read oldrev newrev ref
    
    if [ "$ref" == "refs/heads/$BRANCH" ]; then
        echo "$ref已成功接收, 正在部署$BRANCH分支"
        git --work-tree="$TARGET" --git-dir="$GIT_DIR" pull
        cd $TARGET
        rm -rf public
        hugo --gc --minify
    else
        echo "$ref已成功接收, 但只有$BRANCH分支更改时才进行部署"
    fi
    
    git push -f github
    git push -f gitee
    ```
    
3. 赋予`post-receive`可执行权限: 
   
    ```bash
    chmod +x .git/hooks/post-receive
    ```

### 服务器中的生产环境仓库

> 以下是在云服务器中的操作。

{{< admonition type=question title="服务器有必要两个仓库吗" open=true >}}
为了方便搞事, 比如自动更新twikoo、同步到多个Git Repo源等, 所以服务器上有两个Git仓库, 一个是Remote, 一个是生产环境, `nginx.conf`中`location`的`root`路径指向的正是生产环境。

如果只需要在本地`push`到Remote后, 自动构建Hugo博客, 可以从Remote的`.git`中`checkout`出仓库文件到`.git`的父目录中, 然后配置`nginx.conf`中`location`的`root`路径指向`.git`的父目录。

```bash
#!/bin/bash

BRANCH="master"
GIT_DIR="/home/senzyo/git-repos/blog/.git"
TARGET="/home/senzyo/git-repos/blog"

read oldrev newrev ref

if [ "$ref" == "refs/heads/$BRANCH" ]; then
    echo "$ref已成功接收, 正在部署$BRANCH分支"
    git --work-tree="$TARGET" --git-dir="$GIT_DIR" checkout -f "$BRANCH"
    cd $TARGET
    rm -rf public
    hugo --gc --minify
else
    echo "$ref已成功接收, 但只有$BRANCH分支更改时才进行部署"
fi
```
{{< /admonition >}}

1. 参考[Git Config](../2020-7/)为当前用户进行设置。

2. 复制当前用户的公钥, 粘贴到`/home/senzyo/.ssh/authorized_keys`文件里, 如果有多个公钥, 则一行一个。

3. 将`/home/senzyo/git-repos/blog/.git`这个仓库克隆到`/home/senzyo/project`: 

    ```bash
    cd /home/senzyo/project/
    git clone ssh://senzyo@localhost:[port]/home/senzyo/git-repos/blog/.git
    ```

4. 配置仓库: 

    ```bash
    git remote add github git@github.com:senzyo/blog.git
    git remote add gitee git@gitee.com:senzyo/blog.git
    ```
    
5. `git remote -v`, Remote应该包括GitHub、Gitee和服务器中的Remote仓库: 

    ```bash
    gitee   git@gitee.com:senzyo/blog.git (fetch)
    gitee   git@gitee.com:senzyo/blog.git (push)
    github  git@github.com:senzyo/blog.git (fetch)
    github  git@github.com:senzyo/blog.git (push)
    origin  ssh://senzyo@localhost:[port]/home/senzyo/git-repos/blog/.git (fetch)
    origin  ssh://senzyo@localhost:[port]/home/senzyo/git-repos/blog/.git (push)
    ```

6. 注意权限: 

    ```bash
    chmod 700 /home/senzyo/.ssh
    chmod 600 /home/senzyo/.ssh/*
    ```

### 本地电脑中的Git仓库

> 以下是在本地电脑中的操作。

1. 克隆, 并设置仓库: 

    ```bash
    git clone ssh://senzyo@<domain|ip>:[port]/home/senzyo/git-repos/blog/.git
    ```
    
2. 存储、提交和推送: 

    ```bash
    git add -A && git commit -m "测试" && git push -uf origin master
    ```

## 配置Nginx

不同系统中, Nginx配置文件的路径不同, 在Debian中的路径是`/etc/nginx/nginx.conf`。

1. 备份`/etc/nginx/nginx.conf`文件: 

    ```bash
    sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
    ```

2. 编辑`/etc/nginx/nginx.conf`文件: 

    ```bash
    sudo vim /etc/nginx/nginx.conf
    ```

3. 编辑 `/etc/nginx/nginx.conf`, 以下简单设置仅供参考, 推荐使用 DigitalOcean 的 [nginxconfig.io](https://nginxconfig.io) 来生成 Nginx 配置文件: 

    ```nginx
    # Generated by nginxconfig.io
    # See nginxconfig.txt for the configuration share link

    user                 www-data;
    pid                  /run/nginx.pid;
    worker_processes     auto;
    worker_rlimit_nofile 65535;

    # Load modules
    include              /etc/nginx/modules-enabled/*.conf;

    events {
        multi_accept       on;
        worker_connections 65535;
    }

    http {
        charset                utf-8;
        sendfile               on;
        tcp_nopush             on;
        tcp_nodelay            on;
        server_tokens          off;
        log_not_found          off;
        types_hash_max_size    2048;
        types_hash_bucket_size 64;
        client_max_body_size   16M;

        # MIME
        include                mime.types;
        default_type           application/octet-stream;

        # Logging
        log_format main '[$time_iso8601] $remote_addr '
                        '"$request" $status '
                        '"$http_referer" "$http_user_agent"';

        access_log             off;
        error_log              /var/log/nginx/error.log warn;

        # SSL
        ssl_session_timeout    1d;
        ssl_session_cache      shared:SSL:10m;
        ssl_session_tickets    off;

        # Diffie-Hellman parameter for DHE ciphersuites
        ssl_dhparam            /etc/nginx/dhparam.pem;

        # Mozilla Intermediate configuration
        ssl_protocols          TLSv1.2 TLSv1.3;
        ssl_ciphers            ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;

        # OCSP Stapling
        ssl_stapling           on;
        ssl_stapling_verify    on;
        resolver               8.8.8.8 8.8.4.4 208.67.222.222 208.67.220.220 64.6.64.6 64.6.65.6 valid=60s;
        resolver_timeout       2s;

        # Load configs
        include                /etc/nginx/conf.d/*.conf;

        # www.example.com
        server {
            listen                               443 ssl;
            http2                                on;

            server_name                          www.example.com;
            root                                 /var/www/example.com/public;

            # SSL
            ssl_certificate                      /etc/nginx/ssl/example.com/fullchain.cer;
            ssl_certificate_key                  /etc/nginx/ssl/example.com/example.com.key;

            # security headers
            add_header X-XSS-Protection          "1; mode=block" always;
            add_header X-Content-Type-Options    "nosniff" always;
            add_header Referrer-Policy           "no-referrer-when-downgrade" always;
            add_header Content-Security-Policy   "default-src 'self' http: https: ws: wss: data: blob: 'unsafe-inline'; frame-ancestors 'self';" always;
            add_header Permissions-Policy        "interest-cohort=()" always;
            add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

            # . files
            location ~ /\.(?!well-known) {
                deny all;
            }

            # logging
            access_log /var/log/nginx/www.example.com/access.log main buffer=512k flush=1m;
            error_log  /var/log/nginx/www.example.com/error.log warn;

            # favicon.ico
            location = /favicon.ico {
                log_not_found off;
            }

            # robots.txt
            location = /robots.txt {
                log_not_found off;
            }

            # assets, media
            location ~* \.(?:css(\.map)?|js(\.map)?|jpe?g|png|gif|ico|cur|heic|webp|tiff?|mp3|m4a|aac|ogg|midi?|wav|mp4|mov|webm|mpe?g|avi|ogv|flv|wmv)$ {
                expires 7d;
            }

            # svg, fonts
            location ~* \.(?:svgz?|ttf|ttc|otf|eot|woff2?)$ {
                add_header Access-Control-Allow-Origin "*";
                expires    7d;
            }

            # gzip
            gzip            on;
            gzip_vary       on;
            gzip_proxied    any;
            gzip_comp_level 6;
            gzip_types      text/plain text/css text/xml application/json application/javascript application/rss+xml application/atom+xml image/svg+xml;

            error_page      404 /404.html;
        }

        # non-www, subdomains redirect
        server {
            listen              443 ssl;
            http2               on;
            server_name         .example.com;

            # SSL
            ssl_certificate     /etc/nginx/ssl/example.com/fullchain.cer;
            ssl_certificate_key /etc/nginx/ssl/example.com/example.com.key;
            return              301 https://www.example.com$request_uri;
        }

        # HTTP redirect
        server {
            listen      80;
            server_name .example.com;
            return      301 https://www.example.com$request_uri;
        }
    }
    ```

4. 运行命令 `sudo mkdir -p /var/log/nginx/www.example.com` 创建存放日志的文件夹。
5. 运行命令 `sudo openssl dhparam -out /etc/nginx/dhparam.pem 2048` 生成 Diffie-Hellman keys。
6. 至于自动从 Let's Encrypt 获得SSL证书, 相对于 [Certbot](https://certbot.eff.org/), 更推荐使用 [acme.sh](https://github.com/acmesh-official/acme.sh/wiki/说明)。 
7. 更改Nginx服务器的配置文件后, 重新启动或重新加载服务之前测试配置是否存在任何语法或系统错误: 

    ```bash
    sudo nginx -t
    ```

8. 强制重载Nginx配置: 

    ```bash
    sudo service nginx force-reload
    ```

    > 据测试 `sudo service nginx reload` 并不会重新加载证书。

    或者重启Nginx服务: 

    ```bash
    sudo systemctl restart nginx.service
    ```

9. 设置Nginx服务开机自启动: 

    ```bash
    sudo systemctl enable nginx.service
    ```

10. 查看Nginx服务状态: 

    ```bash
    sudo systemctl status nginx.service
    ```

至此, 网站能够正常访问, 且访问`http://example.com/`、`http://www.example.com/`、`https://example.com/`, 都会跳转到`https://www.example.com/`。

--------------------

{{< admonition type=quote title="推荐阅读" open=true >}}
1. [Nginx 官方文档](http://nginx.org/en/docs/)
{{< /admonition >}}