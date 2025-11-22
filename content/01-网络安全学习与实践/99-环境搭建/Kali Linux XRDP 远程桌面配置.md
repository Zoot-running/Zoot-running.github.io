
## 一、前提准备

1. **环境说明**：阿里云 ECS 实例（Kali Linux 自定义镜像）、本地 Windows 11 电脑
2. **核心目标**：通过 Windows 远程桌面（RDP）连接 Kali，实现图形化管理，支持自定义分辨率和快捷登录
3. **注意事项**：确保实例安全组开放 **3389/TCP** 端口（RDP 默认端口）
## 二、系统初始化与软件源配置
### 1. 登录服务器
通过阿里云控制台 **远程连接** 或 SSH 工具登录 Kali（默认用户：`kali`，密码为实例初始化时设置的密码）。
### 2. 修复软件源（解决更新报错）
Kali 默认源可能存在访问缓慢或 404 问题，替换为国内镜像源：
```bash
\# 备份原有源文件

sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak

\# 编辑源文件（使用 nano 编辑器）

sudo vi /etc/apt/sources.list
```
注释原文件中所有内容，粘贴以下 **阿里云镜像源**：
`deb http://mirrors.aliyun.com/kali kali-rolling main non-free contrib`
### 3. 系统更新与依赖修复
```bash
\# 清理旧缓存

sudo apt clean

\# 更新软件包列表

sudo apt update

\# 修复依赖并升级系统（若遇到 unrar 等包报错，添加 --fix-missing）

sudo apt full-upgrade -y --fix-missing
```
**说明**：若更新过程中出现 `sshd_config` 配置冲突，选择 **保留本地版本**（阿里云默认修改了允许 root 登录和密码认证，避免无法远程 SSH）。
## 三、XRDP 安装与基础配置
### 1. 安装 XRDP 及桌面环境
XRDP 依赖 Xfce 桌面环境（轻量稳定，适合远程使用）：
```bash
sudo apt install xrdp xfce4 xorgxrdp -y
```
### 2. 配置 Xfce 桌面默认启动
XRDP 需要指定登录后加载的桌面环境，创建 `.xsession` 文件（根据登录用户选择路径）：
#### 若登录用户为 `kali`（推荐）：
```bash
\# 创建并编辑配置文件

vi /home/kali/.xsession

\# 写入以下内容（指定启动 Xfce 桌面）

xfce4-session
```
#### 若登录用户为 `root`：
```bash
vi /root/.xsession

echo "xfce4-session" > /root/.xsession
```
保存退出后，确保文件权限正确：
```bash
\# 给 kali 用户添加读写权限

sudo chown kali:kali /home/kali/.xsession

chmod u+r /home/kali/.xsession
```
### 3. 启动并设置 XRDP 开机自启
```bash
\# 启动 XRDP 服务

sudo systemctl start xrdp

\# 设置开机自启

sudo systemctl enable xrdp

\# 检查服务状态（显示 active (running) 即为正常）

sudo systemctl status xrdp
```
### 4. 配置阿里云安全组（关键步骤）
1. 登录阿里云 ECS 控制台，进入实例详情页 → **网络与安全** → **安全组**。
2. 选择实例关联的安全组，点击 **入方向** → **手动添加**。
3. 配置规则：
* 端口范围：`3389/3389`
* 授权对象：`0.0.0.0/0`（允许所有 IP 访问，若需限制，填写本地公网 IP）
* 描述：`允许 RDP 远程连接`
4. 保存规则。
## 四、分辨率与显示优化
### 1. 问题描述
默认分辨率可能过高或过低，导致远程桌面显示异常（如窗口过小、无法全屏）。
### 2. 解决方案：修改 XRDP 配置文件
```bash
\# 编辑 XRDP 主配置文件

sudo nano /etc/xrdp/xrdp.ini
```
找到 `[Xorg]` 段落，添加 / 修改以下参数：
```bash
\[Xorg]
name=Xorg
lib=libxrdp.so  # 这个库真实配置会有差异，还没研究具体含义
username=ask
password=ask
ip=127.0.0.1
port=-1
screen\_width=1366  # 基础分辨率（根据需求调整，如 1920x1080）
screen\_height=768
smart\_sizing=true  # 开启自适应分辨率（核心！支持窗口放大/全屏）
```
保存退出后，重启 XRDP 服务：
```bash
sudo systemctl restart xrdp
```
如果重启服务导致无法连接，考虑重启整个系统。
## 五、Windows 端快捷方式创建（一劳永逸）
每次手动输入 IP 和设置分辨率繁琐，创建快捷方式实现双击登录：
### 1. 创建快捷方式
1. 在 Windows 桌面空白处 → 右键 → **新建** → **快捷方式**。
2. 输入对象位置（替换为你的服务器公网 IP）：
```
mstsc.exe /v:123.45.67.89 /f
```
本来还有/scale:125参数，但是会报错考虑先去掉。
**参数说明**：
* `/v:``123.45.67.89`：指定服务器 IP（替换为实际公网 IP）
* `/f`：全屏模式（连接后自动全屏）
* `/scale:125`：缩放比例（根据需求调整，如 100/150）
点击 **下一步**，命名为「连接 Kali 服务器」，点击 **完成**。
### 2. 保存登录凭据（可选）
1. 双击快捷方式，在弹出的远程桌面窗口中 → 点击 **显示选项**。
2. 输入登录用户名（如 `kali`），勾选 **允许我保存凭据**。
3. 点击 **连接**，输入密码后勾选 **记住我的凭据**，下次双击即可自动登录。
## 六、常见问题排查
### 1. XRDP 启动失败（日志报错 “Could not start log”）
* 原因：日志文件权限不足或路径错误。
* 解决：
```bash
\# 创建日志文件并授权
sudo touch /var/log/xrdp.log
sudo chown xrdp:xrdp /var/log/xrdp.log
sudo systemctl restart xrdp
```
### 2. 远程连接后黑屏 / 闪退
* 原因：`.xsession` 文件配置错误或权限不足。
* 解决：重新检查 `/home/kali/.xsession` 文件，确保内容为 `xfce4-session`，且权限正确。
### 3. 分辨率调整后仍无法全屏
* 原因：Windows 远程桌面未开启全屏模式。
* 解决：连接后按 `Ctrl + Alt + Break` 快捷键切换全屏 / 窗口模式。
## 七、总结
通过以上步骤，完成了 Kali Linux XRDP 远程桌面的完整配置，实现了：
1. 系统更新与软件源优化（国内镜像，更新快速稳定）。
2. XRDP 安装与 Xfce 桌面配置（图形化管理）。
3. 分辨率自适应与显示优化（支持窗口放大 / 全屏）。
4. Windows 端快捷方式创建（双击自动登录，无需重复设置）。
如需后续优化，可安装常用工具（如 Firefox、Terminator）：
```bash
sudo apt install firefox-esr terminator -y
```