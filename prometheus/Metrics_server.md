# 安装过程
## 前期准备
    #下面我们针对 kubernetes 的使用进行演示，由于kubelet启用了 https，所以需要拥有一个认证帐户去访问它，这里我们创建一个ServiceAccount账号；
    # 创建一个监控专用的名称空间 monitor
    [root@master01 ~]# kubectl create ns monitor
    namespace/monitor created
    # 创建一个SA帐号
    [root@master01 ~]# kubectl create serviceaccount monitor -n monitor
    serviceaccount/monitor created
    # 查看创建SA后，生成的 secret 信息
    [root@master01 ~]# kubectl get secret -n monitor
    NAME TYPE DATA AGE
    default-token-kdrzm kubernetes.io/service-account-token 3      34s
    monitor-token-2ktr2 kubernetes.io/service-account-token 3      18s
    [root@master01 ~]#
    # SA：monitor 绑定最高集群角色
    [root@master01 ~]# kubectl create clusterrolebinding monitor-cluster -n monitor --clusterrole=cluster-admin --serviceaccount=monitor:monitor
    clusterrolebinding.rbac.authorization.k8s.io/monitor-cluster created
## 验证
### 根据创建 serviceAccount 帐号 monitor 的 token 去访问 kubelet 的10250端口验证
    [root@master01 ~]# kubectl describe secret monitor-token-2ktr2 -n monitor
    Name: monitor-token-2ktr2
    Namespace: monitor
    Labels:       <none>
    Annotations:  kubernetes.io/service-account.name: monitor
              kubernetes.io/service-account.uid: 718326e6-57ec-490c-9fcb-60698acca518

    Type: kubernetes.io/service-account-token

    Data
    ====
    ca.crt:     1025 bytes
    namespace: 7 bytes
    token:        eyJhbGciOiJSUzI1NiIsImtpZCI6IlZ2bGJjaEN2MjFwazRmLUNWdkxBYVoxUHBleTBCUFBzWW0xU25uMGM1Y3MifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJtb25pdG9yIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6Im1vbml0b3ItdG9rZW4tMmt0cjIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoibW9uaXRvciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjcxODMyNmU2LTU3ZWMtNDkwYy05ZmNiLTYwNjk4YWNjYTUxOCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDptb25pdG9yOm1vbml0b3IifQ.cVml5Of1fZxyv-hRUKnqWWNK_52_btbdISvmP1Fw6Um-D9kqq5CieymC4f5KHVdxdJnA_-54ih3No5VUfetefBryh06yX_Qr01k0TGKKU_MwXcTgKgKs1Ydet7cS3VTBgZHNERdvHmK_phSnwEA87zJUkQNIMWPjTzsAUVlk0nve60MF-EohI_RqxILntlSKRpI5X5WG1p_IT7NebA5UYeKDYoabI9-YqoEPQd6XQ6Lfc5nf_tC1gUMExyaczVZTrsxjnpsZl5cFpAGg1b4NNixTLRbqWdeuu1uV5i_WJTlYMsfPNCvb2eP8KC9d0DE8UMSDNMwrehYyrmviAGqKVQ
### 访问 kubelet 暴露的10250 端口  记得自己替换一下token
    [root@master01 ~]# curl https://127.0.0.1:10250/metrics/cadvisor -k -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IlZ2bGJjaEN2MjFwazRmLUNWdkxBYVoxUHBleTBCUFBzWW0xU25uMGM1Y3MifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJtb25pdG9yIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6Im1vbml0b3ItdG9rZW4tMmt0cjIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoibW9uaXRvciIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjcxODMyNmU2LTU3ZWMtNDkwYy05ZmNiLTYwNjk4YWNjYTUxOCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDptb25pdG9yOm1vbml0b3IifQ.cVml5Of1fZxyv-hRUKnqWWNK_52_btbdISvmP1Fw6Um-D9kqq5CieymC4f5KHVdxdJnA_-54ih3No5VUfetefBryh06yX_Qr01k0TGKKU_MwXcTgKgKs1Ydet7cS3VTBgZHNERdvHmK_phSnwEA87zJUkQNIMWPjTzsAUVlk0nve60MF-EohI_RqxILntlSKRpI5X5WG1p_IT7NebA5UYeKDYoabI9-YqoEPQd6XQ6Lfc5nf_tC1gUMExyaczVZTrsxjnpsZl5cFpAGg1b4NNixTLRbqWdeuu1uV5i_WJTlYMsfPNCvb2eP8KC9d0DE8UMSDNMwrehYyrmviAGqKVQ" | more
     % Total % Received % Xferd Average Speed Time Time Time Current
                                  Dload Upload Total Spent Left Speed
     0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0# HELP cadvisor_version_info A metric with a constant '1' value labeled by kernel version, OS version, docker version, cadvisor version & cadvisor revision.
    # TYPE cadvisor_version_info gauge
    cadvisor_version_info{cadvisorRevision="",cadvisorVersion="",dockerVersion="19.03.8",kernelVersion="3.10.0-   1062.12.1.el7.x86_64",osVersion="CentOS Linux 7 (Core)"} 1
    # HELP container_cpu_load_average_10s Value of container cpu load average over the last 10 seconds.
    # TYPE container_cpu_load_average_10s gauge
    container_cpu_load_average_10s{container="",id="/",image="",name="",namespace="",pod=""} 0 1585634068599
    container_cpu_load_average_10s{container="",id="/kubepods",image="",name="",namespace="",pod=""} 0 1585634068611
    container_cpu_load_average_10s{container="",id="/kubepods/besteffort",image="",name="",namespace="",pod=""} 0 1585634073752
    。。。
