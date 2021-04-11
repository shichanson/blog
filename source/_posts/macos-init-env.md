---
title: "如何快速初始化 MacOS 开发环境"
author: "Mayer Shi"
tags: ["tools"]
categories: ["Tools"]
date: 2020-09-20 20:25:23
draft: false
---

很多开发者比较喜欢用MacBook作为生产力工具，对于新电脑可以快速配置下开发环境。

<!--more-->

# MacOS 开发者初始化工具

**打开以及关闭隐藏目录**
```bash
shift + command + .
```
**安装开发工具**
```bash
xcode-select --install
```
**brew 工具使用**
1. 安装brew工具
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)" 
```
2. brew 基本命令
```bash
brew -h #查找命令
brew search #搜索软件
brew install #安装软件
brew uninstall #卸载软件
brew update #更新所有软件
brew upgrade #更新具体软件
brew list #显示安装软件
brew info / home# 查看软件信息：（home是打开软件的官网）
brew outdated #查看哪些软件需要更新
```
**oh my zsh 安装以及配置插件**
1. 安装oh my zsh
```bash
$ sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
2. 安装autojump插件
```bash
brew install autojump
```
3. 安装zsh-autosuggestion插件
```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions 
```
4. 安装zsh-syntax-highlighting插件
```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```
5. 配置以及激活插件
```bash
vim ~/.zshrc # 添加选项
plugins=(git autojump zsh-autosuggestions zsh-syntax-highlighting)  # 重启控制台
```
**oh my zsh 配置 powerlevel10k 主题**
1. 安装 powerlevel10k
```bash
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k

# 配置方式 Set ZSH_THEME="powerlevel10k/powerlevel10k" in ~/.zshrc.
```
2. 配置powerlevel10k
```bash
p10k configure # 通过交互配置主题
```

**MacOS Hight Sierra清除DNS缓存**
```bash
sudo dscacheutil -flushcache;sudo killall -HUP mDNSResponder;
```

**安装常用软件**
```bash
brew cask install v2rayu
brew cask install shadowsocksx-ng
brew install redis
brew install helm
brew install kubernetes-cli
brew install mysql
brew cask install visual-studio-code
brew cask install typora
brew cask install iterm2
```