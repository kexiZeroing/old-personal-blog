# 科学上网

## 为什么上不去 Google
GFW 实现网络封锁的手段主要包括 dns 劫持和 ip 封锁。IP 是网络上各主机的地址，dns 将域名和 IP 关联起来，形成映射。而 GFW 所做的就是站在用户和 dns 服务器之间，破坏它们的正常通讯，并向用户回传一个假 ip，也就访问不到本想访问的网站了。dns 劫持是 GFW 早期的技术手段，用户通过修改 Hosts 文件就可以突破封锁。dns 劫持之后，GFW 引入了 IP 封锁，直接锁住了访问目标网站的去路，用户发往被封锁 IP 的任何数据都会被墙截断。这个时候，只能依靠在第三方架设服务器，代理与目标服务器间的来往，目前几乎所有的翻墙手段都是基于这个原理。

- dns 污染
- 关键词过滤中断连接
- 端口阻断
- IP 地址批量封锁

## VPN 代理
<img alt="vpn" src="https://cdn.nlark.com/yuque/0/2019/png/398686/1564409310893-64650e9c-8a63-42b5-a335-ad9c0d30d302.png" width="800">

假设我们有一台国外 A 中转服务器，可以通过 SSH 被国内 B 机器连接，那么理论上，A 能做的事情，B 也能做到，因为有一条网络通路，即所谓的隧道技术。用户和境外服务器基于 SSH 建立起一条加密的通道，SSH 客户端侦听本地的端口并转发请求到服务端处理，通过 SSH Server 向真实的服务发起请求，服务最后通过创建好的隧道返回给用户。由于 SSH 本身基于 RSA 加密，所以 GFW 无法从数据传输过程中的加密数据内容进行关键词分析，请求被放行。但由于创建隧道和数据传输的过程中，VPN 代理的特征是明显的或者说过墙的数据都遵循特定的套路（先发送一个建立加密通道的数据包，然后紧跟着一个代理请求），所以 GFW 一度通过分析连接的特征进行干扰，导致 SSH 存在被定向进行干扰的问题。

- VPN 通讯端口被封锁，需要申报
- 提供 VPN 服务的服务器 IP 地址被封
- VPN 的功能是加密通讯，设计原理就不是用来翻墙的

## Shadowsocks（Socks5 代理）
<img alt="vpn" src="https://cdn.nlark.com/yuque/0/2019/png/398686/1564410024573-948a2fda-b882-4676-8885-be3cb1db2900.png" width="800">

Shadowsocks 的出现是一个拐点，它把代理服务器拆分成 server 端和 client 端，经过 GFW 的流量全部加密，从而消除明显的流量特征。在服务器端部署完成后，按照指定的密码、加密方式和端口，使用客户端软件与其连接。在成功连接到服务器后，客户端会在用户的电脑上构建一个本地 socks5 代理。浏览网络时，网络流量会被分到本地 socks5 代理，客户端将其加密之后（在本地完成加密）发送到服务器，服务器以同样的加密方式将流量回传给客户端，以此实现代理上网。ss-local 一般是本机或局域网的其他机器，不经过 GFW，所以不会被 GFW 通过特征分析进行干扰。ss-local 和 ss-server 两端通过多种加密方法进行通讯，经过 GFW 的时候是常规的 TCP 包，没有明显的特征码而且 GFW 也无法对通讯数据进行解密。

收费的是什么？当使用 ss 代理访问网站的时候，请求会被转发到 Shadowsocks 服务端上，再由它请求我们的目标网站。其原理就是在国外的服务器上装了 Shadowsocks 的服务，再开出了一些端口和密码供客户端连接，那么这些端口和密码可能是别人免费提供的，或者收费的，或者自己买 VPS 服务器搭建的。

- Shadowsocks 是一个 project，是一种传输协议（ShadowsocksR, V2Ray, Trojan 也都是协议），分为 client 端和 server 端，实现了两端之间的加密数据传输，它不是一个具体的软件或工具。ShadowsocksX-NG 是 Mac 上免费的一个客户端。
- 浏览器、邮件、文件传输都是在应用层；包括 Shadowsocks，V2Ray 等 Socks5 类型的代理都是在会话层，所以可以代理应用层的数据；游戏数据是直接通过传输层协议 TCP 和 UDP 进行通讯的，不经过会话层，所以 Socks5 正常情况下是不能代理游戏通讯数据的（即使开了全局代理）；PING、TRACE 这些 ICMP 指令都是在网络层，也不通过 Socks5 代理转发；主流的 VPN 协议都是在数据链路层，近乎所有的流量都可以被 VPN 代理。

