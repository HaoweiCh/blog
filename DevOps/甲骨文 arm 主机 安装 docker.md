---
title: 甲骨文 arm 主机 安装 docker
categories:
- DevOps
tags:
- 甲骨文
- arm64
- docker
date: 2022-03-03T15:10:01+08:00
year: 2022
week: 9
updated: 2022-03-04T15:15:43+08:00
---

背景: 甲骨文 ARM 主机真香， 

![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/365890D34B5C860E8DED818AEE5E33BE7D36937A.webp)

<!-- more -->

![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/7D94A120CA252BD19CC390020CB90D498D232B7E.webp)

* 缺点
  * google-chrome 没有
  * chromium 得自行编译
  * vmware virtualbox 都不行

## 部署 docker docker-compose

```bash

# 依赖， centos7 extra repos

wget http://mirror.centos.org/altarch/7/extras/aarch64/Packages/fuse-overlayfs-0.7.2-6.el7_8.aarch64.rpm && \ 
wget http://mirror.centos.org/altarch/7/extras/aarch64/Packages/slirp4netns-0.4.3-4.el7_8.aarch64.rpm && \
yum install -y ./fuse-overlayfs-0.7.2-6.el7_8.aarch64.rpm ./slirp4netns-0.4.3-4.el7_8.aarch64.rpm

yum install -y yum-utils gcc && \
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo && \
yum check-update && \
yum -y install docker-ce docker-ce-cli containerd.io && \
systemctl enable docker && systemctl start docker && systemctl status docker && \

pip3 install --upgrade pip && pip3 install docker-compose && \
docker --version & docker-compose --version
```

## 开机自动获取 IPv6 
> 目前在 arm 主机上 `nmtui` 命令使用异常， 依赖的 NetworkManager 系统服务也没有打开  
> 使用启动脚本解决 IPv6 地址分配问题


```bash
chmod +x /etc/rc.d/rc.local
echo "dhclient -6 enp0s3" >> /etc/rc.d/rc.local
```

* 通过 命令 `ip a` 查看公网出口适配器的名称
* 一般情况需要在服务提供商面板那里打开相应子网的 IPv6 开关

## 开启 GUI

两台主机， 一台被控一台主控
主控生成公私钥， 被控授权主控连接（通过公钥）
被控页面生成 vnc 连接命令
如

```
ssh -o ProxyCommand='ssh -W %h:%p -p 443 ocid1.instanceconsoleconnection.oc1.ap-singapore-1.anzwsljr5czrvhacszkolwzd3podek6brw2fmhubflzsey3ic4lhk5kbegsq@instance-console.ap-singapore-1.oci.oraclecloud.com' -N -L 127.0.0.1:5900:ocid1.instance.oc1.ap-singapore-1.anzwsljr5czrvhac3435osb55er4d5p2vruext4rvjkxlky5zp7khnne5oja:5900 ocid1.instance.oc1.ap-singapore-1.anzwsljr5czrvhac3435osb55er4d5p2vruext4rvjkxlky5zp7khnne5oja
```

* mac finder 自带的 vnc viewer 是没法直接连接的
  * 由于不支持未加密链接
* 需要 brew install vnc-viewer
  * Protocol error: invalid message type 255
    * 需要设置画面为 High (只要不是 automatic 就行， 自动有问题居然会报错，没回撤？)
* 虚拟机自身安装 x11vnc 体验还可