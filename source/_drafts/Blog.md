---
# https://hexo.io/zh-cn/docs/front-matter
layout: post
title: Blog  # 必需
urltitle:  # 注意
date: 2024-06-03 22:39:00 # 2024-06-02 20:32:22 # 必需
updated:  # 注意
comments: true  # 注意
tags:  # 注意
  - 博客
categories:  # 注意
  - Life
  - Diary
permalink: null  # 覆盖永久链接
disableNunjucks: false  # 禁用Nunjucks插件标签
lang:  
published: false  # 是否发布 # 注意

# https://butterfly.js.org/posts/dc584b87/#Post-Front-matter
keywords:  # 注意
description:  # 注意
top_img:  # 注意
cover:  # 封面  # 注意
toc: true
toc_number: true # 是否显示额外数字标题  # 注意
toc_style_simple:  # 文章页侧边栏只显示TOC
copyright: true # 显示版权(转载文章false)
copyright_author:
copyright_author_href:
copyright_url:
copyright_info:
mathjax: false # 是否加载 mathjax.js
katex: false # 是否加载 katex.min.css  # 注意
aplayer:  # aplayer音乐插件
highlight_shrink:  # 代码框是否展开
aside:  # 是否显示侧边栏
abcjs:  # abcjs乐谱渲染
series:  # 系列文章 # https://butterfly.js.org/posts/4aa8abbe/#series-%E7%B3%BB%E5%88%97%E6%96%87%E7%AB%A0

sticky:  # 置顶
---



# Blog框架

Hugo：文章多速度快

