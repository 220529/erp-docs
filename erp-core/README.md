# erp-core 后端服务

## 项目概述

NestJS 11 构建的 RESTful API 服务，提供 ERP 系统的所有后端功能。

## 技术栈

- **框架**: NestJS 11
- **语言**: TypeScript 5
- **ORM**: TypeORM 0.3
- **数据库**: MySQL 8
- **缓存**: Redis 7
- **认证**: JWT + Passport
- **文档**: Swagger
- **定时任务**: @nestjs/schedule
- **文件存储**: 阿里云 OSS

## 目录结构

```
erp-core/
├── src/
│   ├── common/                 # 公共模块
│   │   ├── constants/         # 常量定义
│   │   ├── decorators/        # 自定义装饰器
│   │   ├── dto/               # 公共 DTO
│   │   ├── entities/          # 基础实体
│   │   ├── exceptions/        # 自定义异常
│   │   ├── filters/           # 异常过滤器
│   │   ├── guards/            # 守卫
│   │   ├── interceptors/      # 拦截器
│   │   ├── pipes/             # 管道
│   │   └── services/          # 公共服务
│   │
│   ├── config/                # 配置模块
│   │   ├── config.module.ts
│   │   └── configuration.ts
│   │
│   ├── database/              # 数据库模块
│   │   ├── data-source.ts     # TypeORM 数据源
│   │   ├── database.module.ts
│   │   └── seeder.service.ts  # 数据初始化
│   │
│   ├── entities/              # 实体定义
│   │   ├── customer.entity.ts
│   │   ├── order.entity.ts
│   │   ├── payment.entity.ts
│   │   ├── user.entity.ts
│   │   └── ...
│   │
│   ├── migrations/            # 数据库迁移
│   │
│   ├── modules/               # 业务模块
│   │   ├── auth/             # 认证模块
│   │   ├── customers/        # 客户模块
│   │   ├── orders/           # 订单模块
│   │   ├── payments/         # 收款模块
│   │   ├── materials/        # 材料模块
│   │   ├── products/         # 产品模块
│   │   ├── system/           # 系统模块
│   │   ├── permission/       # 权限模块
│   │   ├── code/             # 低代码模块
│   │   ├── scheduler/        # 定时任务模块
│   │   └── ...
│   │
│   ├── app.module.ts          # 根模块
│   └── main.ts                # 入口文件
│
├── test/                       # 测试文件
├── .env                        # 环境变量
├── .env.example               # 环境变量示例
├── package.json
└── tsconfig.json
```

## 模块说明

### 认证模块 (auth)
- 用户登录/注册
- JWT Token 签发
- Token 刷新
- 密码加密

### 客户模块 (customers)
- 客户 CRUD
- 客户跟进记录
- 客户状态管理
- 客户导出

### 订单模块 (orders)
- 订单 CRUD
- 订单状态流转
- 订单材料管理
- 订单统计

### 收款模块 (payments)
- 收款记录
- 收款确认
- 收款统计

### 权限模块 (permission)
- 菜单管理
- 角色管理
- 权限分配
- 权限校验

### 低代码模块 (code)
- 流程管理
- 代码执行
- 乐观锁控制

### 定时任务模块 (scheduler)
- 任务配置
- 任务执行
- 执行日志

## 环境变量

```bash
# 数据库
DB_HOST=localhost
DB_PORT=3306
DB_USERNAME=root
DB_PASSWORD=root
DB_DATABASE=erp_core

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=redis123

# JWT
JWT_SECRET=your-secret-key
JWT_EXPIRES_IN=7d

# OSS
OSS_REGION=oss-cn-hangzhou
OSS_ACCESS_KEY_ID=xxx
OSS_ACCESS_KEY_SECRET=xxx
OSS_BUCKET=xxx
```

## 常用命令

```bash
# 安装依赖
pnpm install

# 开发模式
pnpm start:dev

# 生产构建
pnpm build

# 生产运行
pnpm start:prod

# 生成迁移
pnpm migration:generate

# 运行迁移
pnpm migration:run

# 回滚迁移
pnpm migration:revert
```

## API 文档

启动服务后访问：http://localhost:3000/api/docs

## 请求/响应格式

### 请求头
```
Authorization: Bearer <token>
Content-Type: application/json
```

### 成功响应
```json
{
  "code": 0,
  "message": "success",
  "data": { ... }
}
```

### 错误响应
```json
{
  "code": 400,
  "message": "错误信息",
  "error": "Bad Request"
}
```

### 分页响应
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "list": [...],
    "total": 100,
    "page": 1,
    "pageSize": 10
  }
}
```

## 核心功能

### 认证流程
```
1. POST /api/auth/login → 获取 token
2. 请求头携带 Authorization: Bearer <token>
3. JwtGuard 校验 token
4. PermissionGuard 校验权限
5. 执行业务逻辑
```

### 权限控制
```typescript
// Controller 中使用
@UseGuards(JwtAuthGuard, PermissionGuard)
@RequirePermission('customer:delete')
@Delete(':id')
async delete(@Param('id') id: number) {
  // ...
}
```

### 操作日志
```typescript
// 自动记录操作日志
@OperationLog({
  module: '客户管理',
  type: 'create',
  description: '创建客户',
})
@Post()
async create(@Body() dto: CreateCustomerDto) {
  // ...
}
```

## 部署

### Docker 部署
```bash
docker build -t erp-core .
docker run -d -p 3000:3000 --env-file .env.prod erp-core
```

### CI/CD
通过 Git Tag 触发 GitHub Actions 自动部署：
```bash
git tag v1.0.0
git push origin v1.0.0
```