## 安装
### node-exporter
    # 这里把node-exporter部署为Pod，使用DaemonSet资源类型部署，方便维护，这样每一个kubernetes集群节点都会部署一个，资源配置清单在prometheus文件夹下。
    # 创建 DaemonSet 资源类型的 Pod
        kubectl apply -f node-exporter.yaml
        daemonset.apps/node-exporter created
    # 验证
        [root@master01 monitor]# curl http://127.0.0.1:9100/metrics|more
          % Total % Received % Xferd Average Speed Time Time Time Current
                                         Dload Upload Total Spent Left Speed
          0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0# HELP go_gc_duration_seconds A summary of the GC invocation durations.
        # TYPE go_gc_duration_seconds summary
        go_gc_duration_seconds{quantile="0"} 0
        go_gc_duration_seconds{quantile="0.25"} 0
        go_gc_duration_seconds{quantile="0.5"} 0
        go_gc_duration_seconds{quantile="0.75"} 0
        go_gc_duration_seconds{quantile="1"} 0
        go_gc_duration_seconds_sum 0
        go_gc_duration_seconds_count 0
        # HELP go_goroutines Number of goroutines that currently exist.
        # TYPE go_goroutines gauge
        go_goroutines 6
    # 查看Pod
        [root@master01 monitor]# kubectl get pods -n monitor -o wide
        NAME READY STATUS RESTARTS AGE IP NODE NOMINATED NODE READINESS GATES
        node-exporter-c67rd 1/1     Running 0          11m   172.31.117.228   node01 <none>           <none>
        node-exporter-jrzfx 1/1     Running 0          11m   172.31.117.227   master03 <none>           <none>
        node-exporter-mqsw5 1/1     Running 0          11m   172.31.117.225   master01 <none>           <none>
        node-exporter-zhnl4 1/1     Running 0          11m   172.31.117.226   master02 <none>           <none>
        # 从上面可以看出，已经监控到所有宿主机 CPU、内存、负载、网络流量、文件系统等指标信息，后续可供 Prometheus 收集。
### kube-state-metrics
    # Kube-state-metrics 它是通过监听 kube-apiserv括r 而生成有关资源对象的指标信息，主要包括Node、Pod、Service 、Endpoint、Namespace等资源的metric，需要注意的是kube-state-  metrics只是简单的提供一个metrics数据，并不会存储这些指标数据，后续可以使用Prometheus 来抓取这些数据然后存储，它主要关注的是业务资源workload的元数据信息。
    # 这里也需要一个ServiceAccount帐户并授权绑定
    #  创建 Pod 及service 配置文件
    # 部署及查看
        [root@master01 monitor]# kubectl apply -f kube-state-metrics-rbac.yaml
            serviceaccount/kube-state-metrics created
            clusterrole.rbac.authorization.k8s.io/kube-state-metrics created
            clusterrolebinding.rbac.authorization.k8s.io/kube-state-metrics created
        [root@master01 monitor]# kubectl apply -f kube-state-metrics-deployment-svc.yaml
            deployment.apps/kube-state-metrics created
            service/kube-state-metrics created
        [root@master01 monitor]#
    # 查看部署情况
        [root@master01 monitor]# kubectl get clusterrolebinding |grep kube-state
        kube-state-metrics ClusterRole/kube-state-metrics 4m4s
        [root@master01 monitor]#
        [root@master01 monitor]# kubectl get pods -n kube-system |grep kube-state-metrics
        kube-state-metrics-84b8477f75-65gcg 1/1     Running 0          4m26s
