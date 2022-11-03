---
title: k8s 部署笔记
description: k8s 部署笔记
categories:
- DevOps
tags:
- Kubernetes
- K8s
date: 2022-01-01T10:54:14+08:00
year: 2021
week: 52
updated: 2022-01-01T11:11:03+08:00
---

![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/C88D2A8823D636B887A391D4DDB2C5B5FD866425.webp)

<!-- more -->

1. 环境部署
    1. 配置 hostname
    2. 安装 docker
    3. 安装基础工具 kubeadm kubelet
    4. 关闭 swap
    5. 关闭防火墙如 firewalld
    6. 开启所有worker及master的路由模式
        * echo net.ipv4.ip_forward=1 >> /etc/sysctl.conf && sysctl -p
    7. 同步时间 ntp 设置时区 
        *  yum install ntp && systemctl start ntpd.service
        *  timedatectl set-timezone Asia/Shanghai
2. 拉 k8s 依赖镜像
3. 修改 kubelet cgroup-driver
4. `kubeadm init`  这里需要确定具体的参数
    1. helm 安装 calico
5. helm 安装 dashboard

## 部署

> (Recommended) If you have plans to upgrade this single control-plane kubeadm cluster to high availability you should specify the --control-plane-endpoint to set the shared endpoint for all control-plane nodes.
---
> 建议：如果有计划为可用性升级该单节点部署，应该通过参数 `--control-plane-endpoint` 指定端点给控制平面

### 国内部署环境问题

镜像站

`https://feisky.gitbooks.io/kubernetes/content/appendix/mirrors.html`

```
# GCR（Google Container Registry）镜像
for i in `kubeadm config images list`; do 
  imageName=${i#k8s.gcr.io/}
  docker pull registry.aliyuncs.com/google_containers/$imageName
  docker tag registry.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
  docker rmi registry.aliyuncs.com/google_containers/$imageName
done;
```

```
for i in `kubeadm config images list`; do 
  imageName=${i#k8s.gcr.io/}
  docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
  docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
  docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
done;
```

```
docker pull coredns/coredns:1.8.4
docker tag docker.io/coredns/coredns:1.8.4 k8s.gcr.io/coredns/coredns:v1.8.4
```

### 端口开放列表

6443
30000-32767

![](media/16315233000811/16315262333003.jpg)

### 基础工具安装 kubeadm kubelet
* kubeadm
    * https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
* kubelet

Letting iptables see bridged traffic
Make sure that the br_netfilter module is loaded. This can be done by running lsmod | grep br_netfilter. To load it explicitly call sudo modprobe br_netfilter.

As a requirement for your Linux Node's iptables to correctly see bridged traffic, you should ensure net.bridge.bridge-nf-call-iptables is set to 1 in your sysctl config, e.g.

```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system
```

```

cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

# Set SELinux in permissive mode (effectively disabling it)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

systemctl enable --now kubelet
```

#### `kubeadm`

kubeadm is network provider-agnostic
`kubeadm` 网络组件不定提供

kubeadm.yaml

#### kubelet
> 会在 join 后才能正常启动
> 
> The kubelet is now restarting every few seconds, as it waits in a crashloop for kubeadm to tell it what to do. This crashloop is expected and normal, please proceed with the next step and the kubelet will start running normally.



docker info |grep Cgroup
vi /etc/sysconfig/kubelet
修改 KUBELET_EXTRA_ARGS=--cgroup-driver=cgroupfs
systemctl show --property=Environment kubelet |cat

systemctl enable --now kubelet

##### 处理启动失败的情况



```
journalctl -xefu kubelet
```

### 平台组件

* https://kubernetes.io/docs/concepts/cluster-administration/addons/



```
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

```

#### 网络组件
> 使每一个 pod 都有全集群唯一的虚拟IP地址

coredns (or kube-dns) should be in the Pending state until you have deployed the network add-on.

* https://tech.ipalfish.com/blog/2020/03/06/kubernetes_container_network/

---

* 实现方式
    * overlay network 覆盖性网络 
    * non-overlay 非覆盖性
* BGP 边界网关协议
    

---

* Flannel 
    * 被公认为是最简单的选择
* Calico 
    * BGP boarder gateway protocol 边界网关协议
    * 以其性能、灵活性而闻名
    * 在大规模集群中可以通过额外的 BGP route reflector 来完成
    * 基于 iptables 还提供了丰富的网络策略
    * 实现了 Kubernetes 的 Network Policy 策略(提供容器间网络可达性限制的功能)
* OVS 
    * Open VSwitch
    * 相对 calico 性价比比较高
    * 相对 calico 稳定性更好

##### Falnnel

https://github.com/flannel-io/flannel#deploying-flannel-manually

```
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

```

---

* *镜像国内服务器拉不下来，需要处理镜像地址*
    * 上传到 dockerhub
    * 上传到阿里云镜像仓库（目前免费）

```
docker pull quay.io/coreos/flannel:v0.14.0
docker tag quay.io/coreos/flannel:v0.14.0 registry.cn-chengdu.aliyuncs.com/haowei_ch/flannel:v0.14.0
docker push registry.cn-chengdu.aliyuncs.com/haowei_ch/flannel:v0.14.0

# 自行替换文件里的镜像地址
```
---

```

kubectl apply -f kube-flannel.yml
kubectl get pods -n kube-system ｜grep flannel
```

###### kube-flannel.yml

