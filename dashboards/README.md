# dashboard组件  
       dashboard组件的安装  
       在 Kubernetes 社区中，有一个很受欢迎的 Dashboard 项目，它可以给用户提供一个可视化的 Web 界面来查看当前集群的各种信息。用户可以用 Kubernetes Dashboard 部署容器化的应用、监控应用的状态、执行故障排查任务以及管理 Kubernetes 各种资源。
## 部署dashboard
       直接执行dashboard.yaml  
       结果： 运行2个pod和service              namespace: kubernetes-dashboard
## 暴露端口(以下方式二选一)
 ### 命令行方式
    kubectl  patch svc kubernetes-dashboard -n kubernetes-dashboard \
    -p '{"spec":{"type":"NodePort","ports":[{"port":443,"targetPort":8443,"nodePort":30443}]}}'
 ### 修改dashboard.yaml
    #修改其中的Service部分  
    vim dashboard.yaml  
        kind: Service  
        apiVersion: v1  
        metadata:  
          labels:  
            k8s-app: kubernetes-dashboard  
          name: kubernetes-dashboard  
          namespace: kubernetes-dashboard  
        spec:  
          type: NodePort  
          ports:  
          - port: 443  
            targetPort: 8443  
            nodePort: 30443  
          selector:  
            k8s-app: kubernetes-dashboard  
## 添加admin用户
       直接执行add-admin.yaml  
## 获取token
       kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')  
       找到token复制
## 登录
        https://ip:端口  
        例子中： https://192.168.0.50:30443
