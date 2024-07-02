# README
***
欢迎你来到我的网站， 该网站是静态网站，主要内容是日常工作学习中的一些技术笔记和博客文章，使用`Markdown`格式编写， 通过`Mkdocs`构建项目文档
（[mkdocs.org](https://www.mkdocs.org)），最终通过`Github Action`部署在`Github Pages`上
***
下面分别介绍了mkdocs的基本命令， 项目的组织结构以及material主题支持的语法

#### Commands

* `mkdocs new [dir-name]` - Create a new project.
* `mkdocs serve` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs -h` - Print help message and exit.

#### Project layout

    mkdocs.yml    # The configuration file.
    docs/
        index.md  # The documentation homepage.
        ...       # Other markdown pages, images and other files.

#### material-theme
在该主题中，以下内容可做参考
- Note
```markdown
!!! note
    This is a note.
```
举例， 以下命令同理
!!! note
    this is a note

- Tip
```markdown
!!! tip
    This is a tip.
```

- Warning
```markdown
!!! warning
    This is a warning.
```

- Danger
```markdown
!!! danger
    This is a danger.
```

- Success
```markdown
!!! success
    This is a success.
```
- Info
```markdown
!!! info
    This is an info.
```
- Quote
```markdown
!!! quote
    This is a quote.
```
- Question
```markdown
??? question "What is the meaning of life, the universe, and everything?"
    42.
```
