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
    # 这里把node-exporter部署为Pod，使用DaemonSet资源类型部署，方便维护，这样每一个kubernetes集群节点都会部署一个，资源配置清单如下：
    cat node-exporter.yaml
        apiVersion: apps/v1
        kind: DaemonSet
        metadata:
          name: node-exporter
          namespace: monitor
          labels:
            name: node-exporter
        spec:
          selector:
            matchLabels:
             name: node-exporter
          template:
            metadata:
              labels:
                name: node-exporter
            spec:
              hostPID: true
              hostIPC: true
              hostNetwork: true
              containers:
              - name: node-exporter
                image: prom/node-exporter:latest
                ports:
                - containerPort: 9100
                resources:
                  requests:
                    cpu: 0.15
                securityContext:
                  privileged: true
                args:
                - --path.procfs
                - /host/proc
                - --path.sysfs
                - /host/sys
                - --collector.filesystem.ignored-mount-points
                - '"^/(sys|proc|dev|host|etc)($|/)"'
                volumeMounts:
                - name: dev
                  mountPath: /host/dev
                - name: proc
                  mountPath: /host/proc
                - name: sys
                  mountPath: /host/sys
                - name: rootfs
                  mountPath: /rootfs
              tolerations:
              - key: "node-role.kubernetes.io/master"
                operator: "Exists"
                effect: "NoSchedule"
              volumes:
                - name: proc
                  hostPath:
                    path: /proc
                - name: dev
                  hostPath:
                    path: /dev
                - name: sys
                  hostPath:
                    path: /sys
                - name: rootfs
                  hostPath:
                    path: /
            # 这里使用hostnetwork为true，使用宿主机网络，会监控在宿主机上面的9100口；
## 验证
### 创建 DaemonSet 资源类型的 Pod
        [root@master01 monitor]# kubectl apply -f node-exporter.yaml
        daemonset.apps/node-exporter created
        [root@master01 monitor]#

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
### 查看Pod
    [root@master01 monitor]# kubectl get pods -n monitor -o wide
    NAME READY STATUS RESTARTS AGE IP NODE NOMINATED NODE READINESS GATES
    node-exporter-c67rd 1/1     Running 0          11m   172.31.117.228   node01 <none>           <none>
    node-exporter-jrzfx 1/1     Running 0          11m   172.31.117.227   master03 <none>           <none>
    node-exporter-mqsw5 1/1     Running 0          11m   172.31.117.225   master01 <none>           <none>
    node-exporter-zhnl4 1/1     Running 0          11m   172.31.117.226   master02 <none>           <none>
    # 从上面可以看出，已经监控到所有宿主机 CPU、内存、负载、网络流量、文件系统等指标信息，后续可供 Prometheus 收集。
        kube-state-metrics
        Kube-state-metrics 它是通过监听 kube-apiserv括r 而生成有关资源对象的指标信息，主要包括Node、Pod、Service 、Endpoint、Namespace等资源的metric，需要注意的是kube-state-  metrics只是简单的提供一个metrics数据，并不会存储这些指标数据，后续可以使用Prometheus 来抓取这些数据然后存储，它主要关注的是业务资源workload的元数据信息。
### 这里也需要一个ServiceAccount帐户并授权绑定
        [root@master01 monitor]# cat kube-state-metrics-rbac.yaml
        ---
        apiVersion: v1
        kind: ServiceAccount
        metadata:
          name: kube-state-metrics
          namespace: kube-system
        ---
        apiVersion: rbac.authorization.k8s.io/v1
        kind: ClusterRole
        metadata:
          name: kube-state-metrics
        rules:
        - apiGroups: [""]
          resources: ["nodes", "pods", "services", "resourcequotas", "replicationcontrollers", "limitranges", "persistentvolumeclaims", "persistentvolumes", "namespaces", "endpoints"]
          verbs: ["list", "watch"]
        - apiGroups: ["extensions"]
          resources: ["daemonsets", "deployments", "replicasets"]
          verbs: ["list", "watch"]
        - apiGroups: ["apps"]
          resources: ["statefulsets"]
          verbs: ["list", "watch"]
        - apiGroups: ["batch"]
          resources: ["cronjobs", "jobs"]
          verbs: ["list", "watch"]
        - apiGroups: ["autoscaling"]
          resources: ["horizontalpodautoscalers"]
          verbs: ["list", "watch"]
        ---
        apiVersion: rbac.authorization.k8s.io/v1
        kind: ClusterRoleBinding
        metadata:
          name: kube-state-metrics
        roleRef:
          apiGroup: rbac.authorization.k8s.io
          kind: ClusterRole
          name: kube-state-metrics
        subjects:
        - kind: ServiceAccount
          name: kube-state-metrics
          namespace: kube-system
###  创建 Pod 及service 配置文件
        [root@master01 monitor]# cat kube-state-metrics-deployment-svc.yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: kube-state-metrics
          namespace: kube-system
        spec:
          replicas: 1
          selector:
            matchLabels:
              app: kube-state-metrics
          template:
            metadata:
              labels:
                app: kube-state-metrics
            spec:
              serviceAccountName: kube-state-metrics
              containers:
              - name: kube-state-metrics
                image: quay.io/coreos/kube-state-metrics:v1.9.5
                ports:
                - containerPort: 8080

        ---
        apiVersion: v1
        kind: Service
        metadata:
          annotations:
            prometheus.io/scrape: 'true'
          name: kube-state-metrics
          namespace: kube-system
          labels:
            app: kube-state-metrics
        spec:
          ports:
          - name: kube-state-metrics
            port: 8080
            protocol: TCP
          selector:
            app: kube-state-metrics
###  部署及查看
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
        
