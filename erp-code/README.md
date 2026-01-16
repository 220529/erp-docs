# erp-code 低代码脚本

## 项目概述

存储和管理 ERP 系统的业务流程代码，支持保存自动上传、乐观锁防冲突。

## 工作原理

```
┌─────────────────────────────────────────────────────────────────┐
│                      低代码执行流程                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐        │
│   │  erp-code   │    │  erp-core   │    │   MySQL     │        │
│   │  脚本仓库   │───▶│  执行引擎   │───▶│  数据库     │        │
│   └─────────────┘    └─────────────┘    └─────────────┘        │
│         │                  │                                    │
│         │ 保存时上传       │ 执行时读取                          │
│         ▼                  ▼                                    │
│   ┌─────────────────────────────────────────────────────┐      │
│   │                  code_flows 表                       │      │
│   │  key | name | code | update_time | status           │      │
│   └─────────────────────────────────────────────────────┘      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## 目录结构

```
erp-code/
├── config/                    # 配置文件
│   ├── dev.json              # 开发环境
│   ├── prod.json             # 生产环境
│   └── README.md
├── src/
│   └── flows/                # 业务流程
│       ├── 客户管理/
│       │   └── 创建客户.js
│       └── 订单管理/
│           └── 创建订单.js
├── scripts/
│   └── upload-with-notify.js # 上传脚本
├── docs/
│   └── updateTime机制说明.md
└── README.md
```

## 流程代码格式

```javascript
/**
 * @flowKey customer/create   ← 对应数据库 key 字段
 * @flowName 创建客户
 * @description 创建客户并记录初次跟进
 * @updateTime 2025-10-30 15:00:00
 */

// ============================================
// 1. 解构上下文
// ============================================
const { repositories, params, user } = context;
const { customerRepository } = repositories;

// ============================================
// 2. 参数校验
// ============================================
const { name, phone } = params;

if (!name) {
  throw new Error('客户名称不能为空');
}

// ============================================
// 3. 业务逻辑
// ============================================
const customer = await customerRepository.save({
  name,
  phone,
  createdBy: user.id,
});

// ============================================
// 4. 返回结果
// ============================================
return {
  success: true,
  data: { customerId: customer.id },
  message: '客户创建成功',
};
```

## 可用的 Repository

```javascript
const {
  userRepository,           // 用户
  companyRepository,        // 公司
  departmentRepository,     // 部门
  roleRepository,           // 角色
  customerRepository,       // 客户
  customerFollowRepository, // 客户跟进
  materialRepository,       // 物料
  orderRepository,          // 订单
  orderMaterialRepository,  // 订单物料
  paymentRepository,        // 支付
  productRepository,        // 产品
  fileRepository,           // 文件
  dictRepository,           // 字典
  menuRepository,           // 菜单
  roleMenuRepository,       // 角色菜单
  logRepository,            // 日志
} = repositories;
```

## @updateTime 乐观锁机制

防止多人协作时的代码覆盖冲突：

```
1. 创建时：文件中的 @updateTime 写入数据库
2. 更新时：对比文件时间和数据库时间
   ✅ 一致 → 允许保存，自动更新时间
   ❌ 不一致 → 拒绝保存，提示冲突
3. 成功后：自动更新文件中的 @updateTime
```

冲突示例：
```
╔══════════════════════════════════════════════════════════╗
║                    ❌ 上传失败！                          ║
╚══════════════════════════════════════════════════════════╝
  ⚠️  错误: 更新冲突！代码已被他人修改
文件时间: 2025-10-30 10:00:00
数据库时间: 2025-10-30 10:30:00

💡 请修改文件中的 @updateTime 为: 2025-10-30 10:30:00
```

## 开发流程

### 1. 配置环境
```bash
npm install

# 编辑 config/dev.json
{
  "apiEndpoint": "http://localhost:3000",
  "apiToken": "",
  "apiKey": "your-api-key",
  "dbName": "erp_core"
}
```

### 2. 安装 VSCode 插件
安装 **Run On Save** (emeraldwalk.RunOnSave)

### 3. 开发流程
1. 在 `src/flows/` 下创建或编辑文件
2. 按 `Ctrl+S` 保存
3. 代码自动上传到数据库
4. 在前端测试执行

## 执行流程

```
前端调用 → POST /api/code/execute
         ↓
后端读取 code_flows 表
         ↓
执行 JavaScript 代码
         ↓
注入 context (repositories, params, user)
         ↓
返回执行结果
```

## 最佳实践

1. **命名规范**
   - 文件名：中文，清晰表达业务含义
   - flowKey：英文，斜杠分隔，如 `customer/create`
   - flowName：中文，简洁明了

2. **代码结构**
   - 使用注释分隔业务逻辑块
   - 参数校验放在最前面
   - 返回统一格式的结果

3. **错误处理**
   - 使用 `throw new Error()` 抛出业务错误
   - 错误信息要清晰友好

4. **并发控制**
   - 编辑前先拉取最新代码
   - 遇到冲突时，合并修改后重新保存
