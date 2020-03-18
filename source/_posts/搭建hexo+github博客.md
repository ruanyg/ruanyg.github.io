---
title: 搭建hexo+github
date: 2020-03-18 18:50:45
tags:
---

### 新建仓库
github新建一个仓库，仓库名必须为 &lt;user-name>.github.io 格式，其中 &lt;user-name> 是你 github 的昵称。

### 全局安装hexo
```shell
npm install -g hexo
```

### 初始化项目
新建本地项目hexo-blog，执行下述命令自动构建hexo项目
```shell
hexo init
```

### 主题引入
主题官网为：https://hexo.io/themes
以引入NexT主题为例，进入hexo项目的themes文件夹下
```bash
git clone https://github.com/theme-next/hexo-theme-next.git
```
在hexo项目根目录下找到_config.xml，将theme属性改为hexo-theme-next

### 部署到github
编辑_config.xml，
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:ruanyg/ruanyg.github.io.git
  branch: master
```
安装hexo-deployer-git，用来帮助我们把博客项目推到仓库上
```shell
npm install hexo-deployer-git --save
```
执行如下命令，把项目自动部署到github上
```shell
# 清除缓存
hexo clean
# 生成静态页面
hexo g
# 将资源提交到服务器中
hexo d
```

### 查看效果
浏览器访问：https://ruanyg.github.io/ 即可看到效果。

### 创建新文章
```shell
# [layout] 为布局，可选项为 `post`、`page`、`draft`，这将决定文章所在文件路径。
# <title> 为文章标题
# 如 hexo new post 除了帅气，我还有啥！
hexo new [layout] <title>
```

### 部署优化
每次都要执行 hexo clean 和 hexo deploy，不如写个新的脚本，编辑package.json
```shell
"dev": "hexo s",
"build": "hexo clean & hexo deploy"
```

### 接入评论系统 - valine
https://valine.js.org/quickstart.html

### 其他
* [映射域名](https://www.jianshu.com/p/683a120c6264)