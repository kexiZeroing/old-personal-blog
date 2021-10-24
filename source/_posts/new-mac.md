---
title: New Mac Development Environment Setup
date: 2021-10-15 22:11:56
tags: [mac, mac m1, dev environment, configuration]
draft: true
---

## Setup for a new M1 Mac
1. 第一步就是科学上网，要保证身边有一台电脑可以从 github 下载 [clashX](https://github.com/yichengchen/clashX/releases) 以及获取 ss 订阅链接。
2. 安装 brew, oh my zsh 等提示 curl: fail to connect raw.githubusercontent.com port 443. DNS 被污染，需要修改 host, `sudo vi /etc/hosts`
    ```sh
    # GitHub Start
    192.30.253.119    gist.github.com
    199.232.28.133    assets-cdn.github.com
    199.232.28.133    raw.githubusercontent.com
    199.232.28.133    gist.githubusercontent.com
    199.232.28.133    cloud.githubusercontent.com
    199.232.28.133    camo.githubusercontent.com
    199.232.28.133    avatars0.githubusercontent.com
    199.232.28.133    avatars1.githubusercontent.com
    199.232.28.133    avatars2.githubusercontent.com
    199.232.28.133    avatars3.githubusercontent.com
    199.232.28.133    avatars4.githubusercontent.com
    199.232.28.133    avatars5.githubusercontent.com
    199.232.28.133    avatars6.githubusercontent.com
    199.232.28.133    avatars7.githubusercontent.com
    199.232.28.133    avatars8.githubusercontent.com
    # GitHub End
    ```
3. `git --version`, 提示安装 Command Line Developer Tools，之后可以查看到 git verison 2.30.1 (Apple Git-130)
4. 安装 [Homebrew](https://brew.sh/) 根据运行最后的提示(Next steps) add Homebrew to your **PATH**，之后可以执行 `brew help`
5. App 安装 (refer to https://formulae.brew.sh), `brew install --cask visual-studio-code google-chrome firefox iterm2` and `brew install yarn`, then use `brew list` and `brew info google-chrome` to check.
    > `cask` is no longer a `brew` command. When you want to install a Cask, you just do `brew install` or `brew install --cask` instead of `brew cask install`.
6. 登录 Chrome，同步插件、收藏夹等，set Devtool's theme to "Dark" and go to Experiments and select "Allow extensions to load custom stylesheets".
7. 使用 iTerm2，安装 [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh) 之后查看 `~/.zshrc` 文件, 设置 theme, alias 等。
8. Set global configuration with Git `touch ~/.gitconfig` and check `git config --list`
9. 安装 [nvm](https://github.com/nvm-sh/nvm), 之后检查 `nvm -v`, 使用 nvm 安装 node, `nvm ls`, and check `node -v`, `npm -v`
    > nvm install script clones the nvm repository to `~/.nvm`, and attempts to add the source lines to the correct profile file like `~/.zshrc` or `~/.bashrc`
    
    > nvm 默认是国外的服务器下载，在国内速度很慢，使用镜像 `NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node nvm install v12` 同理对于 npm install，使用 `npm --registry=https://registry.npm.taobao.org install xxx` 或者用 cnpm (require nodeJs >= 10.0.0), `npm install -g cnpm --registry=https://registry.npm.taobao.org`, 之后使用 `cnpm install`

    > 设置打开终端默认使用的 node 版本: `nvm alias default x.y.z`, 删除 nvm 管理的某个 node 版本: `cd ~/.nvm/versions/node` and `rm -rf x.y.z`

    > 当你用 nvm 尝试去安装 v14 及以下的 Node 版本时，大概率会报错（低版本的 node 并不是基于 arm 架构的），用 Rosetta 打开 iTerm (Get info -> Open using Rosetta)，然后再执行 `nvm install xxx`
10. Github ssh key, `ssh-keygen -t rsa -b 4096 -C your_email@example.com` (multiple ssh keys: `ssh-keygen -t rsa -b 4096 -C email@another.com -f $HOME/.ssh/another/id_rsa`) and `pbcopy < ~/.ssh/id_rsa.pub`, then add it to Github SSH keys. Type `ssh-add -K ~/.ssh/id_rsa` to store the passphrase (`-K` for adding in your keychain). Then clone a project and have a new commit.
11. 默认截屏，`Shift-Command-3` 捕捉整个屏幕，`Shift-Command-4` 捕捉屏幕的一部分，`Shift-Command 5` 打开截屏工具，可以设置延迟截屏，文件的存储位置 (某一路径或剪贴板)。
