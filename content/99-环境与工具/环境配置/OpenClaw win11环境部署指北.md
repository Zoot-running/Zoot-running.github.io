OpenClaw这项目就是一大坨vibe coding出来的屎山，出于各种原因还需要使用，所以在win 11上部署一个作为尝试。
# 0. 失败的尝试
首先排除一个错误选项。
不要用官网的那个命令直接在win11上部署OpenClaw！！！非常的不稳定，会出现奇怪的1006无原因的错误，没有必要和这个错误搏斗。
建议还是部署一个Linux版本的，win11环境下使用wsl2即可。
# 1. 准备工作
需要做的准备工作又两项：
1) 安装wsl2，参考[官方指南]([安装 WSL | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/wsl/install))，后续可以自己选择发行版，按理说没有发行版的区分，本次部署使用的是Ubuntu 22.04；
2) 安装nodejs，手工安装nodejs的LTS版本会获得一个相对稳定运行的OpenClaw，OpenClaw要求22+，目前提供的LTS是24+的，实测可以使用，安装参考[官方下载](https://nodejs.org/en/download)，官方nvm下载命令：
```bash
# Download and install nvm:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.4/install.sh | bash

# in lieu of restarting the shell
\. "$HOME/.nvm/nvm.sh"

# Download and install Node.js:
nvm install 24

# Verify the Node.js version:
node -v # Should print "v24.14.0".

# Verify npm version:
npm -v # Should print "11.9.0".

```
# 2. 安装OpenClaw
使用官方脚本安装
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```
如果需要github，那就申请github账户并且配置公钥，正常github是没有被墙的。
# 3. OpenClaw基础配置
让OpenClaw有一些基础的功能并且能够做基础的设备管理，需要配置两个功能
## 3.1 模型配置
在安装完成后有可能直接进入引导，跟着引导走（能跳过都跳过），会要求配置模型，这时候就需要你又某个模型的API key了。这里以kimi k2.5为例。
首先[注册账户](https://platform.moonshot.cn/console/account)，注册后会有15块钱的Token赠送，用赠送的Token做验证。注意，观察一下自己的账户余额，在赠送Token到位后再进行操作。
之后进入API Key管理，选择新建，Key只会显示一次，要保存好。
然后，在引导中选择moonshot的模型（K2.5），选择.cn的后缀（中国国内），然后填入API Key即可。完成引导，你的网关就应该启动了，可以通过127.0.0.1:18789来访问它了。
如果安装完成没有出现引导，可以使用`openclaw onboard`来唤起引导。
配置完成后就一个可以和OpenClaw对话了，一些问题就可以让它动手解决了。
## 3.2 配置飞书
[配置参考](https://docs.openclaw.ai/zh-CN/channels/feishu)
这个配置文档是基本没有问题的，几个点需要注意：
1) 插件没有必要下载，通过脚本安装的OpenClaw已经集成了飞书的插件，可能会缺依赖，通过web UI让小龙虾自己分析就能解决；
2) 事件配置是依赖于网关与应用建立连接的，所以出现问题也没事，先发布应用第一个版本。然后，回到OpenClaw配置好渠道和APP ID和key，重启网关，它会建立连接，之后就能够搞定正确的事件配置了
3) 注意第一条消息发送后会有一个配对码生成，需要批准这个配对码，可以命令行，也可以小龙虾自己干；
4) 权限可以按照文档给，也可以把一级权限全部授予，看使用场景，如果当前只是验证，可以考虑给的奔放一点。