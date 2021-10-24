---
title: Hello World
date: 2019-09-28 17:45:09
tags: [markdown, hexo, blog]
---

Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## Quick Start

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate

# watch for file changes and only write if file changes are detected.
$ hexo generate --watch  
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
# same as `hexo deploy --generate`
# npm run hexo generate -- --deploy  
$ hexo generate --deploy
```

###  保存源代码（新建分支）
- 我们用 master 分支来保存 hexo 生成的静态网页，对于博客源码，可以新建一个 source 分支来存储。在 username.github.io 仓库建立一个 source 分支
- github 上的仓库初始都会有一个 master 分支，也是默认分支。对于一个仓库，**通过 git clone 下载代码时，拉取的是默认分支 master 的代码**。我们用 hexo 写博客时，对于 deploy 生成的 master 分支代码并不需要特别关注，因此可以**将仓库的默认分支改为保存源码的 source 分支，这样通过 git clone 拉取的就是 source 分支的代码**
- 在仓库的主页面，通过 Settings -> Branches，可以看到 Default branch 显示的默认分支是 master，可以勾选 source，然后 update 即可将默认分支设置为 source
- 将本地 hexo 目录与远程仓库关联，执行 `git remote add origin https://github.com/xxx/xxx.github.io.git`
- 本地建立 track 远程的 source 分支并切换到 source 分支，执行 `git checkout -b source origin/source`
- 将本地的 md 源文件、站点配置文件等推送到 source 分支。注意 source 分支是从 master 分支新建的，初始代码就是 master 的拷贝，因而 public 等 deploy 生成的文件也会一起带过来，这些都不算是博客源文件，可以在本地先把它删掉，然后再 `git add && commit && push origin source`
- hexo deploy 推送的还是 master 分支，hexo 目录下的 `_config.yml` 文件配置的 deploy 推送的目标地址和分支
- 假设更换了电脑，要在原有仓库上写文章，此时通过 git clone 将博客源码拉到本地，然后安装 hexo 即可
