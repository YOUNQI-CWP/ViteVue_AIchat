# ViteVue AIchat - 基于Vue 3的AI聊天应用

## 项目介绍
ViteVue AIchat是一个基于Vue 3和Vite构建的现代化AI聊天应用。它提供了一个简洁优雅的用户界面，支持与AI模型进行实时对话，并具有代码高亮、Markdown渲染等高级功能。

## 功能特点
- 💬 实时对话：支持与AI模型进行流式对话，响应即时显示
- 🎨 优雅界面：使用Element Plus组件库，提供现代化的用户界面
- ✨ 代码高亮：自动识别并高亮显示代码块，支持多种编程语言
- 📝 Markdown支持：AI回复支持Markdown格式，包括代码块、列表等
- 🔄 消息重新生成：支持重新生成AI的回复消息
- 📋 消息复制：一键复制对话内容
- ⚙️ 灵活配置：支持自定义API密钥、接口地址和模型选择

## 技术栈
- Vue 3 - 渐进式JavaScript框架
- Vite - 下一代前端构建工具
- Element Plus - 基于Vue 3的组件库
- Marked - Markdown解析器
- Highlight.js - 代码语法高亮
- Axios - HTTP客户端

## 快速开始

### 环境要求
- Node.js 16.0或更高版本
- npm或yarn包管理器

### 安装步骤
1. 克隆项目到本地：
```bash
git clone <repository-url>
cd ViteVue_AIchat
```

2. 安装依赖：
```bash
npm install
# 或
yarn install
```

3. 启动开发服务器：
```bash
npm run dev
# 或
yarn dev
```

应用将在 http://localhost:3000 启动

### API配置
首次使用时，需要配置API设置：
1. 点击右上角的设置图标
2. 填写以下信息：
   - API密钥：您的API访问密钥
   - API地址：AI服务的接口地址
   - 选择模型：选择要使用的AI模型
3. 点击保存

## 使用说明
1. 发送消息
   - 在底部输入框输入消息
   - 按回车键或点击发送按钮

2. 消息操作
   - 复制消息：点击消息右侧的复制图标
   - 重新生成：点击AI回复右侧的刷新图标

3. 设置修改
   - 点击右上角设置图标
   - 修改API配置
   - 保存更改

## 开发说明
- 开发模式：`npm run dev`
- 构建项目：`npm run build`
- 预览构建：`npm run preview`

## 注意事项
- API密钥请妥善保管，不要泄露给他人
- 建议使用现代浏览器访问以获得最佳体验
- 如遇到问题，请检查API配置是否正确

## Vercel部署教程

### 准备工作
1. 注册Vercel账号
   - 访问 [Vercel官网](https://vercel.com)
   - 使用GitHub账号或邮箱注册

2. 安装Vercel CLI（可选）
```bash
npm install -g vercel
```

### 部署步骤
1. 导入项目
   - 登录Vercel控制台
   - 点击「New Project」
   - 选择并导入GitHub仓库
   - 选择要部署的分支（通常是main或master）

2. 配置项目
   - 构建命令：`npm run build`
   - 输出目录：`dist`
   - Node.js版本：选择16.x或更高版本

3. 环境变量设置
   - 在项目设置中找到「Environment Variables」
   - 添加必要的环境变量（如API密钥等）

4. 部署项目
   - 点击「Deploy」开始部署
   - 等待构建和部署完成
   - 访问分配的域名查看部署结果

### 自定义域名
1. 添加域名
   - 在项目设置中选择「Domains」
   - 输入要绑定的域名
   - 按照提示配置DNS记录

2. SSL/HTTPS配置
   - Vercel自动提供SSL证书
   - 确保DNS记录正确配置

### 自动部署
- 当推送代码到GitHub仓库时，Vercel会自动触发新的部署
- 可以在「Git」设置中配置自动部署的分支和行为

### 常见问题
1. 构建失败
   - 检查依赖是否完整
   - 确认Node.js版本兼容性
   - 查看构建日志定位错误

2. 环境变量问题
   - 确保所有必要的环境变量都已配置
   - 检查环境变量名称是否正确

3. 域名配置问题
   - 等待DNS记录生效（通常需要几分钟到几小时）
   - 确认DNS记录配置正确

## 许可证
本项目基于MIT许可证开源
