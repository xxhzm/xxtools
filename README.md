# 小小在线工具箱

一个基于 Vue 3 + Nuxt.js 开发的在线工具集合网站，包含图片处理、开发工具、格式转换等多种实用功能。

## 功能特点

- 🛠 多种实用工具
- 🎨 美观的 UI 设计
- 📱 响应式布局
- 🔒 本地运算，保护隐私

## 部署方法

### 环境要求

- Node.js >= 18.17.0
- npm >= 10.5.0

### 安装步骤

1. 克隆项目

```bash
git clone [项目地址]
cd [项目目录]
```

2. 安装依赖

```bash
npm install
```

3. 开发环境运行

```bash
npm run dev
```

### 部署方式

#### 1. Node.js 服务器

```bash
# 构建项目
npm run build

# 启动服务
node .output/server/index.mjs
```

#### 2. PM2 部署（推荐）

```bash
# 构建项目
npm run build

# 使用PM2启动
pm2 start .output/server/index.mjs --name xxtools

# 查看运行状态
pm2 status

# 查看日志
pm2 logs xxtools

# 重启服务
pm2 restart xxtools

# 停止服务
pm2 stop xxtools
```

## 技术栈

- Vue 3
- Nuxt.js
- Element Plus
- TypeScript

## 贡献指南

1. Fork 项目
2. 创建功能分支
3. 提交代码
4. 创建 Pull Request

## 许可证

MIT License

## 联系方式

如有问题或建议，请通过以下方式联系：

- 提交 Issue
- [项目地址]
