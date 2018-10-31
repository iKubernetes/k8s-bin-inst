# k8s-bin-inst
kubernetes installation hard way.

## etcd集群
### 安装
~]# yum install epel-release
~]# yum install etcd

### 配置参数
编辑配置文件/etc/etcd/etcd.conf，设置以下参数为合适的值

* ETCD_DATA_DIR="/var/lib/etcd/k8s.etcd"
* ETCD_LISTEN_PEER_URLS：集群节点间通信监听的URL，例如"https://172.20.0.51:2380"；
* ETCD_LISTEN_CLIENT_URLS：向客户端提供服务所监听的URL，例如"https://172.20.0.51:2379"；
* ETCD_NAME：当前节点的名称，例如"k8s-etcd01.magedu.com"；

* ETCD_INITIAL_ADVERTISE_PEER_URLS：此成员节点向etcd集群通告的URL，例如"https://k8s-etcd01.magedu.com:2380"；
* ETCD_ADVERTISE_CLIENT_URLS：此节点通告的用于向客户端提供服务的URL，例如"https://k8s-etcd01.magedu.com:2379"；
* ETCD_INITIAL_CLUSTER="k8s-etcd01.magedu.com：用于引导并启动集群的配置信息，由集群的全部成员节点向集群通告的URL组成的列表，例如https://k8s-etcd01.magedu.com:2380,k8s-etcd02.magedu.com=https://k8s-etcd02.magedu.com:2380,k8s-etcd03.magedu.com=https://k8s-etcd03.magedu.com:2380"；

* ETCD_CERT_FILE：服务器证书，例如"/etc/etcd/pki/server.crt"
* ETCD_KEY_FILE：服务器私钥，例如"/etc/etcd/pki/server.key"
* ETCD_CLIENT_CERT_AUTH：是否需要认证客户端，安全通信的场景需要启用，其值为"true"
* ETCD_TRUSTED_CA_FILE：信任的CA的数字证书，"/etc/etcd/pki/ca.crt"
* ETCD_PEER_CERT_FILE：集群成员通信时用于认证的证书，例如"/etc/etcd/pki/peer.crt"
* ETCD_PEER_KEY_FILE：成员通信时所用证书配对的私钥，例如"/etc/etcd/pki/peer.key"
* ETCD_PEER_CLIENT_CERT_AUTH：是否验正集群成员节点的客户端证书，安全通信的场景需要启用，其值为"true"
* ETCD_PEER_TRUSTED_CA_FILE：用于为成员安全通信签署证书的CA的证书，"/etc/etcd/pki/ca.crt"


## Master

### 配置并启动服务

1. 创建运行者用户及工作目录
~]# useradd -r kube

~]# mkdir /var/run/kubernetes
~]# chown kube.kube /var/run/kubernetes


~]# systemctl start kube-apiserver kube-controller-manager kube-scheduler


### 配置kubectl

~]# mkdir $HOME/.kube
~]# cp /etc/kubernetes/auth/admin.conf $HOME/.kube/config

### 创建ClusterRoleBinding，授予用户相应操作所需要的权限：

~]# kubectl create clusterrolebinding system:bootstrapper --group=system:bootstrappers --clusterrole=system:bootstrapper
或者：
~]# kubectl create clusterrolebinding system:bootstrapper --user=system:bootstrapper --clusterrole=system:node-bootstrapper

## Node

1.提供CNI插件
下载位置：https://github.com/containernetworking/plugins/releases
    
~]# mkdir -p /opt/cni/bin    
~]# tar xf cni-plugins-amd64-v0.7.1.tgz -C /opt/cni/bin/

2.启用ipvs内核模块

创建内核模块载入相关的脚本文件/etc/sysconfig/modules/ipvs.modules，设定自动载入的内核模块。文件内容如下：

    #!/bin/bash
    ipvs_mods_dir="/usr/lib/modules/$(uname -r)/kernel/net/netfilter/ipvs"
    for i in $(ls $ipvs_mods_dir | grep -o "^[^.]*"); do
        /sbin/modinfo -F filename $i  &> /dev/null
        if [ $? -eq 0 ]; then
            /sbin/modprobe $i
        fi
    done


