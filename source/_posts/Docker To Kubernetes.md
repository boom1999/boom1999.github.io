---
title: Docker To Kubernetes after V1.24
date: 2024-05-10
tags: 
    - Docker
    - Kubernetes
categories: Summary
copyright: true
---
:pushpin: 多机搭建Docker和Kebernetes集群环境，以及部署应用程序。

- Kubernetes 是一个开源的容器编排引擎，用来对容器化应用进行自动化部署、扩缩和管理。仅仅用Docker是不够的，增加Docker-Compose可以实现多容器的编排，但是Kubernetes可以实现多机的容器编排，实现更高级的容器编排功能。
:dash::clock10::sleeping:

<!--more-->

## 1. before

1. 在此之前，Master主机上已用`Docker 24.0.2`，但是发现Kubernetes团队在2020年12约初的`Kubernetes 1.20`的[CHANGELOG](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.20.md)中指出"Docker as an underlying runtime is being deprecated. Docker-produced images will continue to work in your cluster with all runtimes, as they always have."，逐渐开始放弃对Docker的支持。<span style="color:grey">所以在此之后的版本中，Kubernetes不再支持Docker作为底层运行时，可能是因为Docker长期以来不支持CRI(Container Runtime Interface)，以至于需要长期维护Dockershim组件来进行适配；当然也可能是由于某些团队之间的利益冲突。</span>
2. 早期k8s通过硬编码的方式支持Docker，后来通过CRI来支持多种容器，但是Docker并没有支持CRI，导致k8s需要维护dockershim。2022年05月的`Kubernetes 1.24`正式将dockershim移除，不再支持Docker作为底层运行时，所以需要优先选择containerd或者CRI-O作为底层运行时。
3. 当然仍然可以继续使用Docker，目前存在支持CRI的shim cri-dockerd，但是这个项目并不由Kubernetes官方维护。
4. `Docker Dasmon`是Docker的守护进程，用于管理Docker容器；`dockerd`是Docker的服务端，用于接收Docker客户端的请求，Docker客户端是Docker的命令行工具，用于与Docker服务端进行交互；`containerd`是Docker的容器运行时，用于管理容器的生命周期,`containerd-shim`是代理，用于与`OCI runtime(runc)`进行交互，`runc`是容器执行器，用于创建和运行容器。
5. 为了避免不必要的警告和报错（*这是件会让人烦心的事情*），所以把移除dockershim前的最后一个大版本1.23.0作为本文使用环境，当然也是用了cri-dockerd作为底层运行时，使用后续版本的k8s也是可行的。

## 2. Versions

- System: Linux
  - Operating System: Ubuntu 18.04.6 LTS
  - Kernel: Linux 5.4.0-84-generic
  - Architecture: x86-64
- Docker: 24.0.2
- containerd: 1.6.21
- *Docker-Compose: 2.2.2*
- cri-dockerd: 0.3.14
- kubeadm: 1.28.10
- kubelet: 1.28.10
- kubectl: 1.28.10
- *Kubespary: maybe later*
- Kubernetes: 1.28.10

## 3. Install

### 3.1 Docker

- Remove old versions

    ```bash
    sudo apt-get remove docker docker-engine docker.io containerd runc  
    ```

- Install

    ```bash
    sudo apt-get update
    sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
    curl -fsSL https://get.docker.com | bash -s docker --version 24.0.2
    ```

- Check

    ```bash
    docker --version
    ```

  - Maybe the following error occurs: `permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/version": dial unix /var/run/docker.sock: connect: permission denied`
  - To avoid this error, add the current user to the docker group (or use the root user):

    ```bash
    sudo gpasswd -a $USER docker
    newgrp docker
    ```

### 3.2 kubeadm、kubelet、kubectl

> `kubeadm`用于创建k8s集群的工具，可以快速创建一个k8s集群，但是不适用于生产环境，可考虑使用`kubespray`或`kops`，更多的安全性、高可用性和监控以及日志，或者直接使用云服务商提供的k8s集群。
> `kubelet`在集群的每一个节点用来启动Pod和容器的组件，kubelet会根据PodSpec中的描述创建和管理容器。
> `kubectl`用于与k8s集群进行交互，可以通过kubectl来部署应用、查看集群资源、查看日志等。

