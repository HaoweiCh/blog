---
title: arm64 版本的 Windows 11 (Parallels) 部署 SSH Daemon
categories:
- DevOps
tags:
- Windows 11
- windows
- Parallels
- arm64
- apple silicon
- apple
date: 2022-03-02T19:11:42+08:00
year: 2022
week: 9
updated: 2022-03-03T19:26:53+08:00
---

背景：不想浪费时间在界面点击上面，终端连接 windows 虚拟机实现命令行自动化操作

![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/B5DA2AB9E1DF4EF608484E1C3862019735050C66.webp)

<!-- more -->

## 1. 安装 ssh server (sshd)


先贴上官方文档, 按步骤安装 OpenSSH Server

* https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse
  * 英文
* https://docs.microsoft.com/zh-cn/windows-server/administration/openssh/openssh_install_firstuse
  * 中文

![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/DF70B2A966E28739D2E6A3759A31CA87C820DCF0.webp)

## 2. 配置服务

```shell
# Start the sshd service 启动服务
Start-Service sshd

# OPTIONAL but recommended: 设置自动启动服务
Set-Service -Name sshd -StartupType 'Automatic'

# Confirm the Firewall rule is configured. It should be created automatically by setup. Run the following to verify 确保防火墙相关规则已建立
if (!(Get-NetFirewallRule -Name "OpenSSH-Server-In-TCP" -ErrorAction SilentlyContinue | Select-Object Name, Enabled)) {
    Write-Output "Firewall Rule 'OpenSSH-Server-In-TCP' does not exist, creating it..."
    New-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
} else {
    Write-Output "Firewall rule 'OpenSSH-Server-In-TCP' has been created and exists."
}
```


## 3. 添加公钥到信任

https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_server_configuration#authorizedkeysfile


!!! 管理员配置文件位置在 %programdata%

### !!!密钥文件属性
> 无法使用密钥免密登陆看这里


尝试通过模块工具修复文件属性

`Install-Module -Force OpenSSHUtils -Scope AllUsers` .

修复文件属性命令: 

`Repair-AuthorizedKeyPermission -FilePath C:\Users\admin\.ssh\authorized_keys`

结果模块在 Windows 11 上安装不了

修改配置文件关键属性 `StrictModes no`
保存并更新配置文件

`Restart-Service sshd`

重启服务了事，visual studio code 远程修改舒服多了，终于不用所有操作都点点点了

## 最后效果

如首页图

![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/B5DA2AB9E1DF4EF608484E1C3862019735050C66.webp)