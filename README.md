# 离线安装ICP社区精简版

注意：该文档旨在让用户能够快速的将 ICP 环境部署起来，其中的安装包是基于 ICP 公开的文档和社区版 Docker 镜像制作的，并非 ICP 官方制作。

下载地址：https://pan.baidu.com/s/1ghPPPgv380xc0H4qq5VvBg

两个离线包，其中一个为镜像包（3.8G, `icp-3.1.0-ce-image.tgz`），另一个为配置包（4K，`icp-3.1.0-ce-config.tgz`）。

需要至少一台机器，CPU在4核及以上，内存6G及以上，磁盘空间30G及以上。

安装完成之后可以根据需求来添加和启用ICP的其他组件。

该离线包包含 Kubernetes/Etcd 和以下 24 种服务：

* auth-apikeys
* auth-idp
* auth-pap
* auth-pdp
* calico
* catalog-ui
* cert-manager
* heapster
* helm-api
* helm-repo
* icp-management-ingress
* image-manager
* image-registry
* kube-dns(coredns)
* mariadb
* mgmt-repo
* mongodb
* nginx-ingress
* platform-api
* platform-ui
* secret-watcher
* security-onboarding
* tiller
* unified-router


## 操作系统

* RHEL/CentOS: 7.3+
* Ubuntu: 16.04/18.04


## 关闭防火墙

通常是 RHEL/CentOS 防火墙会默认关闭除22以外的端口，ICP安装过程中发现这种情况就会终止安装。为了确保一次就能安装成功，所以建议先关闭防火墙，命令如下：

```
systemctl stop firewalld
systemctl disable firewalld
```


## 安装 Docker

* CentOS: https://docs.docker.com/install/linux/docker-ce/centos/
* Ubuntu: https://docs.docker.com/install/linux/docker-ce/ubuntu/

## 配置 Docker log rotation

当磁盘空间比较小时，服务运行一段时间可能就会产生很多日志，会占用相当一部分空间。这时就需要配置 log rotation 了。

建议所有机器都配置一下，详细说明：https://success.docker.com/article/how-to-setup-log-rotation-post-installation

简单步骤：

1. 创建 /etc/docker/daemon.json 文件

2. 写入以下内容

    ```
    {
      "log-driver": "json-file",
      "log-opts": {
        "max-size": "80m",
        "max-file": "3"
      }
    }
    ```
3. 重启 docker

    ```
    systemctl restart docker
    ```

## 在机器上安装依赖包

* CentOS: yum install socat
* Ubuntu: apt install socat python

## 检查机器上的文件

1. /etc/resolv.conf DNS 文件里需要配置正确的DNS服务器

    如果该文件为空并且不知道怎样填写(nameserver不能是127.0.x.x)，可以写入以下内容：

    ```
    nameserver 223.5.5.5
    ```

2. /etc/hosts 文件里需要有以下两行内容

    ```
    127.0.0.1 localhost
    your-machine-ip your-machine-hostname
    ```

3. 保证有默认路由

    ```
    ip route show | grep default
    ```

## 下载安装包到相应机器并导入到docker里


    docker load -i icp-3.1.0-ce-image.tgz


## 下载配置文件并解压


    cd /root
    tar xf icp-3.1.0-ce-config.tgz


## 配置IP地址和机器root用户密码


    vim /root/icp-3.1.0-ce/cluster/hosts


## 配置容器网络，保证不与机器网络冲突


    vim /root/icp-3.1.0-ce/cluster/config.yaml

    network_cidr: 10.1.0.0/16
    service_cluster_ip_range: 10.0.0.0/16

## 开始安装

切换到 cluster 目录，并运行安装命令：

```
cd /root/icp-3.1.0-ce/cluster/
docker run -t --rm --net=host -e LICENSE=accept -v $(pwd):/installer/cluster ibmcom/icp-inception:3.1.0 install -v
```

## 添加并启用ICP的其他组件

注意：开启更多ICP组件也意味着需要更多的CPU和内存、磁盘空间，建议提前做好规划。

### 1.在配置文件中启用某个服务

将以下文件中对应服务的状态从`disabled`改成`enabled`。

```
vim /root/icp-3.1.0-ce/cluster/config.yaml
```

### 2.运行启用命令

```
cd /root/icp-3.1.0-ce/cluster/
docker run -t --rm --net=host -e LICENSE=accept -v $(pwd):/installer/cluster ibmcom/icp-inception:3.1.0 addon -v
```

## 开始使用 ICP

在浏览器里打开 ICP UI 先体验一下，然后可以参考以下文档做一些简单的操作。

https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/manage_applications/manage_workloads.html

## ICP 3.1.0 官方文档

https://www.ibm.com/support/knowledgecenter/SSBS6K_3.1.0/kc_welcome_containers.html