- Update

  ```bash
  sudo apt-get update
  sudo apt-get install -y apt-transport-https ca-certificates curl gpg
  ```

- Add public key

  ```bash
  curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

  # Aliyun
  curl -fsSL https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
  ```

- Add repository

  ```bash
  echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

  # Aliyun
  echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
  ```

- Install and lock version

  ```bash
  sudo apt-get update
  sudo apt-get install -y kubelet kubeadm kubectl
  sudo apt-mark hold kubelet kubeadm kubectl
  ```

- Check

  ```bash
  kubeadm version
  kubelet --version
  kubectl version --client
  ```

- Maybe the following error occurs, then you need to use proxy or VPN to access the Internet or other images such as Aliyun, Tencent Cloud, etc.
  
  ```bash
  E: Unable to locate package kubelet
  E: Unable to locate package kubeadm
  E: Unable to locate package kubect
  ```

### 3.3 CRI-O or containerd or cri-dockerd

> 三选一即可，如果用的是Docker且k8s在1.24之前的版本，可以选择用dockershim替代cri-dockerd。

- cri-dockerd
  - Download the latest release from the [GitHub release page](https://github.com/Mirantis/cri-dockerd/releases)
  - Install

    ```bash
    sudo apt install -y ./cri-dockerd_0.3.14.3-0.ubuntu-focal_amd64.deb 
    ```

  - Check version and status

    ```bash
    cri-dockerd --version
    systemctl status cri-docker
    ```
  
  - Config containerd, and comment out the 'disabled_plugins = ["cri"]' line in the `/etc/containerd/config.toml` file.

    ```bash
    sudo vi /etc/containerd/config.toml
    sudo systemctl restart containerd
    ```

  - Config the cri-dockerd service
  Also you can download the service file from the [GitHub release page](https://github.com/Mirantis/cri-dockerd/blob/master/packaging/systemd/cri-docker.service)

    ```bash
    sudo wget -O /etc/systemd/system/cri-docker.service https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.service
    sudo wget -O /etc/systemd/system/cri-docker.socket https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.socket
    ```

  - Change the `ExecStart` in the `/etc/systemd/system/cri-docker.service` file， add `pause` image from Aliyun mirror.

    ```bash
    ExecStart=/usr/bin/cri-dockerd --container-runtime-endpoint fd:// --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.9
    ```

  - Restart the cri-dockerd service

    ```bash
    sudo systemctl daemon-reload      # 重新加载systemd管理的服务单元文件的命令
    sudo systemctl enable cri-docker
    sudo systemctl start cri-docker
    ```

### 3.4 Important settings

- Change cgroup driver to systemd
`kubeadm`把`kubelet`视为一个系统服务来管理，所以用`kubeadm启动时，推荐使用systemd作为cgroup驱动程序。

  - cgroup for Docker

    ```bash
    sudo vim /etc/docker/daemon.json
    ```

    ```json
    {
      "registry-mirrors": ["https://********.mirror.aliyuncs.com"],
      "exec-opts": [ "native.cgroupdriver=systemd" ]
    }
    ```

    > You can see your Aliyun mirror address in the [Aliyun](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors) after logging in.

  - Restart Docker

    ```bash
    sudo systemctl daemon-reload
    sudo systemctl restart docker
    ```

  - Check cgroup driver in Docker

    ```bash
    docker info | grep -i cgroup
    ```

  - cgroup for kubelet
  在版本 1.22 及更高版本中，如果用户没有在KubeletConfiguration中设置cgroupDriver字段，kubeadm会将它设置为默认值systemd。

- Date and time synchronization, use `Chrony`
  - Install

    ```bash
    sudo apt install chrony
    ```
  
  - Start synchronization and check status

    ```bash
    sudo systemctl start chrony
    systemctl status chrony
    ```

- Set hostname
  
  ```bash
  sudo --static hostnamectl set-hostname master # in master node
  sudo --static hostnamectl set-hostname node1  # in node1
  sudo --static hostnamectl set-hostname node2  # in node2
  .
  .
  .
  ```

- Set host in master node and check ping

  ```bash
  sudo cat >> /etc/hosts <<EOF
  192.168.254.129 master
  192.168.254.130 node1
  192.168.254.131 node2
  EOF
  ```

- Disable selinux, edit `/etc/selinux/config` and set `SELINUX=disabled` and reboot the system.
<span style="color:grey">允许容器访问宿主机的文件系统：Setting SELinux in permissive mode by runningsetenforce 0andsed ...effectively disables it. This is required to allow containers to access the host filesystem, which is needed by pod networks for example. You have to do this until SELinux support is improved in the kubelet.</span>

  - In Ubuntu, there is no selinux, so the following error may occur:

    ```bash
    cat: /etc/selinx/config: No such file or directory
    ```

- disable unix firewall
<span style="color:grey">避免重复的防火墙规则：Theiptablestooling can act as a compatibility layer, behaving like iptables but actually configuring nftables. This nftables backend is not compatible with the current kubeadm packages: it causes duplicated firewall rules and breakskube-proxy.</span>

  - 关闭防火墙，默认应该是关的，`Status: inactive`

    ```bash
    sudo ufw disable
    sudo ufw status
    ```

  - 也可以直接关闭防火墙服务，默认是开的，但防火墙未启动所以无效

    ```bash
    sudo systemctl stop firewalld
    sudo systemctl disable firewalld
    ```

- Disable swap
<span style="color:grey">开启Swap将导致和k8s的初衷有所违背，产生性能下降，详见[Issue](https://github.com/kubernetes/kubernetes/issues/53533)</span>
Reference: [How do i disable swap](https://askubuntu.com/questions/214805/how-do-i-disable-swap), reboot the system after disabling swap.

  ```bash
  # Temporary
  sudo swapoff -a
  # Permanent
  sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
  # Check
  sudo swapon --show
  free -h
  ```

### 3.5 Other settings

- Check MAC and product_uuid, make sure they are unique, otherwise the cluster will not work properly.
  - MAC
  
  ```bash
    sudo apt install net-tools
    ifconfig
    ifconfig "your eth name" | grep -i ether  
  ```
  
  - product_uuid

  ```bash
    sudo cat /sys/class/dmi/id/product_uuid
  ```

## 4. 组件的版本偏差策略

> 若集群的`kube-apiserver`有多个版本，规则都将对每个版本取并集。

### kubelet版本

- `kubelet`版本不能比`kube-apiserver`版本新。
- `kubelet`可以比`kube-apiserver`低三个次要版本（如果`kubelet` < 1.25，则只能比`kube-apiserver`低两个次要版本）。
- 例如
  - `kube-apiserver`处于1.30版本
  - `kubelet`支持1.30、1.29、1.28和1.27版本

### kube-proxy版本

- 其他同`kubelet`一样，`kube-proxy`的版本不能比`kube-apiserver`版本新，小于等于三个次要版本。
- `kube-proxy`可以与`kubelet`有新或旧三个次要版本，`1.25`之前则是两个次要版本。

### kube-controller-manager、kube-scheduler、cloud-controller-manager版本

- 不能比`kube-apiserver`版本新，最多比`kube-apiserver`低一个次要版本。
- 允许实时升级

### kubectl版本

- 与`kube-apiserver`版本相同或者低、高于`kube-apiserver`一个次要版本。

## 5. Start k8s cluster

- Initialize the master node

  ```bash
  sudo kubeadm init \
    --apiserver-advertise-address=192.168.254.129 \
    --image-repository registry.aliyuncs.com/google_containers \
    --kubernetes-version v1.28.0 \
    --service-cidr=10.96.0.0/12 \
    --pod-network-cidr=10.244.0.0/16 \
    --ignore-preflight-errors=all
  
  # This command will occur the following error (cri-dockerd and containerd are compatible):

  Found multiple CRI endpoints on the host. Please define which one do you wish to use by setting the 'criSocket' field in the kubeadm configuration file: unix:///var/run/containerd/containerd.sock, unix:///var/run/cri-dockerd.sock
  To see the stack trace of this error execute with --v=5 or higher
  ```

  ```bash
  sudo kubeadm init \
    --apiserver-advertise-address=192.168.254.129 \
    --image-repository registry.aliyuncs.com/google_containers \
    --kubernetes-version v1.28.0 \
    --service-cidr=10.96.0.0/12 \
    --pod-network-cidr=10.244.0.0/16 \
    --ignore-preflight-errors=all \
    --cri-socket=unix:///var/run/cri-dockerd.sock
  
  # Reply success
  Your Kubernetes control-plane has initialized successfully!

  To start using your cluster, you need to run the following as a regular user:

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

  Alternatively, if you are the root user, you can run:

    export KUBECONFIG=/etc/kubernetes/admin.conf

  You should now deploy a pod network to the cluster.
  Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
    https://kubernetes.io/docs/concepts/cluster-administration/addons/

  Then you can join any number of worker nodes by running the following on each as root:

  kubeadm join 192.168.254.129:6443 --token kgsf73.msjq37v3zwqkaycg \
          --discovery-token-ca-cert-hash sha256:feb06b7d17a02964c162b3f5dda5e5182f8e407383ebf1d974b26416937d65ec
  ```

  - Copy the configuration file to the user directory

    ```bash
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```
  
  - Check the status of the master node and the pods, *we haven't installed the network plugin yet, so the master is in the `NotReady` state.*

    ```bash
    kubectl get nodes

    NAME     STATUS     ROLES           AGE    VERSION
    master   NotReady   control-plane   8m9s   v1.28.10

    kubectl get pod -A
    
    NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE
    kube-system   coredns-66f779496c-6b4rw         0/1     Pending   0          6m13s
    kube-system   coredns-66f779496c-xszmj         0/1     Pending   0          6m13s
    kube-system   etcd-master                      1/1     Running   0          6m27s
    kube-system   kube-apiserver-master            1/1     Running   0          6m26s
    kube-system   kube-controller-manager-master   1/1     Running   0          6m29s
    kube-system   kube-proxy-r87hl                 1/1     Running   0          6m13s
    kube-system   kube-scheduler-master            1/1     Running   0          6m28s
    ```

- Join the worker node
  创建的指令和token在`kubeadm init`输出的最后一行，也需要`--cri-socket /var/run/cri-dockerd.sock`，因为我们的节点都配置了`cri-dockerd`以及`containerd`，存在冲突，需要指定使用runc。token的有效期为24小时，过期后需要`kubeadm token create --print-join-command`重新生成。

  ```bash
  # The command is generated by the master node, the last line of the output of the `kubeadm init` command.
  # Also need --cri-socket /var/run/cri-dockerd.sock'
  sudo kubeadm join 192.168.254.129:6443 --cri-socket unix:///var/run/cri-dockerd.sock --token kgsf73.msjq37v3zwqkaycg \
          --discovery-token-ca-cert-hash sha256:feb06b7d17a02964c162b3f5dda5e5182f8e407383ebf1d974b26416937d65ec
  ```

  - Check in the master node

    ```bash
    kubectl get nodes -A

    NAMESPACE     NAME                             READY   STATUS              RESTARTS   AGE
    kube-system   coredns-66f779496c-6b4rw         0/1     Pending             0          23m
    kube-system   coredns-66f779496c-xszmj         0/1     Pending             0          23m
    kube-system   etcd-master                      1/1     Running             0          24m
    kube-system   kube-apiserver-master            1/1     Running             0          24m
    kube-system   kube-controller-manager-master   1/1     Running             0          24m
    kube-system   kube-proxy-5jgmq                 0/1     ContainerCreating   0          106s
    kube-system   kube-proxy-p9tq5                 0/1     ContainerCreating   0          6m8s
    kube-system   kube-proxy-r87hl                 1/1     Running             0          23m
    kube-system   kube-scheduler-master            1/1     Running             0          24m
    ```

  - Copy the configuration file to the user directory, 否则将导致`kubectl get nodes`时出现`The connection to the server localhost:8080 was refused - did you specify the right host or port?`，以及该节点将始终处于`NotReady`状态，该节点的kube-proxy处于`ContainerCreating`状态。

    ```bash
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/kubelet.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```

- Remove the node from the cluster

  - **In master node**

    ```bash
    # Check the node 和 pods, 先检查一下
    kubectl get nodes
    kubectl get pods -A -o wide
    # 阻止调度新的pod在该节点上，但此时pod可以在node上继续运行
    kubectl cordon node1
    # 驱逐node1上的所有pod
    kubectl drain node1 --ignore-daemonsets --delete-local-data
    # 删除node1
    kubectl delete node node1
    ```
  
  - **In worker node**

    ```bash
    # Leave the cluster
    sudo kubeadm reset --cri-socket /var/run/cri-dockerd.sock
    ```

    ```bash
    # 删除残留的文件
    sudo rm -rf /etc/kubernetes/
    sudo rm -rf $HOME/.kube


    # 清除iptables或者ipvs的配置
    sudo iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
    sudo ipvsadm --clear
    ```

- Add CNI plugin
  到现在为止不同的宿主机之间的pod无法跨主机通信，需要安装网络插件，这里选择`Calico`，也可以选择`Flannel`、`Cilium`、`Weave Net`等。

  - Download the `Calico` plugin

    ```bash
    # wget https://docs.projectcalico.org/manifests/calico.yaml
    curl -O https://docs.tigera.io/archive/v3.25/manifests/calico.yaml
    ```

  - Modify the `calico.yaml` file, change the `CALICO_IPV4POOL_CIDR` to `pod-network-cidr's` value, and apply the `Calico` plugin.

    ```bash
    - name: CALICO_IPV4POOL_CIDR
        value: "10.244.0.0/16"
    ```

    ```bash
    kubectl apply -f calico.yaml
    ```

  - But there is still some bugs those are fixed in [Debug Section](#jump).

- Complete the master ande node1, node2 *(need to wait for a while)*

  ```bash
  boom@master:~$ kubectl get pods -A -o wide
  NAMESPACE     NAME                                       READY   STATUS    RESTARTS      AGE     IP                NODE     NOMINATED NODE   READINESS GATES
  kube-system   calico-kube-controllers-6d668dcdd6-gbrfq   1/1     Running   0             57m     10.244.104.3      node2    <none>           <none>
  kube-system   calico-node-f8jwx                          1/1     Running   0             5m21s   192.168.254.130   node1    <none>           <none>
  kube-system   calico-node-gvs6h                          1/1     Running   0             57m     192.168.254.131   node2    <none>           <none>
  kube-system   calico-node-qk946                          1/1     Running   1 (45m ago)   57m     192.168.254.129   master   <none>           <none>
  kube-system   coredns-66f779496c-6b4rw                   1/1     Running   2 (45m ago)   7h53m   10.244.219.66     master   <none>           <none>
  kube-system   coredns-66f779496c-xszmj                   1/1     Running   2 (45m ago)   7h53m   10.244.219.65     master   <none>           <none>
  kube-system   etcd-master                                1/1     Running   2 (45m ago)   7h54m   192.168.254.129   master   <none>           <none>
  kube-system   kube-apiserver-master                      1/1     Running   2 (45m ago)   7h54m   192.168.254.129   master   <none>           <none>
  kube-system   kube-controller-manager-master             1/1     Running   2 (45m ago)   7h54m   192.168.254.129   master   <none>           <none>
  kube-system   kube-proxy-ff75p                           1/1     Running   0             86m     192.168.254.131   node2    <none>           <none>
  kube-system   kube-proxy-r87hl                           1/1     Running   2 (45m ago)   7h53m   192.168.254.129   master   <none>           <none>
  kube-system   kube-proxy-zgnjd                           1/1     Running   0             5m21s   192.168.254.130   node1    <none>           <none>
  kube-system   kube-scheduler-master                      1/1     Running   2 (45m ago)   7h54m   192.168.254.129   master   <none>           <none>

  boom@master:~$ kubectl get nodes
  NAME     STATUS   ROLES           AGE     VERSION    INTERNAL-IP       EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
  master   Ready    control-plane   7h58m   v1.28.10   192.168.254.129   <none>        Ubuntu 18.04.6 LTS   5.4.0-150-generic   docker://24.0.2
  node1    Ready    <none>          10m     v1.28.10   192.168.254.130   <none>        Ubuntu 18.04.6 LTS   5.4.0-150-generic   docker://24.0.2
  node2    Ready    <none>          90m     v1.28.10   192.168.254.131   <none>        Ubuntu 18.04.6 LTS   5.4.0-150-generic   docker://24.0.2
  ```

- Dashboard

  - Download the `Dashboard` plugin

    ```bash
    curl -O https://raw.githubusercontent.com/kubernetes/dashboard/v2.4.0/aio/deploy/recommended.yaml
    ```

  - Modify the `recommended.yaml` file, add the `type: NodePort` to the `Service` section, then apply the `Dashboard` plugin.

    ```yaml
    kind: Service
    apiVersion: v1
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
      name: kubernetes-dashboard
      namespace: kubernetes-dashboard
    spec:
      ports:
        - port: 443
          targetPort: 8443
      selector:
        k8s-app: kubernetes-dashboard
      type: NodePort
    ```

    ```bash
    kubectl apply -f recommended.yaml
    ```

  - Check the `Dashboard` plugin, but the `Pod` is in the `Pending` state.

    ```bash
    kubectl get pod -n kubernetes-dashboard
    
    NAMESPACE              NAME                                         READY   STATUS    RESTARTS      AGE     IP                NODE     NOMINATED NODE   READINESS GATES
    kubernetes-dashboard   dashboard-metrics-scraper-5657497c4c-rvfnh   1/1     Running   0             3m51s   10.244.166.130    node1    <none>           <none>
    kubernetes-dashboard   kubernetes-dashboard-78f87ddfc-52msp         1/1     Running   0             3m51s   10.244.166.129    node1    <none>           <none>
    ```

  - Get the port of the `Dashboard` plugin, get `32000` **(will change every time you apply the `Dashboard` plugin, so you need to check it every time).**

    ```bash
    kubectl get svc -n kubernetes-dashboard

    NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
    dashboard-metrics-scraper   ClusterIP   10.105.243.112   <none>        8000/TCP        2m42s
    kubernetes-dashboard        NodePort    10.99.228.207    <none>        443:32000/TCP   2m43s
    ```

  - Access the `Dashboard` plugin, use the `master` node's IP and the `NodePort` port.

  ![DashboardLogin][1]

  - Create a `Dashboard` user, and get the token. (Or you can use the `kubeconfig` file)

    ```bash
    # Create a service account
    kubectl create serviceaccount dashboard-admin -n kubernetes-dashboard
    # Create a cluster role binding
    kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:dashboard-admin
    # Get the token
    kubectl create token dashboard-admin -n kubernetes-dashboard
    ```

  ![DashboardWorkloads][2]

## 6.  <span id="jump">Debug</span>

- The pod `kube-proxy` is in `ContainerCreating` status for a long time。

  ```bash
  kubectl  describe pod kube-proxy-4pkw4 -n kube-system

  Events:
  Type     Reason                  Age                   From               Message
  ----     ------                  ----                  ----               -------
  Normal   Scheduled               12m                   default-scheduler  Successfully assigned kube-system/kube-proxy-4pkw4 to node2
  Warning  FailedCreatePodSandBox  4m52s (x2 over 6m1s)  kubelet            Failed to create pod sandbox: rpc error: code = Unknown desc = failed pulling image "registry.k8s.io/pause:3.9": Error response from daemon: Head "https://europe-west2-docker.pkg.dev/v2/k8s-artifacts-prod/images/pause/manifests/3.9": dial tcp 108.177.125.82:443: connect: connection refused
  Warning  FailedCreatePodSandBox  44s (x8 over 10m)     kubelet            Failed to create pod sandbox: rpc error: code = Unknown desc = failed pulling image "registry.k8s.io/pause:3.9": Error response from daemon: Head "https://us-west2-docker.pkg.dev/v2/k8s-artifacts-prod/images/pause/manifests/3.9": dial tcp 108.177.97.82:443: connect: connection refused
  Warning  FailedCreatePodSandBox  10s (x11 over 11m)    kubelet            Failed to create pod sandbox: rpc error: code = Unknown desc = failed pulling image "registry.k8s.io/pause:3.9": Error response from daemon: Head "https://asia-east1-docker.pkg.dev/v2/k8s-artifacts-prod/images/pause/manifests/3.9": dial tcp 74.125.23.82:443: connect: connection refused
    ```

  - 查一下应该是`pause`的镜像拉取失败。但是已经在上文，`/etc/systemd/system/cri-docker.service`中的`ExecStart`中添加了`pause`的镜像，有可能是没有重启`cri-dockerd`服务，导致镜像没有生效；或者是没有重新加载systemd管理的服务单元文件。*修复后加载成功了*

    ```bash
    sudo systemctl daemon-reload
    sudo systemctl restart cri-docker

    kubectl  describe pod kube-proxy-4pkw4 -n kube-system

    Events:
    Type    Reason     Age    From               Message
    ----    ------     ----   ----               -------
    Normal  Scheduled  5m24s  default-scheduler  Successfully assigned kube-system/kube-proxy-ff75p to node2
    Normal  Pulling    5m22s  kubelet            Pulling image "registry.aliyuncs.com/google_containers/kube-proxy:v1.28.0"
    Normal  Pulled     5m16s  kubelet            Successfully pulled image "registry.aliyuncs.com/google_containers/kube-proxy:v1.28.0" in 6.031s (6.032s including waiting)
    Normal  Created    5m16s  kubelet            Created container kube-proxy
    Normal  Started    5m16s  kubelet            Started container kube-proxy
    ```

- The pod `calico-node-fcdmn` is in `Init:ImagePullBackOff` status for a long time, 查一下应该是`calico`的镜像拉取失败

  ```bash
  kubectl  describe pod calico-node-fcdmn -n kube-system

  Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  57m                  default-scheduler  Successfully assigned kube-system/calico-node-fcdmn to node1
  Normal   Pulling    50m (x4 over 56m)    kubelet            Pulling image "docker.io/calico/cni:v3.25.0"
  Warning  Failed     49m (x4 over 55m)    kubelet            Error: ErrImagePull
  Warning  Failed     48m (x7 over 55m)    kubelet            Error: ImagePullBackOff
  Warning  Failed     21m (x9 over 55m)    kubelet            Failed to pull image "docker.io/calico/cni:v3.25.0": rpc error: code = Canceled desc = context canceled
  Normal   BackOff    11m (x127 over 55m)  kubelet            Back-off pulling image "docker.io/calico/cni:v3.25.0"
    ```

  - 查一下还是镜像源的问题，虽然给docker配置了阿里云的镜像源，但是`calico`的镜像还是从`docker.io`拉取，所以怀疑是Calico清单文件出了问题，看一下果然是，网上一堆居然都不解决这个问题能成功加载的，看出来好多都是东拼西凑的Tutorial。
  
    ```bash
    cat calico.yaml |grep 'image:'

    image: docker.io/calico/cni:v3.25.0
    image: docker.io/calico/cni:v3.25.0
    image: docker.io/calico/node:v3.25.0
    image: docker.io/calico/node:v3.25.0
    image: docker.io/calico/kube-controllers:v3.25.0

    # 替换掉所有'docker.io'的前缀为空
    sed -i 's#docker.io/##g' calico.yaml
    # 重新检查
    cat calico.yaml |grep 'image:'
    image: calico/cni:v3.25.0
    image: calico/cni:v3.25.0
    image: calico/node:v3.25.0
    image: calico/node:v3.25.0
    image: calico/kube-controllers:v3.25.0

    # 重启calico
    kubectl delete -f calico.yaml
    kubectl apply -f calico.yaml
    ```

## 7. Add service

- Nginx *(for example, nginx-deployment.yaml)*

  ```yaml
  apiVersion: apps/v1  
  kind: Deployment  
  metadata:  
    name: nginx-deployment-1  
  spec:  
    replicas: 3  
    selector:  
      matchLabels:  
        app: nginx-1  
    template:  
      metadata:  
        labels:  
          app: nginx-1  
      spec:  
        containers:  
        - name: nginx  
          image: nginx:latest  
          ports:  
          - containerPort: 80  
  ---  
  apiVersion: apps/v1  
  kind: Deployment  
  metadata:  
    name: nginx-deployment-2  
  spec:  
    replicas: 3  
    selector:  
      matchLabels:  
        app: nginx-2  
    template:  
      metadata:  
        labels:  
          app: nginx-2  
      spec:  
        containers:  
        - name: nginx  
          image: nginx:latest  
          ports:  
          - containerPort: 80  
  ```

- Apply the `Nginx` service

  ```bash
  kubectl apply -f nginx-deployment.yaml
  ```

## 8. Stop and restart the cluster

- Stop the cluster

  ```bash
  systemctl stop kubelet 
  systemctl stop etcd 
  systemctl stop docker
  ```

- Restart the cluster

  ```bash
  systemctl start docker
  systemctl start etcd
  systemctl start kubelet
  systemctl status docker etcd kubelet
  ```

<!-- markdownlint-disable-file MD025 MD028 MD033 -->
[1]: https://www.lingzhicheng.cn/usr/file/picture/k8s/k8sDashboardLogin.jpg
[2]: https://www.lingzhicheng.cn/usr/file/picture/k8s/k8sWorkloads.jpg