### metrics-server
    # 前期准备
    # 在较早的版本中，集群监控使用的是 heaspter，集群通过它的监控指标实现HPA、VPA和kubectl top等，在新版本中由 metrics-server 替代，至于原因，可以Google一下。metrics-server 是 kubernetes 监控体系中的核心组件之一，从 kubelet 中收集 Pod/Node 等资源指标，然后对这些指标数据进行聚合，最后再通过 Kube-apiserver 中 Metrics API( /apis/metrics.k8s.io/)公开暴露，metrics-server只存储最新的指标数据（CPU/Memory），并不会把指标数据转发给第三方目标，如果想使用 Metrics-server 指标数据，就需要对集群做一些特殊的配置，这些配置默认情况下，是不会安装的，具体配置如下几点，1、kube-apiserver要能访问到metrics-server；2、kube-apiserver启用参数中启用聚合层功能；3、组件要有kubectl的认证配置并且绑定到Metrics-server；4、Pod/Node指标需要由Summary API通过Kubelet公开。
    [root@master01 ~]# cd /etc/kubernetes/manifests/
    [root@master01 manifests]# ls
    kube-apiserver.yaml kube-controller-manager.yaml kube-scheduler.yaml
    [root@master01 manifests]# pwd
    /etc/kubernetes/manifests
    #  二进制安装的话，进入到以上目录，并修改kube-apiserver.yaml，主要是加上- --enable-aggregator-routing=true，其它的默认应该是有的，修改如下配置：
          .............
          - --requestheader-allowed-names=front-proxy-client
            - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
            - --requestheader-extra-headers-prefix=X-Remote-Extra-
            - --requestheader-group-headers=X-Remote-Group
            - --requestheader-username-headers=X-Remote-User
            - --enable-aggregator-routing=true
          .............
    # 部署metrics-server
      kubectl apply -f components.yaml
## 最终效果
        [k8s-master@k8s-master metrics-server]$ kubectl top node
        NAME         CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
        k8s-master   965m         24%    1668Mi          53%       
        k8s-node1    476m         11%    793Mi           46%       
        k8s-node2    275m         6%     661Mi           38%       
        [k8s-master@k8s-master metrics-server]$ kubectl top pods
        NAME         CPU(cores)   MEMORY(bytes)   
        volume-pod   7m           74Mi            
## 坑1
        root@master01 kubernetes]# kubectl top node
        error: metrics not available yet
        [root@master01 kubernetes]#
        # 查看错误日志
        unable to fully collect metrics: [unable to fully scrape metrics from source kubelet_summary:master01: unable to fetch metrics from Kubelet master01 (master01): Get https://master01:10250/stats/summary?only_cpu_and_memory=true: dial tcp: lookup master01 on 10.96.0.10:53: no such host, unable to fully scrape metrics from source kubelet_summary:master03: unable to fetch metrics from Kubelet master03 (master03): Get https://master03:10250/stats/summary?only_cpu_and_memory=true: dial tcp: lookup master03 on 10.96.0.10:53: no such host, unable to fully scrape metrics from source kubelet_summary:master02: unable to fetch metrics from Kubelet master02 (master02): Get https://master02:10250/stats/summary?only_cpu_and_memory=true: dial tcp: lookup master02 on 10.96.0.10:53: no such host, unable to fully scrape metrics from source kubelet_summary:node01: unable to fetch metrics from Kubelet node01 (node01): Get https://node01:10250/stats/summary?only_cpu_and_memory=true: dial tcp: lookup node01 on 10.96.0.10:53: no such host]
        # 这个坑的解决方式是
        - --kubelet-insecure-tls ，修改 components.yaml文件中的metrics-server-deployment部分， 添加这个参数，删除再重新创建
## 坑2
        [root@master01 kubernetes]# kubectl top node
        Error from server (ServiceUnavailable): the server is currently unable to handle the request (get nodes.metrics.k8s.io)
        [root@master01 kubernetes]#

        [root@master01 kubernetes]# kubectl logs -f metrics-server-64b57fd654-bt6fx -n kube-system
        E0331 07:03:59.658787       1 reststorage.go:135] unable to fetch node metrics for node "master03": no metrics known for node
        E0331 07:03:59.658793       1 reststorage.go:135] unable to fetch node metrics for node "node01": no metrics known for node
        # 这个坑的解决方式是：
        添加 - --kubelet-preferred-address-types=InternalIP 启动参数 ，修改 componentes.yaml 中的metrics-server-deployment部分， 添加这个参数，最终如下所示，再删除重建即可
                args:
                  - --cert-dir=/tmp
                  - --secure-port=4443
                  - '--kubelet-preferred-address-types=InternalIP'
                  - '--kubelet-insecure-tls'

