---

date: '2026-05-11T12:50:41+08:00'

draft: false

title: '从零搭建 Hugo + GitHub Pages 个人博客（避坑版）'

tags: ['博客搭建', 'Hugo', 'GitHub']

categories: ['教程']

---



> 这是一篇**踩坑之后重新整理的避坑教程**，由于本人完全菜狗入门，可能有很多蠢问题都遇到了，也许你不会遇到。  
>
> 跟着它走，你不会遇到那些<u>已经被我趟过的雷</u>。



---

## 一、Windows 环境准备：安装 Hugo 和 Git

### 1. 安装 Scoop（Windows 包管理器）

​        Scoop 是一个轻量级的 Windows 包管理器，**强烈建议先安装它**，后续各种开发工具的安装都会非常省心。

​        快捷键Win+R，输入PowerShell：打开 Windows的**PowerShell**（以管理员身份运行），依次执行：

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
irm get.scoop.sh | iex
```

### 2. 用 Scoop 安装 Hugo 和 Git

```bash
scoop install hugo-extended
scoop install git
```

​        **注意**：一定要安装 `hugo-extended` 版本，后续很多主题会依赖它。

​        验证安装是否成功：

```bash
hugo version
git version
```

​		显示出版本号就算成功了。。。

------

## 二、创建本地站点并配置主题

### 1. 安装好基本工具后，可以创建站点了

​		选择一个**你拥有完全读写权限**的目录（我选D盘），创建 Hugo 站点，也就是你的博客在你本地磁盘上的存储位置：

```bash
D:
cd D:\MyBlog
hugo new site [你的用户名].github.io
```

​		进入项目根目录（后续所有命令都在这下面执行）：

```bash
cd D:\MyBlog\[你的用户名].github.io
```

### 2. 初始化 Git 并安装 PaperMod 主题

```bash
git init
git submodule add https://github.com/adityatelange/hugo-PaperMod themes/PaperMod
```

​        PaperMod 是目前 Hugo 社区最流行的主题之一，极简、高性能，适合长期维护。

### 3. 创建 `hugo.toml` 配置文件（避坑版）

​		在项目根目录下创建 `hugo.toml`，填入以下内容：

```toml
baseURL = 'https://dontsupportvectormachine.github.io/'
title = '我的新博客'
theme = 'PaperMod'
locale = 'zh-cn'

[params]
  ShowReadingTime = true
  ShowShareButtons = true
  ShowPostNavLinks = true
  ShowBreadCrumbs = true
  ShowCodeCopyButtons = true
  ShowToc = true

[params.homeInfoParams]
  Title = "Hi there!"
  Content = "欢迎来到我的博客，这里记录了我的学习与思考。"

[[menu.main]]
  identifier = "archives"
  name = "归档"
  url = "/archives/"
  weight = 10

[[menu.main]]
  identifier = "categories"
  name = "分类"
  url = "/categories/"
  weight = 20

[[menu.main]]
  identifier = "tags"
  name = "标签"
  url = "/tags/"
  weight = 30
```

**避坑要点**：

- Hugo 新版中 `languageCode` 已弃用，请使用 `locale`。
- PaperMod 的 `homeInfoParams` **必须用 `[params.homeInfoParams]` 作为独立的表头**，下面用等号赋值，这是 TOML 语法，不是 YAML，冒号写法会直接报错。

### 4. 创建 `.gitignore` 文件

​        **很重要！** 在项目根目录创建 `.gitignore`，内容如下：

```text
.hugo_build.lock
/public/
/resources/
```

> **为什么要加它？**
>
>  `public/` 目录是 Hugo 在**本地**运行时生成的静态网站文件。但如果被推送到 GitHub 上，会与 GitHub Actions 自动构建的 `public/` 目录产生冲突，导致部署失败。`.gitignore` 的作用就是**阻止这些文件被误提交**。



## 三、部署到 GitHub Pages（自动化）

### 1. 创建 GitHub 仓库

- 在 GitHub 上新建仓库，**仓库名必须为** `你滴名字.github.io`（例如 `DontSupportVectorMachine.github.io`）；
- **不要勾选任何初始化选项**（不要 README、不要 .gitignore、不要 License）。

### 2. 关联远程仓库并首次推送

```bash
git remote add origin https://github.com/你的用户名/你的用户名.github.io.git

