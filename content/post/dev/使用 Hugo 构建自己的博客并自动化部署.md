---
title: "使用 Hugo 构建自己的博客并自动化部署"
date: 2023-04-05
slug: "hugo-usage-and-auto-deployment"
tags: ["dev"]
categories: [Dev]
draft: false
---


# 概述

Hugo 构建一个博客网站并结合 Github 实现自动化部署

> 写是最好的学

个人构建了一套从写到发布的自动化机制，所有的写作都用 markdown 语法，其实如果有时间，可以构建一套自动化发布到多平台的程序。

当我在本地写完一篇文章时候，我需要 Push 到 Github，然后你的文章会自动化被发布到多个平台: 个人博客，博客社区、公众号、等等。


# Install & Theme

## Install

Mac

```
brew install hugo
```


## New Site

```
hugo new site quickstart
cd quickstart
```

## Theme

```sh
git init
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke themes/ananke
echo "theme = 'ananke'" >> config.toml
hugo server
```

> to use postcss, you should have hugo extended version installed, then copy package.json and postcss.config.js to the root of your site folder, then npm install

[Theme Usage](https://github.com/wlh320/hugo-theme-hulga)

这个主题


# Config & Writing

## Config.toml

```sh
baseURL = 'https://jerrkill.github.io/'
languageCode = 'zh-cn'
DefaultContentLanguage = "zh-cn"
title = 'Jerrkill - 为学日益为道日损'
enableRobotsTXT = true
enableEmoji = true
theme = "hulga"
hasCJKLanguage = true
paginate = 10
[params.social]
  GitHub = "/jerrkill"
  Email   = "2273716951@qq.com"
[params.gitalk]
owner = "jerrkill"       # 你的GitHub ID       
repo = "blog"        # 博客网址    
clientId = "cf2e140204fdff47bf3c"    # 刚刚记录的client ID      
clientSecret = "bfc9f4bc2de4004a6af295ea89ec53cd693948be" # 刚刚记录的client secret
labels = "gitalk"
[Permalinks]
  posts = "/:year/:month/:slug/"
[params]
  # show in HTML meta tag
  author = "jerrkill"
  keywords = "Blog, Golang, PHP, blockchain, trading, life"
  description = "jerrkill's blog, dev trading life"

  # change bulma's primary color
  primaryColor = "#1793d0"

  # subtitle on homepage
  subtitle = "时间是最有限的资源，知识是最强大的杠杆。不要用时间交换任何不重要的东西。"

  # copyright text on footer
  copyright = "Copyright © 2022 jerrkill. All rights reserved."

  # enable katex rendering on every post page, default false
  math = false

  # enable postcss, mainly for css purge (129kB->20kB->4.8kB gzipped, but this makes build slower), default false
  postcss = true

  # enable showing content summary below post title in home page, default false
  showSummary = true

  # set paginate on taxonomy term page (tags or categories), default 10
  termPaginate = 3

  # enable toc on every post page, default false
  toc = true

  # enable TOC auto collapse, default false
  autoCollapseToc = true

  # enable prefers-color-scheme:dark, default false
  darkMedia = true

  # enable hero section's is-bold effect, default false
  heroBold = false

  # enable hero section that looks like steam deck's home page, default false
  heroSteamDeck = false

  # enable PWA, prepare your icons and DON'T forget to modify manifest.json, default false
  pwa = true

  # disable jsdelivr cdn, default false
  noCDN = false

[author]
  name = "jerrkill"

  [params.publisher]
    name = "jerrkill"

    [params.publisher.logo]
      url = "logo.png"
      width = 127
      height = 40

  [params.logo]
    url = "logo.png"
    width = 127
    height = 40


# to enable different hightlight themes in light/dark mode 
[markup]
  [markup.highlight]
    noClasses = false

[menu]
  [[menu.main]]
    identifier = "index"
    name = "首页"
    url = "/"
    weight = 1
  [[menu.main]]
    identifier = "archives"
    name = "归档"
    url = "/archives/"
    weight = 2
  [[menu.main]]
    identifier = "tags"
    name = "标签"
    url = "/tags/"
    weight = 3
  [[menu.main]]
    identifier = "categories"
    name = "类别"
    url = "/categories/"
    weight = 4
  [[menu.main]]
    identifier = "about"
    name = "关于"
    url = "/about/"
    weight = 5

[taxonomies]
category = "categories"
tag = "tags"


```


## 补充 关于和归档页面

new file in content dir

about.md
```
---
title: "关于本站"
date: 2023-01-28
type: about
---

# Welcome

> 本站是 jerrkill 的个人博客，不定期更新博文，包括但不限于 Dev/Trading/Living/BlockChain

# About Me

TODO

# Contact Me

你可以从一下地方找到我

- [Github](https://www.github.io/jerrkill)
- [给我发email](mailto:jerrkill123@gmail.com)
- [TV](https://cn.tradingview.com/u/jerrkill/)


# 版权

>本博客系列文章采用《[CC 协议](https://creativecommons.org/licenses/by-nc-nd/4.0/legalcode.zh-Hans) 协议》转载必须注明作者和文本链接
```
archives.md
```
---
title: 归档
description: "Archives"
type: archives
---
```

## Create a new post

```sh
hugo new post/living/my-first-post.md
```


```sh
hugo server
```

# 自动化部署


1. 创建一个Github账号并登录。在Github上新建一个repository，该repository的名称必须是username.github.io格式，其中username是你的用户名。

2. 生成一个Github访问令牌。

在Github中进入Settings -> Developer settings -> Personal access tokens，点击Generate new token按钮。

为令牌添加名称和权限，勾选repo和workflow两项权限，之后复制访问令牌。

3. 将访问令牌复制到Github的设置页面中。

在Github中进入Settings -> Secrets，在这里创建一个名为ACTIONS_DEPLOY_KEY的secret。将步骤4中生成的访问令牌粘贴到secret中即可。

4. 在本地仓库的根目录下，新建一个.github/workflows目录。

5. 在.github/workflows目录下，新建一个名为deploy.yml的文件。

6. 将以下代码复制到deploy.yml文件中：
`.github/workflows/deploy.yml`

```
name: github pages

on:
  push:
    branches:
      - master

jobs:
  build-deploy:
    runs-on: ubuntu-18.04

    steps:
      - uses: actions/checkout@master
      - name: Checkout submodules
        shell: bash
        run: |
          auth_header="$(git config --local --get http.https://github.com/.extraheader)"
          git submodule sync --recursive
          git -c "http.extraheader=$auth_header" -c protocol.version=2 submodule update --init --force --recursive --depth=1
          npm install

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build
        run: hugo --gc --minify --cleanDestinationDir

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          deploy_key: ${{ secrets.ACTIONS_DEPLOY_KEY }}
          publish_dir: ./public
          publish_branch: deploy # deploying branch
```

7. 在仓库中选择 Settings (设置) 选项卡。在 Settings 选项卡下，向下滚动直到找到 GitHub Pages 部分。在本部分下，你应该可以看到一个下拉菜单，其中列出了不同的源。将默认的 ”Branch:main” 改为 ”Branch:deploy”，然后点击 Save (保存) 按钮。

8. 最后 `git push origin master` 即可

# Docs

[Hugo Usage](https://gohugo.io/getting-started/quick-start/)
[Theme Usage](https://github.com/wlh320/hugo-theme-hulga)

