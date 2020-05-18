# 外部服务发现ingress
        ingress提供http域名访问方式，增加了7层的识别能力，可以根据 http header, path 进行路由转发，原有的NodePort暴露方式占用端口严重，4层协议转发，无法
        根据http header 和 path 进行路由转发。
## 部署ingress-controller
        直接执行ingress-controller.yaml 创建ingress-controller  
        产生多个ingress-controller     namespace:   ingress-nginx
## 部署ingress
            根据下列配置代码修改即可
            apiVersion: networking.k8s.io/v1beta1          # kubernetes1.16之后改为该api，之前为extensions/v1beta1
            kind: Ingress
            metadata: 
               name: grafana                                 
               namespace: istio-system                     # namespace
            spec: 
               rules: 
               - host: grafana.prometheus.natian.com       # 访问ip域名（自行修改）
                 http:
                   paths: 
                   - backend: 
                       serviceName: grafana                # 服务名称
                       servicePort: 3000                   # 服务端口
             修改完成后执行即可。
## 修改hosts
    修改宿主机上的hosts文件 （windows位置： C:\Windows\System32\drivers\etc）
    增加该域名   
      192.168.0.50    grafana.prometheus.natian.com
## 访问
    打开宿主机上的浏览器，访问该域名即可。
