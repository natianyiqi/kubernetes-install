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

