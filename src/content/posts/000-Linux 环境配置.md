---
title: 000-Linux 环境配置
published: 2026-01-01
description: 关于XXX的笔记
image: api
tags:
  - 主线
category: 全栈开发基础
---
参考链接: https://blog.csdn.net/daizhe/article/details/151224325

# VMware Fusion + Ubuntu 24.04 Server 配置方案

## 一、准备工作

### 1.1 下载 Ubuntu Server ARM64 镜像

下载 ARM64 版本的 Ubuntu Server：

```bash
# 使用 Homebrew 下载（可选）
brew install wget
wget - https://mirrors.ustc.edu.cn/ubuntu-cdimage/releases/24.04.2/release/ubuntu-24.04.2-live-server-arm64.iso
```

或者直接从官网下载：
- 访问：https://cn.ubuntu.com/download/server/arm
- 选择 **Ubuntu 24.04.2 LTS** 版本
- 文件名：`ubuntu-24.04.2-live-server-arm64.iso` 

> **注意**：必须下载 `-arm64` 版本，标准 `amd64` 版本无法在 M 系列芯片上运行。Server 版默认不带图形界面，适合学习 Linux 命令行和服务器管理 。

### 1.2 下载 VMware Fusion

VMware Fusion 对个人用户免费，需要从官网获取：

1. 访问 Broadcom 官网并注册账户
2. 下载 **VMware Fusion 13.6.1** 或更新版本
3. 文件名：`VMware-Fusion-13.6.1-xxxxx_universal.dmg` 

安装完成后，打开 VMware Fusion 并同意许可协议。

---

## 二、创建虚拟机

### 2.1 新建虚拟机

1. 打开 VMware Fusion
2. 点击 **"新建"** 或 **File → New**
3. 选择 **"从光盘或映像中安装"** → **继续**
4. 将下载的 ISO 文件拖入窗口，或点击"选择"手动定位
5. 点击 **"继续"**
.
### 2.2 配置虚拟机资源

| 配置项 | 建议值 | 说明 |
|--------|--------|------|
| 处理器核心 | 2-4 核 | M4 Pro 性能充裕，2 核已足够学习使用 |
| 内存 | 4-8 GB | 学习 Linux 基础 4GB 够用 |
| 硬盘 | 40-80 GB | 默认 40GB 即可，后续可扩展  |

配置完成后，点击 **"完成"**，可以自定义虚拟机名称（如 `Ubuntu-Server`）。

---

## 三、安装 Ubuntu Server 系统

### 3.1 启动虚拟机并选择安装

1. 点击虚拟机窗口的 **"启动"** 按钮
2. 在启动菜单中选择 **"Try or Install Ubuntu Server"**（使用键盘方向键选择，按回车确认）
3. 等待系统加载

### 3.2 安装步骤

| 步骤       | 操作                                                                                   |
| -------- | ------------------------------------------------------------------------------------ |
| **语言选择** | 选择 **English**（推荐）或中文                                                                |
| **安装类型** | 选择 **Ubuntu Server**（最小化安装）                                                          |
| **网络配置** | 保持 DHCP 默认，直接选择 **Done**（后续可改为静态 IP）                                                 |
| **代理配置** | 留空，选择 **Done**                                                                       |
| **镜像源**  | 国内用户建议改为清华源：`https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports`，直接 **Done** 跳过则使用官方源 |
| **磁盘分区** | 选择 **Use an entire disk** → **Done**（默认即可）                                           |
| **磁盘确认** | 选择 **Continue**                                                                      |

### 3.3 用户信息设置

| 字段 | 说明 | 示例 |
|------|------|------|
| **Your name** | 你的全名 | `zhang san` |
| **Your server's name** | 主机名 | `ubuntu-server` |
| **Pick a username** | 登录用户名 | `zhang` |
| **Choose a password** | 密码 | 自行设置 |

### 3.4 SSH 服务配置（重要）

在 **SSH Setup** 页面：
- **用空格键选中** `Install OpenSSH server`
- 其他选项保持默认，选择 **Done** 

> 这一步很关键：如果不安装 OpenSSH Server，后续无法通过 Mac 终端远程连接虚拟机。

### 3.5 完成安装

1. 等待安装完成（约 5-10 分钟）
2. 选择 **"Reboot Now"** 重启
3. 重启后如遇 "Please remove installation medium" 提示，按回车即可 

---

## 四、系统基础配置

### 4.1 登录系统

系统启动后，使用安装时设置的用户名和密码登录。

### 4.2 更换国内软件源（可选，推荐）

Ubuntu 24.04 使用新的 DEB822 格式源配置文件，路径为 `/etc/apt/sources.list.d/ubuntu.sources` ：

```bash
# 备份原配置
sudo cp /etc/apt/sources.list.d/ubuntu.sources /etc/apt/sources.list.d/ubuntu.sources.bak

# 编辑配置文件
sudo vim /etc/apt/sources.list.d/ubuntu.sources
```