### Shadowsocks 服务端
- VPS (virtual private server)，可以把它简单地理解为一台在远端的强劲电脑。当你租用了它以后，可以给它安装操作系统、软件，并通过一些工具连接和远程操控它。Vultr、搬瓦工都是 VPS 服务器提供商。
- SSH 是一种网络协议，作为每一台 Linux 电脑的标准配置，用于计算机之间的加密登录。当租用的 VPS 安装上 Linux 系统后，就可以用 SSH在自己的电脑上远程登录该 VPS。
- 部署 Shadowsocks 服务器端，可以使用 shadowsocks-libev 软件包来部署 https://github.com/shadowsocks/shadowsocks-libev
- 记录服务器 IP、服务器 ss 端口、设置的密码、混淆方式和加密方式。（如果某一天你的 ss 突然无法使用了，很可能就是端口被封了，这时候可以尝试修改配置将端口改为 1-65535 间任意的其他数字）

### Shadowsocks 客户端
> 相比服务器端的安装，客户端的安装就简单许多，大多数用户只需要这一步。具体文档参考：https://github.com/Shadowsocks-Wiki/shadowsocks

#### Mac 客户端 
- 下载地址 https://github.com/shadowsocks/ShadowsocksX-NG/releases/
- 运行 ShadowsocksX 后，menubar 显示纸飞机图标
- 服务器设定，手动添加服务器设置或扫描二维码（根据服务端配置，填写对应的客户端配置：服务器地址，端口，加密方法，密码）
- 连接代理服务器，成功建立连接后，便会创建一个 SOCKS5 代理：`127.0.0.1:1080`
- 开启系统代理（主菜单显示打开 shadowsocks），测试科学上网，如果系统代理运行成功，就可以访问 Google
- 从 GFWList 更新 PAC 文件，更新翻墙列表

「系统代理与 SOCKS5 代理」  
Shadowsocks 创建的系统代理将自动接管浏览器访问的全部请求，浏览器默认不需要任何设置，也无需安装代理插件，如果浏览器安装了代理插件，需要禁用代理插件或将代理插件设置为使用系统代理。SOCKS5 代理下，若不启用系统代理 shadowsocks 成功连接代理服务器后，仅创建了 SOCKS5 代理，浏览器需要安装代理插件并设置 shadowsocks 创建的 SOCKS5 代理端口，才能科学上网。

