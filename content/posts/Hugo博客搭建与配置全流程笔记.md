---
title: "Hugo博客搭建与配置全流程笔记"
date: 2026-02-15T16:13:48+08:00
draft: false
tags: ["Hugo", "GitHub Pages", "VSCode", "静态博客", "技术笔记"]
categories: ["技术文档"]
summary: "详细记录使用Hugo+GitHub Pages搭建个人技术博客的全过程，包括环境配置、VSCode插件设置、自动化工作流和问题解决方案。"
authors: ["faraway1913"]
---

## 引言

作为一名嵌入式软件工程师，技术沉淀和知识复盘是提升专业能力的重要途径。本文记录我搭建个人技术博客的完整过程，主要技术栈为 **Hugo** + **GitHub Pages** + **PaperMod主题**，配合 **VSCode** 实现高效的写作发布工作流。

本文适合有一定Git和Markdown基础的技术人员参考，特别是非前端背景的开发者。

<!--more-->

## 技术栈概览

| 组件 | 选择 | 说明 |
|------|------|------|
| **静态网站生成器** | Hugo v0.155.3 | Go语言编写，构建速度快，单二进制文件 |
| **主题** | PaperMod | 响应式设计，功能丰富，适合技术博客 |
| **托管平台** | GitHub Pages | 免费静态网站托管，集成Git工作流 |
| **编辑器** | Visual Studio Code | 强大的Markdown编辑和插件生态 |
| **版本控制** | Git + GitHub | 代码和内容版本管理 |
| **图片管理** | Paste Image插件 | 自动化图片保存和Markdown引用 |

## 环境配置

### 1. Hugo安装与配置

#### 下载与安装
```bash
# Windows系统下载Hugo扩展版
# 下载地址：https://github.com/gohugoio/hugo/releases
# 选择 hugo_extended_x.xx.x_windows-amd64.zip

# 解压到 D:\APPS\hugo_0.155.3_windows-amd64\
# 添加系统PATH环境变量：D:\APPS\hugo_0.155.3_windows-amd64
```

#### 验证安装
```bash
hugo version
# 输出：hugo v0.155.3... windows/amd64 BuildDate=...
```

### 2. 项目初始化

#### 创建本地项目
```bash
# 创建项目目录
mkdir 个人博客
cd 个人博客

# 初始化Hugo项目
hugo new site . --force
```

#### 添加PaperMod主题
```bash
# 初始化Git仓库
git init

# 添加PaperMod主题作为Git子模块
git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

### 3. GitHub仓库创建

#### 命名规则
- **个人站点**：`用户名.github.io` (如 `faraway1913.github.io`)
- **项目站点**：任意名称，访问地址为 `用户名.github.io/仓库名`

#### 创建仓库
1. 访问 https://github.com/new
2. 仓库名：`faraway1913.github.io`
3. 设置为 Public（公开）
4. 不初始化README等文件

#### 配置远程仓库
```bash
git remote add origin https://github.com/faraway1913/faraway1913.github.io.git
# 或使用SSH方式
git remote add origin git@github.com:faraway1913/faraway1913.github.io.git
```

## 配置文件详解

### 1. 主配置文件 (config.yaml)

```yaml
baseURL: "https://faraway1913.github.io/"
title: "技术笔记与复盘"
theme: "PaperMod"

# 多语言配置
defaultContentLanguage: "zh"
defaultContentLanguageInSubdir: false
hasCJKLanguage: true

# 网站参数
params:
  env: "production"
  title: "技术笔记与复盘"
  description: "嵌入式开发笔记、技术复盘与日常记录"
  keywords: ["嵌入式", "C语言", "单片机", "RTOS", "电子技术"]

  # 主页信息
  homeInfoParams:
    Title: "欢迎来到我的技术博客"
    Content: |
      这里记录我的嵌入式开发笔记、项目复盘和技术思考。

      主要领域：
      - 嵌入式C语言开发
      - STM32/ESP32等单片机应用
      - FreeRTOS/RT-Thread等实时操作系统
      - 电子电路设计与调试
      - 技术学习与成长复盘

# 分页设置（Hugo v0.128.0+）
pagination:
  pagerSize: 10
  path: "page"

# 菜单配置
menu:
  main:
    - identifier: "posts"
      name: "文章"
      url: "/posts/"
      weight: 1
    - identifier: "tags"
      name: "标签"
      url: "/tags/"
      weight: 2
    - identifier: "categories"
      name: "分类"
      url: "/categories/"
      weight: 3
    - identifier: "about"
      name: "关于"
      url: "/about/"
      weight: 4