git add .
git commit -m "Initial commit"
git branch -M main
git push -u origin main
```

**注意**：这必须是一个新建的仓库。

### 3. 配置 GitHub Actions 自动部署

在<u>项目根目录下</u>（也就是你本地磁盘上的目录）创建 `.github/workflows/hugo.yaml`，填入：

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: xxxx  # 请换成你本地的实际版本号，别直接复制我的，会报错
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${{ env.HUGO_VERSION }}/hugo_extended_${{ env.HUGO_VERSION }}_linux-amd64.deb
          sudo dpkg -i ${{ runner.temp }}/hugo.deb

      - name: Install Dart Sass
        run: sudo snap install dart-sass

      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0

      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5

      - name: Build with Hugo
        run: |
          hugo --gc --minify --baseURL "${{ steps.pages.outputs.base_url }}/"

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
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

**避坑要点**：

- `HUGO_VERSION` 必须与你的**本地版本严格一致**，否则可能导致构建失败。在终端运行 `hugo version` 查看后填入。（一定要换成自己的！！！！！！！
- 必须使用 `hugo_extended` 版本，因为 PaperMod 等主题通常依赖 Sass 处理样式。

### 4. 切换部署源

```bash
git add .
git commit -m "添加 GitHub Actions 部署脚本"
git push
```

​		推送后，在github中进入你的 GitHub 仓库 → 找到上侧**Settings** → 在左侧侧边栏找到**Pages** → **Build and deployment**，将 **Source** 从 `Deploy from a branch` 切换为 **GitHub Actions**。

​		现在，每次你 `git push` 代码后，GitHub Actions 就会自动构建并部署你的博客。



## 四、日常创作流程

### 1. 新建文章

```bash
hugo new posts/my-new-post.md
```

### 2. 撰写内容

用 Markdown 语法撰写正文。文章头部的 `draft: true` 表示草稿状态。

### 3. 发布

将文件头部的 `draft: true` 改为 `draft: false`（或删除该行），文章即可正式发布。

### 4. 本地预览（可选）

```bash
hugo server -D
```

浏览器打开 `http://localhost:1313/` 查看效果，按 `Ctrl+C` 停止。

### 5. 一键发布

```bash
git add .
git commit -m "发布新文章: 文章标题"
git push
```

推送后，GitHub Actions 会在 1-2 分钟内自动完成部署，稍等片刻就能在线上看到新文章。也可在“Actions”选项中，查看一下你的推送状态，是不是报错了；如果没报错，一般就是能正常显示了。



## 五、自定义外观样式（零基础也能做）

​		PaperMod 支持通过 **自定义 CSS** 来美化博客，完全不需要修改主题源码，升级主题也不会丢失你的修改。

### 1. 创建自定义样式文件

​		在项目根目录下，依次进入 `assets` → `css` → `extended`（没有路径就自己创建出来），创建一个 `custom.css` 文件。最终路径是：

```text
assets/css/extended/custom.css
```

​		**这就是你专属的样式文件**，Hugo 会自动加载它。

### 2. 一个简单的美化示例

​		将以下代码粘贴进 `custom.css`，保存后立刻就能看到效果：

```css
/* 全局背景：使用一张网络图片，也可以换成自己的图片 */
body {
    background: url('https://picsum.photos/1920/1080?random=1') no-repeat center center fixed;
    background-size: cover;
}

/* 首页欢迎区域 - 半透明卡片 */
.home-info {
    background: rgba(255, 255, 255, 0.6);
    backdrop-filter: blur(10px);
    border-radius: 16px;
    padding: 30px !important;
    max-width: 700px;
    margin: 20px auto;
    text-align: center;
    box-shadow: 0 8px 20px rgba(0,0,0,0.08);
}

/* 文章列表卡片 - 同样半透明，增加悬停效果 */
.main .post-entry {
    background: rgba(255, 255, 255, 0.7);
    backdrop-filter: blur(5px);
    border-radius: 10px;
    padding: 15px 20px;
    margin: 15px auto;
    max-width: 700px;
    box-shadow: 0 4px 10px rgba(0,0,0,0.05);
    transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.main .post-entry:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 18px rgba(0,0,0,0.12);
}

/* 导航栏增加轻微透明感 */
.nav {
    background: rgba(255,255,255,0.5);
    backdrop-filter: blur(5px);
}
```



### 3. 如何换成自己的背景图

1. 把你自己喜欢的背景图片（例如 `bg.jpg`）放入项目的 `static` 文件夹。

2. 修改 `custom.css` 中 `body` 的背景图路径为：

   ```css
   background: url('/bg.jpg') no-repeat center center fixed;
   ```

3. 重新运行 `hugo server -D` 预览，确认无误后推送即可。

### 4. 进阶微调

如果你熟悉前端，可以继续在 `custom.css` 里添加更多样式，修改任何元素的外观。建议保持“微调”原则，不要一次性大改，改一点 → 预览 → 再改，这样不容易出错。

------

## 六、总结

以上就是用 Hugo + GitHub Pages 搭建一个完全免费、自动部署的个人博客的全过程，以及如何简单地美化它。

**你不需要懂 Go 语言，不需要手写 HTML，也不需要手动维护首页。**
你只需要：

1. 用 Markdown 写文章
2. 用 CSS 微调样式（可选）
3. `git push` 一下就上线