按 `i` 进入编辑模式，将内容替换为：

```
Types: deb
URIs: https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Types: deb
URIs: http://ports.ubuntu.com/ubuntu-ports/
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

> **注意**：URIs 中必须是 `ubuntu-ports`，这是 ARM 架构的源地址。如果写成 `ubuntu` 会导致安装软件时报 `Unable to locate package` 错误 。

按 `ESC`，输入 `:wq` 保存退出。

### 4.3 更新软件包

```bash
sudo apt update
sudo apt upgrade -y
```

---

## 五、配置 SSH 远程访问（推荐）

安装时已勾选 OpenSSH Server，但还需要确认服务状态并允许连接。

### 5.1 查看虚拟机 IP 地址

```bash
ip addr show
# 或简写
ip a
```

记录下 IP 地址（通常是 `ens33` 或 `eth0` 接口下的 `inet` 值，如 `192.168.x.x`）。

### 5.2 检查 SSH 服务状态

```bash
sudo systemctl status ssh
```

如果状态为 `inactive`，执行启动命令：

```bash
sudo systemctl start ssh
sudo systemctl enable ssh   # 设置开机自启
```

### 5.3 配置防火墙（如已启用）

```bash
# 查看防火墙状态
sudo ufw status

# 如果防火墙已启用，允许 SSH 端口
sudo ufw allow ssh
```

### 5.4 Mac 终端远程连接

在 Mac 终端中执行：

```bash
ssh 用户名@虚拟机IP地址
# 例如：ssh zhang@192.168.1.100
```

输入密码后即可远程操作 Ubuntu，无需打开虚拟机窗口 。

---

## 六、安装 VMware Tools 增强工具

VMware Tools 可以优化虚拟机性能，支持共享剪贴板、自动调整分辨率等功能 。

```bash
# 安装 open-vm-tools（开源版本，推荐）
sudo apt install open-vm-tools -y

# 如果后续安装了桌面环境，还需安装桌面增强工具
sudo apt install open-vm-tools-desktop -y
```

安装完成后重启：

```bash
sudo reboot
```

### 6.1 配置共享目录（可选）

1. **虚拟机设置**：VMware Fusion 菜单栏 → **虚拟机** → **设置** → **共享**
2. 点击 **"+"** 添加要共享的 Mac 文件夹
3. 在 Ubuntu 中创建挂载点并配置自动挂载：

```bash
# 创建挂载目录
sudo mkdir -p /mnt/hgfs

# 编辑 fstab 实现开机自动挂载
sudo vim /etc/fstab
```

添加以下内容：

```
.host:/ /mnt/hgfs fuse.vmhgfs-fuse allow_other,defaults 0 0
```

保存后，重启或执行 `sudo mount -a`，共享目录将出现在 `/mnt/hgfs` 下 。

---

## 七、可选：安装桌面环境

Server 版默认无图形界面。如果后续需要桌面环境，可以安装 Ubuntu Desktop：

```bash
# 安装 Ubuntu 桌面
sudo apt install ubuntu-desktop -y

# 安装完成后重启
sudo reboot
```

> **注意**：安装桌面环境会占用较多磁盘空间（约 5-10GB）和内存资源。学习 Linux 建议优先使用命令行 。

---

## 八、常用命令速查

| 操作 | 命令 |
|------|------|
| 更新软件源 | `sudo apt update` |
| 升级软件包 | `sudo apt upgrade` |
| 查看 IP 地址 | `ip a` |
| 查看 SSH 服务状态 | `sudo systemctl status ssh` |
| 重启虚拟机 | `sudo reboot` |
| 关闭虚拟机 | `sudo poweroff` |

---

## 九、注意事项

1. **ARM 架构兼容性**：所有安装的软件包都需要 ARM64 版本，Ubuntu 官方源已提供大部分常用软件 。

2. **网络模式**：默认使用 NAT 模式，虚拟机通过 Mac 共享网络，可以访问互联网。如需与局域网其他设备互通，可改为桥接模式 。

3. **快照功能**：VMware Fusion 支持快照，在进行重要配置前（如安装服务、修改系统文件），建议先拍摄快照以便快速恢复 。

4. **SSH 连接被拒**：如遇 "Connection refused"，检查 SSH 服务是否启动（`systemctl status ssh`）和防火墙设置 。

---

## 十、学习资源推荐

完成以上配置后，你已经拥有一个完整的 Linux 学习环境。建议按以下顺序学习：

1. Linux 基础命令（`ls`、`cd`、`cp`、`mv`、`rm`、`grep`、`find`）
2. 文件权限管理（`chmod`、`chown`）
3. 用户与用户组管理
4. 进程管理（`ps`、`top`、`systemctl`）
5. 网络配置与诊断（`ip`、`ping`、`ss`）
6. Shell 脚本编程
7. 软件包管理（`apt`）
8. 服务配置（Nginx、MySQL 等）

详见笔记[[Linux]]
