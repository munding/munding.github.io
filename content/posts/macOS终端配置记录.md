---
title: "MacOS终端配置记录"
date: 2021-07-24
tags: ["",""]
categories: ["",""]
description: ""
summary: "macOS终端环境快速配置"
draft: false
---

### 安装iTerm2

使用iTerm2替代macOS自带终端Terminal

Home&Download：*https://iterm2.com/*

### 安装iTerm2主题

This is a set of color schemes for iTerm (aka iTerm2).

Github：*https://github.com/mbadolato/iTerm2-Color-Schemes*

Example:

- [Dracula](https://draculatheme.com/)
- [iterm2-material-design](https://www.martinseeler.com/iterm2-material-design)

### 安装/升级zsh

```bash
# https://github.com/robbyrussell/oh-my-zsh
brew install zsh
# The installation script should set zsh to your default shell, but if it doesn't you can do it manually:
chsh -s $(which zsh)
```

### 安装 oh-my-zsh

Oh My Zsh is a delightful, open source, community-driven framework for managing your Zsh configuration.

Home：*https://ohmyz.sh/*

Github：*https://github.com/ohmyzsh/ohmyzsh*

```bash
# Install oh-my-zsh
$ sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### 安装 zplug 插件管理器

Github：*https://github.com/zplug/zplug*

```bash
$ curl -sL --proto-redir -all,https https://raw.githubusercontent.com/zplug/installer/master/installer.zsh | zsh
```

### 安装命令行工具

```bash
brew install fzf
# To install useful key bindings and fuzzy completion:
$(brew --prefix)/opt/fzf/install

brew install autojump
```

#### fzf

命令行模糊搜索

Github：*https://github.com/junegunn/fzf*

#### autojump

记录常用目录，不需要输入目录详细路径，`j`加目录名关键字即可

```bash
❯ j github.
.
~/aladdinding.github.io main*
```

### 修改.zshrc

```bash
# User configuration

# export MANPATH="/usr/local/man:$MANPATH"

# You may need to manually set your language environment
# export LANG=en_US.UTF-8

# Preferred editor for local and remote sessions
# if [[ -n $SSH_CONNECTION ]]; then
#   export EDITOR='vim'
# else
#   export EDITOR='mvim'
# fi

# Compilation flags
# export ARCHFLAGS="-arch x86_64"

# Set personal aliases, overriding those provided by oh-my-zsh libs,
# plugins, and themes. Aliases can be placed here, though oh-my-zsh
# users are encouraged to define aliases within the ZSH_CUSTOM folder.
# For a full list of active aliases, run `alias`.
#
# Example aliases
# alias zshconfig="mate ~/.zshrc"
# alias ohmyzsh="mate ~/.oh-my-zsh"

# the fuck config, must brew install thefuck
eval $(thefuck --alias)

# autojump config, brew install autojump
[[ -s `brew --prefix`/etc/autojump.sh  ]] && . `brew --prefix`/etc/autojump.sh
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion


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

# NOTE: use cc to clear screen. I use tmux ctrl+hjkl switch panel, but ctrl+l conflict with clear-screen
#bindkey "cc" clear-screen

# customize alias
[ -f ~/.alias ] && source ~/.alias

# To customize prompt, run `p10k configure` or edit ~/.p10k.zsh.
[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh

# fzf config, must brew install fzf
# NOTE: put this line in the end of your zshrc, or it may not work, https://github.com/junegunn/fzf/issues/1304
[ -f ~/.fzf.zsh ] && source ~/.fzf.zsh
```
