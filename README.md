<div align="center">

  # 一个拖延症

  一个基于 Chirpy 主题的技术博客网站，用于分享编程、技术和个人经验。

  [![GitHub license](https://img.shields.io/github/license/cotes2020/jekyll-theme-chirpy.svg)](https://github.com/cotes2020/jekyll-theme-chirpy/blob/master/LICENSE)

  [**访问网站 →**](https://jkfdy.scfhao.cn)

</div>

## 项目介绍

这是一个基于 Jekyll 和 Chirpy 主题构建的个人技术博客网站，专注于分享编程知识、技术教程和个人经验。网站内容涵盖 C 语言编程、Git 版本控制、AI 大模型等多个技术领域。

## 功能特点

- 🌙 深色/浅色主题模式切换
- 🌍 多语言支持（默认中文）
- 📌 固定帖子功能
- 📁 分层分类系统
- 🏷️ 趋势标签展示
- 📑 文章目录导航
- 📅 帖子最后修改日期
- ✨ 代码语法高亮
- 📊 Mermaid 图表和流程图
- 🎥 嵌入视频支持
- 💬 Cusdis 评论系统
- 🔍 站内搜索功能
- 📡 Atom 订阅源
- 📈 Google Analytics 统计
- 📱 响应式设计，支持移动端
- 🚀 PWA 支持，可离线访问

## 技术栈

- **静态站点生成器**: Jekyll
- **前端框架**: Bootstrap
- **图标库**: Font Awesome
- **样式预处理**: SCSS
- **JavaScript 构建**: Rollup
- **评论系统**: Cusdis
- **部署平台**: GitHub Pages

## 项目结构

```
├── _data/          # 数据文件（多语言支持、配置等）
├── _includes/      # 布局组件
├── _javascript/    # JavaScript 源代码
├── _layouts/       # 页面布局
├── _plugins/       # Jekyll 插件
├── _posts/         # 博客文章
├── _sass/          # SCSS 样式文件
├── _tabs/          # 标签页
├── assets/         # 静态资源
├── tools/          # 工具脚本
├── _config.yml     # 网站配置
├── package.json    # 项目依赖和脚本
└── README.md       # 项目说明
```

## 本地开发

### 环境要求

- Ruby 2.7 或更高版本
- Node.js 14 或更高版本
- Bundler

### 安装依赖

```bash
# 安装 Ruby 依赖
bundle install

# 安装 Node.js 依赖
npm install
```

### 开发服务器

```bash
# 构建 JavaScript 资源
npm run build

# 启动 Jekyll 开发服务器
bundle exec jekyll serve
```

访问 `http://localhost:4000` 查看网站。

### 构建生产版本

```bash
# 构建 JavaScript 资源
npm run build

# 构建静态站点
bundle exec jekyll build
```

构建产物将生成在 `_site` 目录中。

## 内容管理

### 发布新文章

在 `_posts` 目录中创建新的 Markdown 文件，文件名格式为 `YYYY-MM-DD-title.md`。

文章头部需要包含以下 Front Matter：

```yaml
---
title: 文章标题
date: 2024-01-01 00:00:00 +0800
categories: [分类1, 分类2]
tags: [标签1, 标签2]
---
```

### 自定义页面

在 `_tabs` 目录中创建 Markdown 文件，用于添加自定义页面（如关于、归档等）。

## 配置说明

主要配置文件为 `_config.yml`，可修改以下内容：

- 网站基本信息（标题、描述、URL 等）
- 社交链接
- Google Analytics 设置
- 评论系统配置
- 主题设置

## 贡献

欢迎提交 Issue 和 Pull Request 来改进这个项目。

## 许可证

本项目基于 [MIT](https://github.com/cotes2020/jekyll-theme-chirpy/blob/master/LICENSE) 许可证开源。

## 致谢

- 感谢 [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) 主题的开发者
- 感谢所有开源工具和库的贡献者