```

### 2. GitHub Actions工作流 (.github/workflows/hugo.yaml)

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

env:
  HUGO_VERSION: "0.155.3"

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: ${{ env.HUGO_VERSION }}
          extended: true

      - name: Build
        run: hugo --minify

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

## VSCode环境配置

### 1. 推荐插件列表

| 插件名称 | 作者 | 功能 |
|----------|------|------|
| **Paste Image** | mushan | 截图自动保存并插入Markdown引用 |
| **Markdown All in One** | yzhang | Markdown编辑增强 |
| **markdownlint** | David Anson | Markdown语法检查 |
| **Code Spell Checker** | Street Side Software | 代码拼写检查 |
| **Hugo Language and Syntax Support** | budparr | Hugo语法支持 |

### 2. 工作区配置 (.vscode/settings.json)

```json
{
  // Paste Image 插件配置
  "pasteImage.path": "${projectRoot}/static/images/posts",
  "pasteImage.prefix": "/images/posts/",
  "pasteImage.encodePath": "none",
  "pasteImage.forceUnixStyleSeparator": true,
  "pasteImage.insertPattern": "![${imageFileNameWithoutExt}](${imageFilePath})",
  "pasteImage.namePrefix": "${currentFileNameWithoutExt}-",
  "pasteImage.defaultName": "YYYY-MM-DD-HH-mm-ss",

  // Markdown相关配置
  "markdown.preview.breaks": true,
  "markdown.preview.fontSize": 14,
  "markdown.preview.scrollEditorWithPreview": true,
  "markdown.preview.scrollPreviewWithEditor": true,

  // 文件排除配置
  "files.exclude": {
    "**/.git": true,
    "**/.DS_Store": true,
    "public/": true,
    "resources/_gen/": true
  }
}
```

### 3. 任务配置 (.vscode/tasks.json)

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "启动Hugo本地服务器",
      "type": "shell",
      "command": "/d/APPS/hugo_0.155.3_windows-amd64/hugo.exe server -D",
      "group": "build",
      "detail": "启动Hugo本地开发服务器，包含草稿文章"
    },
    {
      "label": "构建静态网站",
      "type": "shell",
      "command": "/d/APPS/hugo_0.155.3_windows-amd64/hugo.exe",
      "group": { "kind": "build", "isDefault": true },
      "detail": "生成静态网站到public目录"
    }
  ]
}
```

## 完整工作流程

### 1. 创建新文章

```bash
# 使用Hugo命令创建文章模板
hugo new posts/STM32-GPIO配置详解.md

# 文件位置：content/posts/STM32-GPIO配置详解.md
# 自动生成Front Matter：
# ---
# title: "STM32-GPIO配置详解"
# date: 2026-02-15T16:30:00+08:00
# draft: true
# tags: []
# categories: []
# summary: ""
# authors: []
# ---
```

### 2. 编辑文章内容

在VSCode中打开文章文件，编写Markdown内容：

```markdown
## GPIO工作模式详解

STM32的GPIO支持8种工作模式：

| 模式 | 描述 | 应用场景 |
|------|------|----------|
| 输入浮空 | 高阻抗输入 | 按键检测 |
| 输入上拉 | 内部上拉电阻 | 节省外部元件 |
| 输入下拉 | 内部下拉电阻 | 节省外部元件 |
| 模拟输入 | ADC采集 | 模拟信号输入 |

### 代码示例

```c
// GPIO初始化示例
void GPIO_Init(void) {
    GPIO_InitTypeDef GPIO_InitStruct = {0};

    // 配置PC13为推挽输出
    __HAL_RCC_GPIOC_CLK_ENABLE();
    GPIO_InitStruct.Pin = GPIO_PIN_13;
    GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
    GPIO_InitStruct.Pull = GPIO_NOPULL;
    GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
    HAL_GPIO_Init(GPIOC, &GPIO_InitStruct);
}
```

### 插入图片

1. 截图（推荐使用Snipaste，按F1截图）
2. 在VSCode编辑器中按 `Ctrl+Alt+V`
3. 自动完成：
   - 图片保存到：`static/images/posts/STM32-GPIO配置详解-2026-02-15-16-35-45.png`
   - Markdown插入：`![GPIO结构图](/images/posts/STM32-GPIO配置详解-2026-02-15-16-35-45.png)`
```

