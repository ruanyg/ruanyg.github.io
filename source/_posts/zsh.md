---
title: zsh安装与使用
date: 2020-03-18 23:05:29
tags:

---

### 安装oh-my-zsh
```shell
git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
```
### zsh的开启
设置zsh为你的默认的shell
```shell
chsh -s /bin/zsh
```
重启并开始使用你的zsh (打开一个新的终端窗口便可)

### 插件安装方法
以安装zsh-autosuggestions为例
```shell
git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```
编辑~/.zshrc文件，找到plugins=(git)这一行，然后再添加autosuggestions，最后为
```shell
plugins=(git zsh-autosuggestions)
```
source ./zshrc更新下你的zsh

### 常用插件
* z
* zsh-autosuggestions

### 修改主题
编辑~/.zshrc文件，找到ZSH_THEME="robbyrussell"这一行，修改为推荐主题ys，最后为
```shell
ZSH_THEME="ys"
```