```

```

##### Calico
> Calico is a networking and network policy provider. 
> Calico supports a flexible set of networking options so you can choose the most efficient option for your situation, including non-overlay and overlay networks, with or without BGP. 
> Calico uses the same engine to enforce network policy for hosts, pods, and (if using Istio & Envoy) applications at the service mesh layer

https://docs.projectcalico.org/latest/introduction/
https://docs.projectcalico.org/getting-started/kubernetes/quickstart

* calico 默认模式是 node to node mesh 需要注意集群节点数量
    * Full-mesh works great for small and medium-size deployments of say 100 nodes or less
    * https://docs.projectcalico.org/networking/bgp#full-mesh
* route reflectors

**一定要修改 node ip detection method **

```
kubectl edit configmap/coredns -n kube-system
```

###### 卸载

```
kubectl delete --grace-period=0 --force -f https://docs.projectcalico.org/manifests/tigera-operator.yaml

rm -f /etc/cni/net.d/10-calico.conflist
rm -f /etc/cni/net.d/calico-kubeconfig

```

###### 通过 helm 安装

https://docs.projectcalico.org/getting-started/kubernetes/helm

```
helm repo add projectcalico https://docs.projectcalico.org/charts

helm show values projectcalico/tigera-operator --version v3.20.1

helm install calico projectcalico/tigera-operator --version v3.20.1

watch kubectl get pods -n calico-system
```




#### Dashboard 控制台

https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

```
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml

# 修改相关信息

kubectl apply -f kube-dashboard.yml
```

```
# 查看容器状态
kubectl get pods -n kubernetes-dashboard
```

删除
切换 ns 到 kubernetes-dashboard
`k delete $(k get pod -o name | grep dashboard)`
`k delete deploy dashboard-metrics-scraper kubernetes-dashboard`

使用

Warning: spec.template.metadata.annotations[seccomp.security.alpha.kubernetes.io/pod]: deprecated since v1.19; use the "seccompProfile" field instead

```
kubectl proxy
http://127.0.0.1:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```


##### helm 安装

```
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
helm install dashboard kubernetes-dashboard/kubernetes-dashboard
```

`kubectl proxy`

open url

    http://127.0.0.1:8001/api/v1/namespaces/default/services/https:dashboard-kubernetes-dashboard:443/proxy/#/login


#### Descheduler 重平衡工具

* 已实现的调度策略
    * RemoveDuplicates 移除重复 pod
    * LowNodeUtilization 节点低度使用
    * RemovePodsViolatingInterPodAntiAffinity 移除违反pod反亲和性的 pod
    * RemovePodsViolatingNodeAffinity

### 多集群联邦

* [Kubernetes 多集群管理](https://mp.weixin.qq.com/s?__biz=MzIzNTU1MjIxMQ==&mid=2247483886&idx=1&sn=d397c17088a6a5c2516d7a77acb961e6&chksm=e8e42d52df93a44416c4f250c581158e15d44ba17bc11f5abcd310d59bf9c9fe5fef5aa0e4b8&scene=21#wechat_redirect)
* https://github.com/kubernetes-sigs/kubefed

### 日志聚合

https://kubernetes.io/zh/docs/concepts/cluster-administration/logging/

目前的解决方案

filebreat => logstash => graylog => elasticsearch

#### rotate

需要 rotate 日志

k8s 默认脚本部署的话,存在一个 logrotate，每小时运行一次。 kubeadm 需要手动设置容器运行时来自动地轮转应用日志。

https://www.elastic.co/guide/en/beats/filebeat/7.14/_log_rotation.html

## 维护

https://kubernetes.io/docs/tasks/administer-cluster/

### 节点动态加入

```
Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 139.9.33.44:6443 --token n6t4er.xtc9vaop9evvdqca \
	--discovery-token-ca-cert-hash sha256:2a2c106c94414a23a43f8cd4a4c96120c91792c0837a90f36655f8f56e96aff4
```

### CLI 工具

#### `kubectx` `kubens`

* https://github.com//ahmetb/kubectx

kubectx + kubens: Power tools for kubectl

命令行切换 ctx 或 namespace 
           上下文（集群） 或 命名空间

### 证书管理
https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/

```
kubeadm certs check-expiration
```

### 升级集群

https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

### 配置修改

```
kubectl -n kube-system get configmap kubeadm-config -o jsonpath='{.data.ClusterConfiguration}' > kubeadm.cfg.yaml
```

#### 添加证书许可DNS
> https://blog.scottlowe.org/2019/07/30/adding-a-name-to-kubernetes-api-server-certificate/


修改 kubeadm.cfg.yaml

```
apiServer:
  certSANs:
  - "139.9.33.44"
  - "127.0.0.1"
```

移动现有 cert （kubeadm 不支持覆盖）
`mv /etc/kubernetes/pki/apiserver.{crt,key} .`

重新生成
`kubeadm init phase certs apiserver --config kubeadm.cfg.yaml`

强制重启 api server
`docker ps | grep kube-apiserver | grep -v pause`

docker stop {ID}

## 使用

### 合并 kube config

```
KUBECONFIG=file1:file2:file3 kubectl config view --merge --flatten > out.txt
KUBECONFIG=config:zbanx.local kubectl config view --merge --flatten > config

# 拆分
KUBECONFIG=in.txt kubectl config view \
    --minify --flatten --context=context-1 > out.txt
```