「PAC 代理模式（Proxy auto-config）」  
shadowsocks 成功连接代理服务器后会创建一个 SOCKS5 代理，系统代理是由 shadowsocks 客户端在 SOCKS5 上层实现的代理功能，支持两种代理模式：自动代理模式 (即 PAC 模式，默认）和全局代理模式。PAC 代理模式下，当浏览器访问某个网站时，会去匹配 PAC 配置文件里的 URL 列表。如果能匹配到 PAC 文件配置的 URL 就会使用 SOCKS5 代理访问该网站；否则不使用代理，直接访问网站。既节省 ss 流量，也会提高 国内网站的访问速度。PAC 列表一般会从 GFWList 更新（github.com/gfwlist），而 GFWList 定期会更新被墙的网站。

「浏览器代理插件」  
启用系统代理后，IE、Chrome 无需安装代理插件（浏览器默认设置使用的系统代理），就可以通过 shadowsocks 创建的系统代理科学上网了。使用系统代理时需要禁用浏览器的代理插件，或将其设置为使用系统代理。如果使用浏览器代理插件上网（比如 Proxy SwitchyOmega），可以关闭 shadowsocks 的系统代理，然后配置浏览器的代理插件，通过 shadowsocks 创建的 SOCKS5 代理来科学上网。

Chrome 安装 Proxy SwitchyOmega 插件之后，在选项菜单中修改情景模式中的 proxy 配置项：
- 代理协议 `SOCKS5`
- 代理服务器 `127.0.0.1`
- 代理端口 `1080`

<img alt="SwitchyOmega-1" src="https://ftp.bmp.ovh/imgs/2020/11/329f3854845b1c97.png" width="700">

继续修改情景模式中的 auto switch 配置项：
- 规则列表规则：匹配规则列表的请求选择使用 proxy 代理，默认情景模式选择直接连接
- 规则列表格式勾选 AutoProxy
- 规则列表网址 https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt

<img alt="SwitchyOmega-2" src="https://ftp.bmp.ovh/imgs/2020/11/2c5615c5045fc66c.png" width="700">

此时，proxy 为全局代理，所有访问请求全部通过 SOCKS5 代理；auto switch 为自动代理，根据 GFWlist 规则匹配自动代理上网。

#### iOS 客户端
- 美区 appstore 下载安装 potatso lite
- 扫码或手动填写配置，添加 shadowsocks 服务配置
- 启用智能路由
- 第一次连接 shadowsocks 服务会弹出是否允许 APP 在设置中添加 VPN 配置的对话框，点击允许
- 确认设置 — VPN 中默认使用的是 potatso lite 创建的 VPN 配置，此时可以访问 Google

## Shadowsocks 升级为 Trojan 服务
> 原 Shadowsocks 入门版服务升级至 Trojan Lite 服务，原 Shadowsocks 高级版和旗舰版服务升级至 Trojan Pro 服务。

### Trojan 客户端
ClashX，Trojan-QT5，TrojanX 均为 Mac 客户端，不同客户端主要是界面区别，选择一个使用即可。

#### TrojanX 设置方法
- TrojanX 的使用方法与 ShadowsocksX-NG 基本一致 (初次打开时，在应用程序文件夹中找到 TrojanX 右键打开，需要输入用户密码)
- 打开需要添加的节点的服务器配置页面，复制 Trojan 链接，然后点击客户端菜单中的从剪贴板导入服务器配置链接即可
- PAC 模式表示可以实现自动代理，即本来可以访问的网站不会经过代理，推荐日常使用。全局模式表示流量都会经过代理，不推荐日常使用。手动模式为不设置系统代理，浏览器需要配合扩展才可以使用
- TrojanX 默认提供的本地 Socks5 监听端口为 1080 

#### ClashX 设置方法
- 下载地址 https://github.com/yichengchen/clashX/releases
- 获取 Clash 的服务器订阅链接后，点击 menubar 图标添加配置：配置 > 托管配置 > 管理，Url 中填入订阅链接，Config Name 填写一个备注名称
- 点击设置为系统代理即可开启系统代理，此时即可正常使用
- 出站模式可以选择客户端的代理模式：全局模式/规则链接/直接连接
- 如果不使用客户端的系统代理，配合 SwitchyOmega 
- 需要修改为 Clash 对应的 7890 端口

<img alt="SwitchyOmega-clashX" src="https://ftp.bmp.ovh/imgs/2020/11/9070862ff6cb3281.png" width="700">

#### Spectre 设置方法
> Spectre VPN for iOS，请使用非中国区帐号在 App Store 下载安装

- 扫码添加单节点
- 第一次连接时，客户端会请求 VPN 设置权限，请选择允许
- 点击切换服务器进行添加或更换节点
- 可以选择使用全局模式或是规则模式，在规则设置中选择网址，参考规则列表网址: https://raw.githubusercontent.com/pexcn/daily/gh-pages/shadowrocket/whitelist.conf

## Proxy server configuration on Mac
When you configure a proxy server on your Mac, applications will send their network traffic through the proxy server before going to their destination. This may be required by your employer to bypass a firewall, or you may want to use a proxy to bypass geoblocking and access websites that aren’t available in your country.

In the network settings, if you want to configure the proxies used while connected to Wi-Fi networks, select “Wi-Fi”. If you want to configure the proxies used while connected to wired networks, click “Ethernet”.

- To have your Mac detect whether a proxy is necessary and automatically configure the proxy settings, enable the **Auto Proxy Discover** checkbox. Your Mac will use the Web Proxy Auto Discover protocol, or WPAD, to automatically detect whether a proxy is necessary.
- To use an automatic proxy configuration script, also known as a `.PAC` file, enable the **Automatic Proxy Configuration** checkbox. Enter the address of the script in the URL box.
- To manually configure a proxy, you’ll need to enable one or more of the **Web Proxy (HTTP)**, **Secure Web Proxy (HTTPS)**, **FTP Proxy**, **SOCKS Proxy**, and **Streaming Proxy (RTSP)** checkboxes. Enter the address and port number of the proxy for each option you enable.

The image below is proxy settings after enabling ClashX client (you don't need to configure them manually). If you don’t want to configure a proxy, ensure all these boxes are unchecked.

<img alt="proxy settings" src="https://ftp.bmp.ovh/imgs/2021/01/270e4181ab74221e.png" width="600">
