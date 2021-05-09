---
title: "vscode 远程开发"
date: 2019-10-06T15:39:15+08:00
summary: "Vscode remote development"
category: vscode
tags:
    - vscode
    - remote
    - ssh
---

Vscode remote development 是个非常强大的功能，它可以通过 ssh 连接远程的主机进行开发

## 它支持以下几种类型

- SSH
- WSL 即适用于 Windows 的 Linux 子系统
- Container


## Windows 下面使用步骤

### ssh 认证
安装 Openssh 以产生密钥，与远程主机进行验证，在 Windows 可选功能中安装

- 打开 Windows 设置 --> 应用 --> 可选功能 --> 添加openssh客户端
- 打开 Powershell 生成密钥，并上传到主机

```psh
PS > ssh-keygen             # 生成密钥
PS > ssh-copy-id user@host  # 将密钥复制到远程主机
```

### 安装 Remote development
在 Vscode 扩展中搜索安装 `remote development`，并进行如下配置

- 快捷键 `ctrl+shift+p`
  输入 `remote-ssh`
  可看见 `open configuration file...`
  打开配置主机的连接信息

```config
Host cli-server
    HostName remote-server     # 远程主机的 IP 或域名
    Port 22                    # ssh端口，默认 22
    User username              # 登录用户名
```

- 再次输入 `remote-ssh` 打开 `Connect to Host...` 开始连接主机


## 结语
经过一段时间的使用发现非常方便，我可以远程到 VPS 或者，本地的 Ubuntu server。有些文件格式用 vim 还是不太方便，用 Vscode 就非常好了 😁