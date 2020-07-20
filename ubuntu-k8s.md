## 下文是Ubuntu下kubernetes安装过程，使用kubeadm，过程详细，供新手菜鸟参考。

## Docker与Kuberentes的安装与部署
      # 环境：Ubuntu 16.04(实验环境)       (centos版本请直接查看README.md)
      # 集群网络： 192.168.0.50    k8s-master
      #           192.168.0.51    k8s-node1
      #           192.168.0.52    k8s-node2
### 1. 安装前的准备： 
    1. 最好先固定ip（既静态Ip），方法不展示(自行百度 or Google)
    2. 禁用selinux和防火墙
        #禁用selinux
        sudo vim /etc/selinux/config
        SELINUX=disabled
        #禁用防火墙
        setenforce 0
        sudo ufw disable
        #验证
        getenforce
        Disabled
    3. 关闭交换空间  
       $ swapoff -a  
       $ vim /etc/fstab       #注释掉swap那一整行  
    4. 修改hostname  
       $ vim /etc/hostname    #修改为k8s节点名称  
       $ hostname $(cat /etc/hostname)    #立即生效  
### 2. 安装Docker: 
        # step 1: 安装必要的一些系统工具
        sudo apt-get update
        sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
        # step 2: 安装GPG证书
        curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
        # Step 3: 写入软件源信息
        sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
        # Step 4: 更新并安装 Docker-CE
        sudo apt-get -y update
        sudo apt-get -y install docker-ce
  
  #### 其他安装注意事项(不需要跳过)：
        # 安装指定版本的Docker-CE:
        # Step 1: 查找Docker-CE的版本:
        # apt-cache madison docker-ce
        #   docker-ce | 17.03.1~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
        #   docker-ce | 17.03.0~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
        # Step 2: 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.1~ce-0~ubuntu-xenial)
        # sudo apt-get -y install docker-ce=[VERSION]
        # 通过经典网络、VPC网络内网安装时，用以下命令替换Step 2、Step 3中的命令
        # 经典网络：
        # curl -fsSL http://mirrors.aliyuncs.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
        # sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyuncs.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
        # VPC网络：
        # curl -fsSL http://mirrors.cloud.aliyuncs.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
        # sudo add-apt-repository "deb [arch=amd64] http://mirrors.cloud.aliyuncs.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
### 3. 修改linux内核
    $vim /etc/sysctl.d/k8s.conf       # 在该文件中添加如下内容  
      net.bridge.bridge-nf-call-ip6tables = 1  
      net.bridge.bridge-nf-call-iptables = 1  
    $sysctl -p  
### 4. 修改Docker 镜像源(改为阿里镜像源)
    $sudo mkdir -p /etc/docker
    $sudo tee /etc/docker/daemon.json <<-'EOF'
    {
      "registry-mirrors": ["https://xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.mirror.aliyuncs.com"]       # xxxxxxxxxxxxxxxxxxxx部分可以自己注册一个阿里云账号，在容器镜像服务中查找镜像加速器页面。
    }
    EOF
    $sudo systemctl daemon-reload
    $sudo systemctl restart docker
