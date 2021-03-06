# Docker Nginx 反向代理

## 1.创建Nginx和要被反向代理的服务

Nginx对应宿主机8080端口, 将conf.d文件夹挂载到宿主机

```shell
docker run -d \
--name nginx_80 \
-p 8080:80 \
-v /root/nginx_80/conf.d:/etc/nginx/conf.d \
nginx
```

如果有需要,可以把Nginx以下文件夹都挂载到宿主机

- /etc/nginx/conf.d
- /etc/nginx/nginx.conf
- /var/log/nginx
- /usr/share/nginx/html



对应命令:

```shell
docker run -d \
-p 8001:2368 \
--name ghost_if404 \
-v /root/ghost_if404/content/:/var/lib/ghost/content/ \
-v /root/ghost_if404/config.production.json:/var/lib/ghost/config.production.json \
ghost
```



创建你想要代理的网络应用(例如wordpress, php, golang web, python web, java)

## 2.配置Nginx反向代理

进入宿主机的/root/nginx_80/conf.d, 创建www.test.com.conf命令搭建配置文件

```nginx
server {
listen  80;
    server_name  www.if404.com; //域名
    access_log /var/log/nginx/if404.access.log main;
    error_log /var/log/nginx/if404.error.log error;
    location / {
        proxy_set_header  Host  $http_host;
        proxy_set_header  X-Real-IP  $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass  http://宿主机IP:web应用端口;
    }
}
```

# 负载均衡

```nginx
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
    upstream server {
        // 注意使用局域网或者 ip 不能使用 127.0.0.1 或者 localhost
        server 172.19.207.192:9222;
        server 172.19.207.192:9223;
    }

    server {
        listen 80;
        server_name 172.19.207.192;
        location / {
            proxy_pass http://server;
            root html;
            index index.html index.htm;
        }
    }
}

```

