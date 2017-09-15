---
layout: post
title: 搭建hexo
date: '2016-02-12 08:29'
comments: true
customizeurl: bulid-hexo
tags:
  - node
  - npm
  - hexo
abbrlink: 41883
---

- 前往 http://nodejs.cn/download/ 下载nodejs系统环境对应安装包,进行安装

- 切换npm 淘宝镜像源(Linux添加一下环境变量)

```
临时使用: npm --registry https://registry.npm.taobao.org install express
持久使用:npm config set registry https://registry.npm.taobao.org
查看:npm config get registry
```

- 安装hexo环境
<!-- more -->

```
npm config set user 0   			#Linux执行此命令
npm config set unsafe-perm true		#Linux执行此命令

npm install -g hexo
```

- 新建文件夹 : blog 进入 

```
heox init   #初始化
npm install #安装模块
heox g      # 生成
heox s      # 启动服务
```

- 切换主题:在github选择好看的hexo主题,例:[hexo-theme-jekyll](https://github.com/pinggod/hexo-theme-jekyll),[hexo-theme-yilia](https://github.com/litten/hexo-theme-yilia).使用git clone 到 blog/themes文件下.

- 使用主题:在blog文件夹下_config.yml文件中配置theme为要修改的主题名称:theme: yilia.输入heox g,heox s 浏览[http://localhost:4000/](http://localhost:4000/).

- 展示所有文章列表 安装

```
npm i hexo-generator-json-content --save
```
- blog目录下_config.yml添加此配置项

````json
jsonContent:
  meta: false
  pages: false
  posts:
    title: true
    date: true
    path: true
    text: false
    raw: false
    content: false
    slug: false
    updated: false
    comments: false
    link: false
    permalink: false
    excerpt: false
    categories: false
    tags: true
````

- 用于生成分享插件
```
npm install hexo-helper-qrcode --save #安装即可
```
[![](/img/20170915214312.png)](/img/20170915214312.png)