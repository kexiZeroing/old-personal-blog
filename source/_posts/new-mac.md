1. 第一步就是科学上网，要保证身边有一台电脑可以从 github 下载 [clashX](https://github.com/yichengchen/clashX/releases) 以及获取 ss 订阅链接
2. 安装 brew, oh my zsh 等提示 curl: fail to connect raw.githubusercontent.com port 443. DNS 被污染，需要修改 host, sudo vi /etc/hosts
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
3. git --version, 提示安装 Command Line Developer Tools
4. 安装 Homebrew, https://brew.sh/  根据运行最后的提示(Next steps) add Homebrew to your **PATH**
5. app 安装, `brew install --cask visual-studio-code google-chrome firefox iterm2` and `brew install yarn nvm`
> `cask` is no longer a `brew` command. When you want to install a Cask, you just do `brew install` or `brew install --cask` instead of `brew cask install`.
6. 登录 Chrome，同步插件、收藏夹等，set devtool's theme to "Dark" and go to Experiments and select "Allow extensions to load custom stylesheets".
7. 使用 iTerm2，可以设置 hotkey 比如全屏方式，安装 Oh My Zsh https://github.com/ohmyzsh/ohmyzsh 查看 ~/.zshrc, 设置 theme, alias 等
8. Set global configuration with Git `touch ~/.gitconfig`
9. 使用 nvm 安装 node, `nvm --help`, and check `node -v`, `npm -v`
10. 根据之前电脑，安装需要的 VS Code 插件
11. Github ssh key, `ssh-keygen -t rsa -b 4096 -C "your_email@example.com"` and `pbcopy < ~/.ssh/id_rsa.pub`, then add it to Github SSH keys.
