---
title: vagrant安装kubernetes集群
date: 2017-06-16 13:39:00
tags:
  - docker
  - kubernetes
description: 容器时代，我们谈的最多的是如何利用docker将现有的单体应用架构转变成微服务架构，实施微服务，我们必须借助一个编排工具来管理dokcer。目前主流的docker编排工具有 Docker Swarm， Mesos 和 kubernetes。本文将介绍在centos7虚拟机下安装kubernetes集群。
---
## 前言
容器时代，我们谈的最多的是如何利用docker将现有的单体应用架构转变成微服务架构，实施微服务，我们必须借助一个编排工具来管理dokcer。目前主流的docker编排工具有 Docker Swarm， Mesos 和 kubernetes。本文将介绍在centos7虚拟机下安装kubernetes集群。
## 一. 环境说明
#### 1. 虚拟机系统版本
三台虚拟机均为centos7
#### 2. 需要安装的软件列表软件列表

*  Kubernetes
*  docker
*  flanne
*  etcd

#### 3 节点列表IP
*  master节点 : 192.168.118.120
*  node节点1  : 192.168.118.121
*  node节点2  : 192.168.118.122

## 二. 安装和配置
### 1. 关闭每个节点的selinux
-  永久关闭， `cat /etc/selinux/config`
```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```
- 临时关闭
```
setenforce 0
```

### 2. 配置master节点
- 安装软件  
```
sudo yum install -y docker kubernetes-master etcd
```
- etcd配置文件：`egrep -v '^#' /etc/etcd/etcd.conf`
```
ETCD_NAME=default
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="http://192.168.118.120:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_INITIAL_ADVERTISE_PEERURLS="http://192.168.118.120:2380"
ETCD_INITIAL_CLUSTER="default=http://192.168.118.120:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.118.120:2379"
```
- 初始化flannel的etcd配置
```bash
 etcdctl -C 192.168.118.120:2379 set /coreos.com/network/config '{ "Network": "10.1.0.0/16" }'
```
- `cat /etc/kubernetes/config`
```
###
# kubernetes system config
#
# The following values are used to configure various aspects of all
# kubernetes services, including
#
#   kube-apiserver.service
#   kube-controller-manager.service
#   kube-scheduler.service
#   kubelet.service
#   kube-proxy.service
# logging to stderr means we get it in the systemd journal
KUBE_LOGTOSTDERR="--logtostderr=true"
# journal message level, 0 is debug
KUBE_LOG_LEVEL="--v=0"
# Should this cluster be allowed to run privileged docker containers
KUBE_ALLOW_PRIV="--allow-privileged=false"
# How the controller-manager, scheduler, and proxy find the apiserver
KUBE_MASTER="--master=http://192.168.118.120:8080"
```
- `cat /etc/kubernetes/apiserver`
```
###
# kubernetes system config
#
# The following values are used to configure the kube-apiserver
#
# The address on the local server to listen to.
KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0"
# The port on the local server to listen on.
 KUBE_API_PORT="--port=8080"
# Port minions listen on
# KUBELET_PORT="--kubelet-port=10250"
# Comma separated list of nodes in the etcd cluster
KUBE_ETCD_SERVERS="--etcd-servers=http://192.168.118.120:2379"
# Address range to use for services
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"
# default admission control policies
KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota"
# Add your own!
KUBE_API_ARGS=""
```
- `cat /etc/kubernetes/controller-manager`
```
###
# The following values are used to configure the kubernetes controller-manager
# defaults from config and apiserver should be adequate
# Add your own!
KUBE_CONTROLLER_MANAGER_ARGS="--node-monitor-grace-period=10s --pod-eviction-timeout=10s"
```
- 启动 master,并查看状态
```bash
for service in etcd kube-apiserver kube-controller-manager kube-scheduler; do
  systemctl enable $service
  systemctl restart $service
  systemctl status $service
done
```

### 3. 配置node节点
- 安装
```bash
yum install -y docker kubernetes-node flannel
```
- `cat /etc/kubernetes/config`
```
###
# kubernetes system config
#
# The following values are used to configure various aspects of all
# kubernetes services, including
#
#   kube-apiserver.service
#   kube-controller-manager.service
#   kube-scheduler.service
#   kubelet.service
#   kube-proxy.service
# logging to stderr means we get it in the systemd journal
KUBE_LOGTOSTDERR="--logtostderr=true"
# journal message level, 0 is debug
KUBE_LOG_LEVEL="--v=0"
# Should this cluster be allowed to run privileged docker containers
KUBE_ALLOW_PRIV="--allow-privileged=false"
# How the controller-manager, scheduler, and proxy find the apiserver
KUBE_MASTER="--master=http://192.168.118.120:8080"
```
- `cat /etc/kubernetes/kubelet`
```
###
# kubernetes kubelet (minion) config
# The address for the info server to serve on (set to 0.0.0.0 or "" for all interfaces)
KUBELET_ADDRESS="--address=0.0.0.0"
# The port for the info server to serve on
# KUBELET_PORT="--port=10250"
# You may leave this blank to use the actual hostname
KUBELET_HOSTNAME="--hostname-override=192.168.118.121"
# location of the api-server
KUBELET_API_SERVER="--api-servers=http://192.168.118.120:8080"
# pod infrastructure container
KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=registry.access.redhat.com/rhel7/pod-infrastructure:latest"
# Add your own!
KUBELET_ARGS="
```
- `cat /etc/sysconfig/flanneld`
```
# Flanneld configuration options
# etcd url location.  Point this to the server where etcd runs
FLANNEL_ETCD="http://192.168.118.120:2379"
# etcd config key.  This is the configuration key that flannel queries
# For address range assignment
#FLANNEL_ETCD_PREFIX="/atomic.io/network"
FLANNEL_ETCD_KEY="/coreos.com/network"
# Any additional options that you want to pass
#FLANNEL_OPTIONS="-iface=eth1"#这里特别注意，因为用的是虚拟机，网卡需要选择eth1
```
- 启动
```bash
for SERVICES in kube-proxy kubelet docker flanneld; do
  systemctl restart $SERVICES
  systemctl enable $SERVICES
  systemctl status $SERVICES
done
```

### 3. 验证
- master几点查看flannel网络是否分配:`etcdctl ls /coreos.com/network/subnets`
```
/coreos.com/network/subnets/10.1.95.0-24
/coreos.com/network/subnets/10.1.96.0-24
```
- 在 master上查看node:`kubectl get nodes`
```
NAME              STATUS    AGE
192.168.118.121   Ready     13m
192.168.118.122   Ready     14m
```
