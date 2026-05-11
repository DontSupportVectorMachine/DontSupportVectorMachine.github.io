\---



date: '2026-05-11T12:50:41+08:00'

draft: false

title: '博客主题切换指南：从 PaperMod 到 Stack'

tags: \['博客搭建', 'Hugo', 'Stack']

categories: \['教程']

cover:

&#x20;   image: '/img/default-cover.jpg'

&#x20;   alt: '主题切换'

&#x20;   relative: false



\---







> 上一篇记录了如何从零搭建 Hugo + PaperMod 博客。用了一段时间后，贪图视觉升级，决定换成\*\*自带侧边栏、支持封面图、暗色模式切换\*\*的 Stack 主题。这篇把完整切换流程和踩过的坑全记下来，方便以后自己回溯，也希望对同行者有点用。







\---



\## 为啥想换？



其实就是想要更好看的。



PaperMod 极简、快速，但默认\*\*单栏布局\*\*，想加侧边栏或文章封面需要较大改动。Stack 是 Hugo 社区里视觉表现力最丰富的主题之一，支持：



\- 左侧常驻侧边栏（头像、菜单、社交链接）

\- 文章封面图

\- 暗色模式一键切换

\- 搜索、归档、标签、分类页面开箱即用

\- 代码高亮风格可自定义



\*\*代价是需要稍微适应新的配置逻辑\*\*，这篇文章就是帮你平滑过渡。（依然是梳理了AI大人的结果，感谢AI大人帮我一步步排雷纠错。。。）







\---



\## 一、移除旧主题 PaperMod



\### 1. 进入项目、确认状态

```bash

D:

cd D:\\MyBlog\\DontSupportVectorMachine.github.io

git status   # 确保没有未提交的修改，如果有，提交或者放弃

```



\### 2. 删除子模块



```bash

git submodule deinit -f themes/PaperMod

```



​	这一步会取消注册子模块。接下来删除目录时，Windows PowerShell 里\*\*坑来了\*\*：`rm -rf` 是 Linux 命令，在这里无效。



​	正确的 PowerShell 命令是，直接输入：



```powershell

Remove-Item -Recurse -Force themes/PaperMod -ErrorAction SilentlyContinue

```



​	如果目录已被 `deinit` 清空，这条命令会静默跳过，也不会报错。



\### 3. 提交移除



```bash

git add .

git commit -m "移除 PaperMod 主题"

```



\------



\## 二、添加新主题 Stack



```bash

git submodule add https://github.com/CaiJimmy/hugo-theme-stack themes/Stack

```



​	这会克隆 Stack 到 `themes/Stack` 目录并自动生成 `.gitmodules` 记录。



\------



\## 三、改造 `hugo.toml` 配置文件



​	\*\*重要\*\*：Stack 的配置风格和 PaperMod 有很大区别，建议\*\*全量替换\*\*。下面是我的配置（已踩坑修正版）：



toml



```

baseURL = 'https://dontsupportvectormachine.github.io/'

title = '我的新博客'

theme = 'Stack'

locale = 'zh-cn'

paginate = 5



\[params]

&#x20; mainSections = \["posts"]

&#x20; sidebar.avatar = '/img/avatar.png'

&#x20; sidebar.subtitle = '记录我的学习生涯'

&#x20; article.excerpt = true

&#x20; highlight.highlightStyle = 'github-dark'



\[\[menu.main]]

&#x20; identifier = "home"

&#x20; name = "首页"

&#x20; url = "/"

&#x20; weight = 1



\[\[menu.main]]

&#x20; identifier = "archives"

&#x20; name = "归档"

&#x20; url = "/archives/"

&#x20; weight = 2



\[\[menu.main]]

&#x20; identifier = "categories"

&#x20; name = "分类"

&#x20; url = "/categories/"

&#x20; weight = 3



\[\[menu.main]]

&#x20; identifier = "tags"

&#x20; name = "标签"

&#x20; url = "/tags/"

&#x20; weight = 4



\[\[menu.social]]

&#x20; identifier = "github"

&#x20; name = "GitHub"

&#x20; url = "https://github.com/DontSupportVectorMachine"

&#x20; weight = 1

```



\*\*踩坑笔记\*\*：



\- `mainSections = \["posts"]` 这一行\*\*非常关键\*\*。没有它，首页只会显示侧边栏，不会列出任何文章。

\- 头像图片需放在 `static/img/avatar.png`，可以先用任意正方形图片替代。



\------



\## 四、清理旧的自定义样式



​	PaperMod 时代我们在 `assets/css/extended/custom.css` 里写了不少样式，这些 class 在 Stack 下不再生效，直接删除：



```powershell

Remove-Item -Recurse -Force assets/css/extended/custom.css -ErrorAction SilentlyContinue

```



​	后续想美化 Stack 时，可以重新创建同名文件，用 Stack 的 class 名调整。



\------



\## 五、让首页直接显示文章列表



​	Stack 默认首页布局是“个人资料卡片”，只有头像和社交链接。需要新建 `content/\_index.md` 并指定 \*\*layout: list\*\*：



```yaml

\---

layout: list

title: "Hi there！"

subtitle: "欢迎前来观看一个缓慢进步的菜狗的学习历程。"

\---

```



​	保存后，首页右栏就会按时间倒序列出 `content/posts/` 下所有已发布文章。\*\*再次强调\*\*：配合刚才 `hugo.toml` 里的 `mainSections = \["posts"]` 才能生效，缺一不可。



\------



\## 六、添加文章封面图



​	在 `static/img/` 中放入一张默认封面（例如 `default-cover.jpg`），然后在每篇文章的 Front Matter 里加上：



```yaml

cover:

&#x20;   image: '/img/default-cover.jpg'

&#x20;   alt: '文章封面'

&#x20;   relative: false

```



​	没有封面的文章在首页卡片里会缺少左侧缩略图，视觉上略逊一筹，建议新手期先用同一张默认图统一风格。



\------



\## 七、本地预览与上线



```bash

hugo server -D

\# 确认首页、归档、标签页都工作正常

git add .

git commit -m "切换主题为 Stack，修复首页列表显示"

git push

```



​    部署完成后，线上网站就焕然一新了。



\------



\## 八、后续可做的事



\- \*\*搜索功能\*\*：Stack 内置搜索，只需创建 `content/search/index.md`，内容为 `layout: "search"`，然后在菜单里添加搜索链接。

\- \*\*暗色模式\*\*：侧边栏底部有切换按钮，开箱可用。

\- \*\*评论系统\*\*：Stack 支持 Disqus、Waline 等，按需在 `hugo.toml` 里添加对应 block 即可。







\------



\## 总结



从 PaperMod 到 Stack，核心变化只有三步：\*\*删旧子模块、添新子模块、重写配置文件\*\*。过程中注意：



\- Windows 下用 `Remove-Item -Recurse -Force` 替代 `rm -rf`

\- 务必配置 `mainSections`，否则首页无文章

\- 创建 `\_index.md` 并设 `layout: list`



切换后视觉体验提升显著，适合愿意花点时间在博客外观上的同学。希望这篇记录能让你少走点弯路。







