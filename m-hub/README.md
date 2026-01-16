# m-hub 移动端 H5

## 项目概述

React + antd-mobile 构建的移动端 H5 页面，主要用于营销活动页面。

## 技术栈

- **框架**: React 18
- **UI 库**: antd-mobile 5
- **构建**: Vite 6
- **语言**: TypeScript 5
- **路由**: React Router 7
- **样式**: Less
- **适配**: postcss-px-to-viewport

## 目录结构

```
m-hub/
├── public/                    # 静态资源
│   ├── privacy.html          # 隐私政策
│   └── ...
├── src/
│   ├── pages/                # 页面
│   │   ├── activity-spring/  # 春季活动
│   │   ├── activity-summer/  # 夏季活动
│   │   └── static/           # 静态页面
│   │
│   ├── shared/               # 共享模块
│   │   ├── api/             # API 接口
│   │   │   └── request.ts
│   │   ├── styles/          # 公共样式
│   │   └── utils/           # 工具函数
│   │
│   └── vite-env.d.ts
│
├── index.html
├── package.json
├── tsconfig.json
└── vite.config.ts
```

## 功能模块

### 营销活动页
- 春季促销活动
- 夏季促销活动
- 节日活动页面

### 静态页面
- 隐私政策
- 用户协议
- 关于我们

## 移动端适配

使用 `postcss-px-to-viewport` 实现自动适配：

```javascript
// vite.config.ts
import pxToViewport from 'postcss-px-to-viewport-8-plugin';

export default {
  css: {
    postcss: {
      plugins: [
        pxToViewport({
          viewportWidth: 375,  // 设计稿宽度
          unitPrecision: 5,
          viewportUnit: 'vw',
          minPixelValue: 1,
        }),
      ],
    },
  },
};
```

## 常用命令

```bash
# 安装依赖
pnpm install

# 开发模式
pnpm dev

# 生产构建
pnpm build

# 预览构建结果
pnpm preview
```

## 开发规范

### 页面结构
```
pages/
└── activity-spring/
    ├── index.tsx        # 页面入口
    ├── components/      # 页面组件
    ├── hooks/           # 页面 Hooks
    └── index.less       # 页面样式
```

### 样式规范
```less
// 使用 px 编写，自动转换为 vw
.container {
  padding: 20px;
  font-size: 14px;
}
```

## 部署

### 构建产物
```bash
pnpm build
# 输出到 dist/ 目录
```

### Nginx 配置
```nginx
location /m {
    alias /usr/share/nginx/html/m-hub;
    try_files $uri $uri/ /m/index.html;
}
```

### CI/CD
通过 Git Tag 触发 GitHub Actions 自动部署。
