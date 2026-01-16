# db-app 数据库服务

## 项目概述

使用 Docker Compose 管理 MySQL 和 Redis 服务，支持本地开发和生产部署。

## 服务架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        db-app 服务                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────────┐         ┌─────────────────┐              │
│   │     MySQL 8     │         │    Redis 7      │              │
│   │    主数据库     │         │   缓存/会话     │              │
│   │    :3306        │         │    :6379        │              │
│   └─────────────────┘         └─────────────────┘              │
│           │                           │                         │
│           ▼                           ▼                         │
│   ┌─────────────────┐         ┌─────────────────┐              │
│   │   mysql_data    │         │   redis_data    │              │
│   │   数据持久化    │         │   数据持久化    │              │
│   └─────────────────┘         └─────────────────┘              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 目录结构

```
db-app/
├── init-scripts/              # 初始化脚本
│   └── 01-create-databases.sql
├── mysql/                     # MySQL 数据目录
├── redis_data/                # Redis 数据目录
├── backups/                   # 备份目录
├── scripts/                   # 运维脚本
│   ├── backup.sh
│   └── restore.sh
├── docker-compose.yml         # 开发环境
├── docker-compose.prod.yml    # 生产环境
└── README.md
```

## 本地开发

### 启动服务
```bash
docker-compose up -d
```

### 停止服务
```bash
docker-compose down
```

### 查看状态
```bash
docker-compose ps
```

### 查看日志
```bash
docker-compose logs -f mysql
docker-compose logs -f redis
```

## 服务信息

### 开发环境

| 服务 | 地址 | 账号密码 |
|-----|------|---------|
| MySQL | localhost:3306 | root / root |
| Redis | localhost:6379 | redis123 |

### 自动创建的数据库

| 数据库 | 说明 |
|-------|------|
| erp_core | ERP 核心数据库 |
| erp_test | 测试数据库 |

### 自动创建的用户

| 用户 | 密码 | 权限 |
|-----|------|-----|
| erp_user | erp_password_123 | erp_core 全部权限 |

## 数据库操作

### 连接 MySQL
```bash
# 命令行
docker exec -it dev-mysql mysql -uroot -proot

# 或使用客户端工具
Host: localhost
Port: 3306
User: root
Password: root
```

### 连接 Redis
```bash
docker exec -it dev-redis redis-cli -a redis123
```

### 备份数据库
```bash
docker exec dev-mysql mysqldump -uroot -proot erp_core > backup.sql
```

### 恢复数据库
```bash
docker exec -i dev-mysql mysql -uroot -proot erp_core < backup.sql
```

## 生产部署

### 通过 Git Tag 部署
```bash
git tag v1.0.0
git push origin v1.0.0
```

### 手动部署
```bash
docker-compose -f docker-compose.prod.yml --env-file .env.prod up -d
```

## 添加新数据库

编辑 `init-scripts/01-create-databases.sql`:

```sql
CREATE DATABASE IF NOT EXISTS `your_db`
  CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

CREATE USER IF NOT EXISTS 'your_user'@'%' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON `your_db`.* TO 'your_user'@'%';
FLUSH PRIVILEGES;
```

重建容器:
```bash
docker-compose down -v
docker-compose up -d
```

## 常见问题

### 容器启动失败
```bash
# 查看日志
docker-compose logs mysql

# 常见原因：端口被占用
netstat -ano | findstr :3306
```

### 数据丢失
```bash
# 检查数据卷
docker volume ls

# 数据目录
./mysql/       # MySQL 数据
./redis_data/  # Redis 数据
```

### 性能优化
```yaml
# docker-compose.yml
mysql:
  command:
    - --max_connections=500
    - --innodb_buffer_pool_size=256M
```
