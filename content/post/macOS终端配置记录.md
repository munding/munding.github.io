---
title: "MacOS 终端配置记录"
date: 2021-06-04
tags: ["macOS",""]
categories: ["操作系统","最佳实践"]
description: ""
summary: "macOS 下 iTerm2 + oh-my-zsh + powerlevel10k 环境快速配置"
draft: false
---

## 安装 iTerm2

使用 iTerm2 替代 macOS 自带终端 Terminal

Home&Download：*https://iterm2.com/*

## 安装 iTerm2 主题

This is a set of color schemes for iTerm (aka iTerm2).

Github：*https://github.com/mbadolato/iTerm2-Color-Schemes*

Example:

- [Dracula](https://draculatheme.com/)
- [iterm2-material-design](https://www.martinseeler.com/iterm2-material-design)

## 安装 oh-my-zsh

Oh My Zsh is a delightful, open source, community-driven framework for managing your Zsh configuration.

Home：*https://ohmyz.sh/*

Github：*https://github.com/ohmyzsh/ohmyzsh*

```bash
# Install oh-my-zsh
$ sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## 安装 zplug 插件管理器

Github：*https://github.com/zplug/zplug*

```bash
$ curl -sL --proto-redir -all,https https://raw.githubusercontent.com/zplug/installer/master/installer.zsh | zsh
```

## 安装命令行工具

### fzf

命令行模糊搜索

Github：*https://github.com/junegunn/fzf*

```bash
brew install fzf
# To install useful key bindings and fuzzy completion:
$(brew --prefix)/opt/fzf/install
```

**NOTE：put this line in the end of your zshrc, or it may not work, https://github.com/junegunn/fzf/issues/1304**

```bash
[ -f ~/.fzf.zsh ] && source ~/.fzf.zsh
```



## 新增 zpug 插件配置

添加到 `~/.zshrc` 文件

```bash
# install zplug, plugin manager for zsh, https://github.com/zplug/zplug
# curl -sL --proto-redir -all,https https://raw.githubusercontent.com/zplug/installer/master/installer.zsh | zsh
# zplug configruation
if [[ ! -d "${ZPLUG_HOME}" ]]; then
  if [[ ! -d ~/.zplug ]]; then
    git clone https://github.com/zplug/zplug ~/.zplug
    # If we can't get zplug, it'll be a very sobering shell experience. To at
    # least complete the sourcing of this file, we'll define an always-false
    # returning zplug function.
    if [[ $? != 0 ]]; then
      function zplug() {
        return 1
      }
    fi
  fi
  export ZPLUG_HOME=~/.zplug
fi
if [[ -d "${ZPLUG_HOME}" ]]; then
  source "${ZPLUG_HOME}/init.zsh"
fi
zplug 'plugins/git', from:oh-my-zsh, if:'which git'
zplug 'romkatv/powerlevel10k', use:powerlevel10k.zsh-theme
zplug "plugins/vi-mode", from:oh-my-zsh
zplug 'zsh-users/zsh-autosuggestions'
zplug 'zsh-users/zsh-completions', defer:2
zplug 'zsh-users/zsh-history-substring-search'
zplug 'zsh-users/zsh-syntax-highlighting', defer:2

if ! zplug check; then
  zplug install
fi

zplug load
```

## 关于字体

在配置 [powerlevel10k](https://github.com/romkatv/powerlevel10k) 主题时若缺少字体会提示下载，当然你也可以自行安装

> Best option if on **macOS** and want to use **Homebrew**.

All fonts are available via [Homebrew Cask Fonts](https://github.com/Homebrew/homebrew-cask-fonts) on macOS (OS X)

```bash
brew tap homebrew/cask-fonts
brew install --cask font-hack-nerd-font
```

iTerm2 -> Preferences -> Profiles -> Text -> Non-Ascii-Font -> nerd-font -> restart iTerm2