### 3. 本地预览

```bash
# 启动Hugo本地服务器
hugo server -D

# 访问 http://localhost:1313
# 编辑保存后自动刷新页面
```

### 4. 发布文章

```bash
# 1. 将文章状态改为发布
# 编辑文章Front Matter：draft: true → draft: false

# 2. 提交到Git
git add .
git commit -m "发布新文章：STM32-GPIO配置详解"

# 3. 推送到GitHub
git push origin main

# 4. 等待GitHub Actions自动构建（约1-2分钟）
# 5. 访问 https://faraway1913.github.io
```

## 图片管理策略

### 自动化图片处理
- **保存位置**：`static/images/posts/`
- **命名规则**：`文章名-年-月-日-时-分-秒.扩展名`
- **引用格式**：`![描述](/images/posts/文件名.png)`
- **版本控制**：图片随代码一起提交到Git

### 图片目录结构
```
static/images/
└── posts/           # 文章图片
    ├── STM32-GPIO配置详解-2026-02-15-16-35-45.png
    ├── FreeRTOS任务调度-2026-02-16-09-20-30.jpg
    └── 电路设计-2026-02-17-14-15-20.png
```

## 遇到的问题与解决方案

### 1. Hugo命令找不到
**问题**：`hugo: command not found`
**原因**：Hugo未添加到系统PATH
**解决**：
```bash
# 临时添加PATH
export PATH=$PATH:/d/APPS/hugo_0.155.3_windows-amd64

# 或永久添加到.bashrc
echo 'export PATH=$PATH:/d/APPS/hugo_0.155.3_windows-amd64' >> ~/.bashrc
source ~/.bashrc
```

### 2. GitHub Pages 404错误
**问题**：`https://faraway1913.github.io` 显示404
**原因**：
1. 仓库命名错误（应为 `用户名.github.io`）
2. GitHub Actions构建失败
3. 首次部署需要等待缓存更新
**解决**：
1. 检查仓库命名
2. 查看GitHub Actions构建日志
3. 手动重新运行工作流
4. 等待10-15分钟缓存更新

### 3. 图片预览不显示
**问题**：Markdown Preview Enhanced插件无法显示图片
**原因**：插件使用文件系统路径，Hugo使用HTTP服务器路径
**解决**：
- **推荐方案**：使用Hugo本地服务器预览（100%准确）
- **备选方案**：配置插件路径映射

### 4. Paste Image插件不工作
**问题**：`Ctrl+Alt+V` 提示 "There is not a image in clipboard"
**原因**：截图未正确保存到剪贴板
**解决**：
1. 使用可靠截图工具：Snipaste（F1截图）
2. 确保截图后点击"完成"保存到剪贴板
3. 使用 `Win+Shift+S`（Windows自带截图）

## 最佳实践建议

### 1. 写作规范
- **Front Matter完整**：填写title、date、tags、categories、summary
- **代码高亮**：使用正确的语言标记（如 ````c`）
- **图片描述**：为每张图片添加有意义的描述文字
- **标签分类**：合理使用标签便于内容组织

### 2. 版本控制
- **提交规范**：每次提交有明确的描述信息
- **分支策略**：main分支用于发布，feature分支用于开发
- **备份重要**：定期备份 `content/` 目录

### 3. 性能优化
- **图片压缩**：使用TinyPNG等工具压缩图片
- **代码分割**：长文章使用 `<!--more-->` 分隔摘要
- **缓存利用**：Hugo内置缓存机制，充分利用

## 总结

通过 Hugo + GitHub Pages + VSCode 的组合，我搭建了一个完全免费、自主可控、高效易用的技术博客系统。主要优势包括：

1. **零成本**：GitHub Pages提供免费托管
2. **高性能**：Hugo静态生成，访问速度快
3. **版本管理**：所有内容通过Git管理，永不丢失
4. **写作高效**：VSCode插件生态提供强大编辑能力
5. **部署自动化**：GitHub Actions自动构建发布

对于技术背景的写作者，这套方案特别适合：
- 嵌入式开发者的技术笔记
- 项目复盘和经验总结
- 学习记录和知识沉淀
- 技术分享和交流

**在线博客**：[https://faraway1913.github.io](https://faraway1913.github.io)
**源码仓库**：[https://github.com/faraway1913/faraway1913.github.io](https://github.com/faraway1913/faraway1913.github.io)

---

**最后更新**：2026-02-15 16:30