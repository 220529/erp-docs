# nginx-app 网关服务

## 项目概述

Nginx 反向代理服务，负责请求转发、SSL 终结、静态资源服务。

## 架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                        nginx-app                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │                    Nginx                                 │  │
│   │                    :80 / :443                            │  │
│   └─────────────────────────────────────────────────────────┘  │
│                             │                                   │
│         ┌───────────────────┼───────────────────┐              │
│         │                   │                   │              │
│         ▼                   ▼                   ▼              │
│   ┌───────────┐       ┌───────────┐       ┌───────────┐       │
│   │  /        │       │  /api     │       │  /m       │       │
│   │  erp-web  │       │  erp-core │       │  m-hub    │       │
│   │  静态文件 │       │  反向代理 │       │  静态文件 │       │
│   └───────────┘       └───────────┘       └───────────┘       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 目录结构

```
nginx-app/
├── conf.d/                    # 配置文件
│   ├── nginx-ssl.conf        # SSL 配置
│   └── default.conf          # 默认配置
├── scripts/                   # 脚本
│   ├── init-ssl.sh           # SSL 初始化
│   └── deploy.bat            # 部署脚本
├── docs/                      # 文档
├── nginx.conf                 # 主配置
├── docker-compose-prod.yml   # 生产配置
└── README.md
```

## 路由配置

| 路径 | 目标 | 说明 |
|-----|------|-----|
| / | erp-web | PC 前端 |
| /api | erp-core:3000 | 后端 API |
| /m | m-hub | 移动端 H5 |

## Nginx 配置示例

```nginx
server {
    listen 80;
    listen 443 ssl;
    server_name erp.example.com;

    # SSL 证书
    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;

    # PC 前端
    location / {
        root /usr/share/nginx/html/erp-web;
        try_files $uri $uri/ /index.html;
    }

    # 后端 API
    location /api {
        proxy_pass http://erp-core:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # 移动端 H5
    location /m {
        alias /usr/share/nginx/html/m-hub;
        try_files $uri $uri/ /m/index.html;
    }

    # 静态资源缓存
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
}
```

## 部署

### 通过 Git Tag 部署
```bash
.\scripts\deploy.bat
# 或
git tag v1.0.0
git push origin v1.0.0
```

### 手动部署
```bash
docker-compose -f docker-compose-prod.yml up -d
```

## SSL 证书

### 使用 Let's Encrypt
```bash
# 初始化证书
./scripts/init-ssl.sh

# 证书自动续期（crontab）
0 0 1 * * certbot renew --quiet
```

### 证书路径
```
/etc/nginx/ssl/
├── fullchain.pem    # 证书链
└── privkey.pem      # 私钥
```

## 常用命令

```bash
# 测试配置
nginx -t

# 重载配置
nginx -s reload

# 查看日志
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log
```

## 性能优化

```nginx
# nginx.conf
worker_processes auto;
worker_connections 1024;

# Gzip 压缩
gzip on;
gzip_types text/plain text/css application/json application/javascript;
gzip_min_length 1000;

# 缓冲区
proxy_buffer_size 128k;
proxy_buffers 4 256k;
proxy_busy_buffers_size 256k;
```
