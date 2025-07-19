# Readme

> 📚 一个基于 **MkDocs** 和 **Material for MkDocs** 的个人技术文档站点，自动部署在 [GitHub Pages](https://pages.github.com/) 上。

---

## 项目简介

本项目用于整理、编写、发布个人学习与工作中积累的笔记与技术文档，使用 [**MkDocs**](https://www.mkdocs.org/) 作为静态网站生成器，配合 [**Material for MkDocs**](https://squidfunk.github.io/mkdocs-material/) 主题，自动渲染 Markdown 文件为响应式、可搜索的文档网站，并通过 GitHub Pages 持续集成部署。

站点地址：👉 [https://xjg-0216.github.io](https://xjg-0216.github.io)

---

##  项目结构

```plaintext
.
├── mkdocs.yml           # MkDocs 站点核心配置文件
├── docs/                # 所有 Markdown 文档（站点内容）
│   ├── index.md         # 主页
│   └── ...              # 其他内容分类
├── mkdocs/css/          # 可选，自定义 CSS
├── mkdocs/javascripts/  # 可选，自定义 JS
├── site/                # 由 MkDocs 自动生成的静态页面（build 后产生）
└── README.md            # 项目说明文档
```


##  使用方法

### 1. 安装依赖

> 需要 Python 环境（推荐 `Python >= 3.8`）

```bash
pip install mkdocs-material
```


### 2. 本地预览

```bash
mkdocs serve
```

* 启动本地开发服务器： [http://127.0.0.1:8000](http://127.0.0.1:8000)
* 编辑 Markdown 文件后，浏览器会自动刷新。


### 🏗️ 3. 构建静态站点

```bash
mkdocs build
```

* 输出到 `site/` 文件夹。
* `site/` 是可直接部署到任何静态网站托管的 HTML。


### ☁️ 4. 部署到 GitHub Pages

本仓库已配置 `GitHub Pages`，可使用 MkDocs 提供的部署命令：

```bash
mkdocs gh-deploy --force
```

* 会自动生成 `gh-pages` 分支并推送。
* Pages 访问地址为：[https://xjg-0216.github.io](https://xjg-0216.github.io)

---

## 主要特性

✅ Material 主题：

* 自动亮/暗色模式切换
* 响应式设计，手机可读
* 强大的搜索功能（支持中英日多语言）
* 代码块高亮，支持复制按钮
* 数学公式（KaTeX）渲染
* 支持提示块、标签、脚注、任务列表等 Markdown 扩展
* 与 GitHub 仓库集成，支持页面一键跳转编辑

✅ 完全静态、可持续部署：

* 所有内容由 Markdown 驱动，易读易写
* 使用 `GitHub Actions` 或本地命令一键发布
* 免费托管，无需额外服务器资源

---

## 相关链接

* [MkDocs 官方文档](https://www.mkdocs.org/)
* [Material for MkDocs 主题文档](https://squidfunk.github.io/mkdocs-material/)
* [GitHub Pages 使用指南](https://docs.github.com/en/pages)

---

## 🗂️ License

本项目遵循 MIT License，可自由使用与修改。

---

## ✨ Maintainer

* **Author:** xujg
* **Site:** [https://xjg-0216.github.io](https://xjg-0216.github.io)


