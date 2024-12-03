---
title: 自动构建社交feed到你的主页
date: 2024-12-03 23:19:05
tags: 
  - 持续集成
---

项目地址：[social-readme](https://github.com/zylele/social-readme)，欢迎点个star～

Automatically build Social feeds in your Profile Readme everyday, preview: <a href="https://github.com/zylele" target="_blank">github.com/zylele</a>

自动构建社交feed到你的主页readme中，预览：<a href="https://github.com/zylele" target="_blank">github.com/zylele</a>

![image](https://user-images.githubusercontent.com/12383106/233308515-09e00a2a-9277-44a7-ba7b-1e975c50a326.png)

或者我的博客，“关于我”页面，预览：<a href="https://zylele.github.io/about/" target="_blank">「关于」 | 乐章</a>

![image](https://user-images.githubusercontent.com/12383106/233309981-20be571c-92d6-4ff5-b9b1-a7df9049a3ad.png)

## 目前支持

- rss，符合rss2.0或atom标准，比如 [我的博客](https://zylele.github.io/) rss链接是https://zylele.github.io/atom.xml
- 豆瓣（想看、在看和看过的书和电影，想听、在听和听过的音乐）

## GitHub主页构建使用方式

### 1.确认存在Profile Repository(`<username>/<username>`)项目

> 仓库名与你的GitHub用户名相同的，就是Profile Repository
> 
> 这是GitHub的一个彩蛋，仓库根目录的README.md文件将会被渲染展示在你的个人公共主页上
> 
> 比如我的Profile Repository是 [github.com/zylele/zylele](https://github.com/zylele/zylele) ，README.md将会展示在我的主页上：[zylele(Eric)](https://github.com/zylele)

### 2.修改根目录文件

根据你的需要，在你的readme中增加以下内容

博客:
```html
<!-- START_SECTION:blog -->
<!-- END_SECTION:blog -->
```

豆瓣:
```html
<!-- START_SECTION:douban -->
<!-- END_SECTION:douban -->
```

这些是构建feed信息的识别点

### 2.配置workflow文件

- 在你项目仓库的根目录，新建`.github/workflows/social-readme.yml`，或者编辑其他已有的workflow文件

- 拷贝以下代码到上一步的文件中，根据你的需要，选填博客atom链接`blog_rss_link`，豆瓣用户名`douban_name`（进入豆瓣个人主页，douban.com/people/username/，这里地址中的username就是用户名）

我的GitHub主页参考配置如下：

```yml
name: Social Readme

on:
  schedule:
    - cron: '0 0 * * *'
  workflow_dispatch:
  push:
    branches:
      - master

jobs:
  update-social:
    runs-on: ubuntu-latest
    steps:
      - uses: zylele/social-readme@master
        with:
          blog_rss_link: https://zylele.github.io/atom.xml
          douban_name: znyalor
```

## 非根目录readme文件构建使用方式

如我的博客的源码仓库，其中 [关于我](https://zylele.github.io/about/) 页面对应的仓库文件是`source/about/index.md`

### 1.修改目标文件

同样的，在目标文件中插入上述的“识别点”

### 2.配置workflow文件

同样的，在仓库的工作流文件中增加workflow配置。

因为是更新非根目录的readme文件，则需要配置`file_path`参数来指定文件路径

这里我的配置如下：

```yml
name: Social Readme

on:
  schedule:
    - cron: '0 0 * * *'
  workflow_dispatch:
  push:
    branches:
      - master

jobs:
  update-social:
    runs-on: ubuntu-latest
    steps:
      - uses: zylele/social-readme@master
        with:
          douban_name: znyalor
          file_path: source/about/index.md
```

## 完整配置说明

如果你想定制更多构建细节，在workflow文件中的`with`有如下可选参数

```yml
- uses: zylele/social-readme@master
  with:
    blog_rss_link: # 博客链接
    blog_limit: 5 # blog数量
    douban_name: # 豆瓣用户名
    douban_limit: 5 # 豆瓣最新动态数量
    commit_message: Updated social rss by social-readme # commit说明
    file_path: # 更新非readme文件，填写仓库中的文件路径，如source/about/index.md
```

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=zylele/social-readme&type=Date)](https://star-history.com/#zylele/social-readme&Date)