Hexo：主题更好看些
[hugo主题](https://themes.gohugo.io/)	|	[hexo主题](https://hexo.io/themes/)	|	[npm rank hexo theme](https://www.npmjs.com/search?q=keywords%3Atheme%20hexo%20&ranking=popularity)



## Hexo7.2.0

[官方文档](https://hexo.io/zh-cn/docs/)



### 目录结构

```bash
.
├── _config.yml  # 站点配置文件
├── package.json
├── scaffolds  # 模板文件夹 预设布局layout
├── source  # 用户资源
|   ├── _drafts  # 草稿
|   └── _posts  # 文章
└── themes
	└── butterfly  # butterfly主题
		├── _config.yml  # 主题配置
		├── languages
		├── layout  # 模板
		├── scripts
		└── source
```



### hexo cli脚手架命令

| 项目目录下> npx hexo <Command> | 释义                                             | 选项                            | 释义                     |
| ------------------------------ | ------------------------------------------------ | ------------------------------- | ------------------------ |
| **部署**                       |                                                  |                                 |                          |
| clean                          | 清理缓存文件db.json、静态文件public              |                                 |                          |
| generate                       | 生成静态文件 public                              | --watch                         | 增量生成                 |
| server                         | 启用本地HTTP静态服务器                           |                                 |                          |
| deploy                         | public推送到指定的远程仓库分支，完全覆盖已有内容 | --m ”“                          |                          |
| **文章**                       |                                                  |                                 |                          |
| init [folder]                  |                                                  |                                 |                          |
| new [layout] <title>           |                                                  | layout 预设预设布局  scaffolds/ | post, draft, page...     |
|                                |                                                  | --path                          | 指定相对Source的相对路径 |
|                                |                                                  | --lang                          |                          |
| publish [layout] <filename>    | 将draft转换为layout发布                          |                                 |                          |



### hexo-theme-butterfly@4.13.0

[官方文档](https://butterfly.js.org/categories/Docs%E6%96%87%E6%AA%94/)

#### 搜索

algolia



#### 评论

选型：
[Giscus](https://github.com/giscus/giscus)(最佳实践)：基于 GitHub Discussions，star比DisqusJS高，维护容易
[DisqusJS](https://github.com/SukkaW/DisqusJS)：基于[disqus](https://disqus.com/)
~~[Waline](https://waline.js.org/)：从[Valine](https://github.com/xCss/Valine)衍生，[github](https://github.com/walinejs/waline)，需要ICP备案~~
~~[remark42](https://github.com/umputun/remark42)：只支持私有部署，样式一般~~
~~[artalk](https://github.com/ArtalkJS/Artalk)：只支持私有部署(有点重，需要用docker部署，无云服务)，样式一般~~
~~[Twikoo](https://github.com/twikoojs/twikoo)：样式一般~~



不行：
~~[Valine](https://github.com/xCss/Valine)：不维护了~~
~~[livere](https://livere.com/city-demo?livereVersion=new)：样式太老~~
~~[Gittalk](https://github.com/gitalk/gitalk)：不维护了~~
~~[Utterances](https://github.com/utterance/utterances)：不维护了~~



##### Giscus

功能：访客可以直接在 GitHub Discussion 里评论(Announcements类)，作者也可以在 GitHub 上管理评论

原理：当 giscus 加载时，GitHub Discussion search API 用于根据所选映射（URL、 pathname 、 title 等）查找与页面关联的Discussion。如果找不到匹配的讨论，由 giscus-bot 将创建讨论与所选映射关联的 Discussion

维护：
如果文档的路径更换了，需要同步修改下对应 Discussion 中的 SHA1 Hash 值（可使用 SHA-1 在线工具进行计算，比如 [SHA-1 hash calculator](https://xorbin.com/tools/sha1-hash-calculator)）



#### 参考

https://www.pil0txia.com/post/2021-08-05_java-only-pass-by-value/

https://blog.ccknbc.cc/posts/hexo-webpushr-notification/

https://butterfly.js.org/

https://blog.crazywong.com/

https://www.fomal.cc/



### 插件

hexo-wordcount

~~hexo-abbrlink：解决中文路径~~

[hexo-renderer-markdown-it](https://github.com/hexojs/hexo-renderer-markdown-it)



## hugo

golang 实现，比 Hexo 渲染速度更快，但没看到喜欢的 theme



# 建站



## 站点图标

站点图标比较小，需要简单些

AIGC生成图片
[online photoshop - photopea](https://www.photopea.com/)改成透明背景png
[bitbug](https://www.bitbug.net/)生成 .ico 图标

## 部署项目

push 源项目目录到 <username>.github.io ，使用 Github Actions 部署，https://<username>.github.io

push 源项目目录到任意项目，使用 GitHub Actions 部署，https://<username>.github.io/<projectname>

push public，使用 hexo-deployer-git 插件一键部署(npx hexo deploy)
缺点：push --force 会覆盖一切，commit记录不可改



### CI/DC

CI/DC(持续集成/持续部署)，设置从分支部署(配置发布源为main) ~~通过Github Action 实现~~，即 git push 后自动更新博客内容。



[hexo-action](https://github.com/sma11black/hexo-action)：仅供参考，两年没更新了
[Hexo文档部署Github Action](https://hexo.io/zh-cn/docs/github-pages)

hexo source code repo 是私有的
Github Pages repo 是公开的，配置发布源



## SEO

规则：
一般SEO只爬三层
中文不利于 SEO



# 问题

问题1：

```bash
(node:23108) [DEP0040] DeprecationWarning: The `punycode` module is deprecated. Please use a userland alternative instead.
(Use `node --trace-deprecation ...` to show where the warning was created)
```

原因：可能是KaTex markdown-it-kate的问题 或 hexo依赖punycode
解决：TODO

问题2：永久链接路径含有中文会别转译为百分十六进制编码
解决：
~~title英文，页面title不利于查看~~
~~hexo-abbrlink插件，路径数字没有语义~~
front-matter 中添加一个 urlname 作为永久链接路径



# TODO

1上SEO
hexo-generator-sitemap 使博客能被搜索引擎收录

2添加github项目信息

3看github page action部署

4解决DeprecationWarn警告

5了解备案



部署在 Vercel 上
