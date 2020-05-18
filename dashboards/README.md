# dashboard组件  
       dashboard组件的安装
## 部署dashboard
  直接执行dashboard.yaml  
  结果： 运行2个pod和service
## 暴露端口
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