### 5. 安装kubernetes
  ###### step1：添加kubernetes软件源
        curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -    # --socks5 IP:PORT  指定代理
        #添加Kubernetes软件源
        cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
        deb http://mirrors.ustc.edu.cn/kubernetes/apt kubernetes-xenial main
        EOF
  ###### step2：运行apt-get安装kubeadm，kubelet，kubectl（指定版本）worker节点可以不安装kubectl
      $sudo apt-get update && apt-get install kubelet=1.16.0-00 kubeadm=1.16.0-00 kubectl=1.16.0-00
      ##! 注意：node节点可以不需要安装kubectl
  ###### step2：启动kubelet服务并设置开机自动启动
      $systemctl enable kubelet  && systemctl start kubelet
  ###### step4：拉取镜像并tag为所需镜像
    1> 查看所需镜像
      $kubeadm config images list --kubernetes-version v1.16.0  
                    k8s.gcr.io/kube-apiserver:v1.16.0  
                    k8s.gcr.io/kube-controller-manager:v1.16.0  
                    k8s.gcr.io/kube-scheduler:v1.16.0  
                    k8s.gcr.io/kube-proxy:v1.16.0  
                    k8s.gcr.io/pause:3.1  
                    k8s.gcr.io/etcd:3.3.15-0  
                    k8s.gcr.io/coredns:1.6.2  
    2> 拉取列出的镜像
      $docker pull mirrorgooglecontainers/kube-proxy-amd64:v1.16.0   
      $docker pull mirrorgooglecontainers/kube-scheduler-amd64:v1.16.0  
      $docker pull mirrorgooglecontainers/kube-apiserver-amd64:v1.16.0  
      $docker pull mirrorgooglecontainers/kube-controller-manager-amd64:v1.16.0  
      $docker pull mirrorgooglecontainers/pause:3.1  
      $docker pull mirrorgooglecontainers/etcd:3.3.15-0  
      $docker pull coredns/coredns:1.6.2  
    3> docker tag将拉取的镜像改为所需镜像 
      $docker tag mirrorgooglecontainers/kube-proxy-amd64:v1.16.0 k8s.gcr.io/kube-proxy:v1.16.0  
      ……………… 
  ###### step5：初始化master节点
      $kubeadm init --kubernetes-version:v1.16.0  
           [init] Using Kubernetes version: v1.14.0  
           [preflight] Running pre-flight checks  
           [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please            follow the guide at https://kubernetes.io/docs/setup/cri/  
           [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 19.03.5. Latest validated                    version: 18.09  
           ………  
           Your Kubernetes control-plane has initialized successfully!  
           To start using your cluster, you need to run the following as a regular user：    
           
           mkdir -p $HOME/.kube  
           sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
           sudo chown $(id -u):$(id -g) $HOME/.kube/config  
              
           You should now deploy a pod network to the cluster.  
           Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:  
           https://kubernetes.io/docs/concepts/cluster-administration/addons/  
           Then you can join any number of worker nodes by running the following on each as root:  
           
           kubeadm join 192.168.0.50:6443 --token xpu8my.9ao7vyibwbx1cz3e \  
           --discovery-token-ca-cert-hash sha256:e4d501e46a9eb95f5eea6433748bbfd107bd9a0094e063f59eb57ab5533a0979  
      ！！！！！！！！！  
           此时需要记录下上面日志中的kubeadm join以及后面得跟随的内容，供添加node节点使用 即  
           kubeadm join 192.168.0.50:6443 --token xpu8my.9ao7vyibwbx1cz3e \  
           --discovery-token-ca-cert-hash sha256:e4d501e46a9eb95f5eea6433748bbfd107bd9a0094e063f59eb57ab5533a0979
    
  ###### step6：将kubectl命令添加到普通用户
      #使用普通用户执行如下命令：  
        $mkdir -p $HOME/.kube  
        $sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
        $sudo chown $(id -u):$(id -g) $HOME/.kube/config  
  ###### step7: 部署weave网络
        $kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"  
        #使用kubectl get命令查看集群是否均已正常运行  
        $kubectl get pods -n kube-system  
        NAME                                                 READY   STATUS    RESTARTS   AGE  
       coredns-fb8b8dccf-l4pjd                              1/1     Running       0       58m  
       coredns-fb8b8dccf-mckcj                              1/1     Running       0       58m  
       etcd-k8s-master                                      1/1     Running       0       57m  
       kube-apiserver-k8s-master                            1/1     Running       0       58m  
       kube-controller-manager-k8s-master                   1/1     Running       0       58m  
       kube-proxy-682g5                                     1/1     Running       0       58m  
       kube-scheduler-k8s-master                            1/1     Running       0       58m  
       weave-net-tv29g                                      2/2     Running       0       92s  
   ###### step8：添加node
       1> 在node节点上执行环境准备，安装docker，修改镜像源，安装kubeadm和kubelet，注意需要拉取kube-proxy、pause、coredns三个镜像，否则weave容器等无法在该节点上正常运行。
       2> 在node节点上执行我们保存的kubeadm join命令
       3> 在master节点上执行： kubectl get nodes 查看节点是否正常。正常状态为Ready。 如果不正常查看pods解决。
-----------------------------------------------------------------------------------------------------------------
## 至此kubernetes安装结束，其他组件请查看相应文件夹下内容。
    
