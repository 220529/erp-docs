# erp-web 前端项目

## 项目概述

React 18 + Ant Design 构建的 ERP 管理后台，采用 Monorepo 架构。

## 技术栈

- **框架**: React 18
- **UI 库**: Ant Design 5
- **构建**: Vite 6
- **语言**: TypeScript 5
- **路由**: React Router 7
- **状态**: Zustand
- **请求**: Axios
- **样式**: Less

## 目录结构

```
erp-web/
├── packages/
│   └── main/                   # 主应用
│       ├── src/
│       │   ├── api/           # API 接口
│       │   │   ├── request.ts # Axios 封装
│       │   │   ├── auth.ts    # 认证接口
│       │   │   ├── customer.ts
│       │   │   ├── order.ts
│       │   │   └── ...
│       │   │
│       │   ├── components/    # 公共组件
│       │   │   ├── Layout/    # 布局组件
│       │   │   ├── AuthButton.tsx  # 权限按钮
│       │   │   ├── AuthGuard.tsx   # 权限守卫
│       │   │   └── ListPage.tsx    # 列表页组件
│       │   │
│       │   ├── config/        # 配置
│       │   │   └── microApps.ts
│       │   │
│       │   ├── constants/     # 常量
│       │   │   └── enums.ts   # 枚举定义
│       │   │
│       │   ├── features/      # 业务模块
│       │   │   ├── auth/      # 登录
│       │   │   ├── customer/  # 客户管理
│       │   │   ├── order/     # 订单管理
│       │   │   ├── payment/   # 收款管理
│       │   │   ├── material/  # 材料管理
│       │   │   ├── product/   # 产品管理
│       │   │   ├── system/    # 系统管理
│       │   │   ├── codeflow/  # 低代码管理
│       │   │   └── ...
│       │   │
│       │   ├── hooks/         # 自定义 Hooks
│       │   │   ├── usePermission.ts
│       │   │   └── index.ts
│       │   │
│       │   ├── router/        # 路由配置
│       │   │   ├── index.tsx
│       │   │   └── menu.config.tsx
│       │   │
│       │   ├── styles/        # 样式
│       │   │   ├── global.less
│       │   │   ├── theme.less
│       │   │   └── variables.less
│       │   │
│       │   ├── utils/         # 工具函数
│       │   │   ├── auth.ts
│       │   │   └── format.ts
│       │   │
│       │   ├── App.tsx
│       │   └── main.tsx
│       │
│       ├── index.html
│       ├── package.json
│       ├── tsconfig.json
│       └── vite.config.ts
│
├── package.json
├── pnpm-workspace.yaml
└── tsconfig.json
```

## 功能模块

### 登录认证
- 用户名密码登录
- Token 存储
- 自动刷新
- 登出

### 客户管理
- 客户列表（分页、搜索、筛选）
- 新增/编辑客户
- 客户跟进记录
- 客户导出

### 订单管理
- 订单列表
- 订单详情
- 订单状态变更
- 订单材料管理

### 收款管理
- 收款列表
- 创建收款
- 收款确认

### 系统管理
- 用户管理
- 角色管理
- 菜单管理
- 日志查看

### 低代码管理
- 流程列表
- 流程发布
- 执行测试

## 权限控制

### 菜单权限
```typescript
// 登录后获取菜单，动态生成路由
const { menus } = await login(username, password);
// menus 用于生成侧边栏
```

### 按钮权限
```tsx
// 使用 AuthButton 组件
<AuthButton permission="customer:delete" danger onClick={handleDelete}>
  删除
</AuthButton>

// 使用 usePermission Hook
const { hasPermission } = usePermission();
if (hasPermission('customer:export')) {
  // 显示导出按钮
}
```

## 常用命令

```bash
# 安装依赖
pnpm install

# 开发模式
pnpm dev:main

# 生产构建
pnpm build:main

# 类型检查
pnpm typecheck

# 代码检查
pnpm lint
```

## 环境变量

```bash
# .env.development
VITE_API_BASE_URL=http://localhost:3000/api

# .env.production
VITE_API_BASE_URL=/api
```

## API 请求封装

```typescript
// api/request.ts
import axios from 'axios';

const request = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 10000,
});

// 请求拦截器 - 添加 Token
request.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// 响应拦截器 - 统一错误处理
request.interceptors.response.use(
  (response) => response.data,
  (error) => {
    if (error.response?.status === 401) {
      // Token 过期，跳转登录
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);
```

## 页面开发规范

### 列表页
```tsx
// features/customer/List.tsx
export default function CustomerList() {
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(false);
  const [pagination, setPagination] = useState({ page: 1, pageSize: 10 });

  const fetchData = async () => {
    setLoading(true);
    const res = await customerApi.list(pagination);
    setData(res.data.list);
    setLoading(false);
  };

  return (
    <Table
      dataSource={data}
      loading={loading}
      pagination={pagination}
      columns={columns}
    />
  );
}
```

### 表单页
```tsx
// features/customer/Form.tsx
export default function CustomerForm({ id }: { id?: number }) {
  const [form] = Form.useForm();

  const onFinish = async (values) => {
    if (id) {
      await customerApi.update(id, values);
    } else {
      await customerApi.create(values);
    }
    message.success('保存成功');
  };

  return (
    <Form form={form} onFinish={onFinish}>
      <Form.Item name="name" label="客户姓名" rules={[{ required: true }]}>
        <Input />
      </Form.Item>
      {/* ... */}
    </Form>
  );
}
```

## 部署

### Docker 部署
```bash
docker build -t erp-web .
docker run -d -p 80:80 erp-web
```

### Nginx 配置
```nginx
server {
    listen 80;
    root /usr/share/nginx/html;
    
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    location /api {
        proxy_pass http://erp-core:3000;
    }
}
```
