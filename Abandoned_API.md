# Kubernetes v1.16 will remove the obsolete API.
      # As the development of Kubernetes， API are changing all the time. The obsolete API will be replaced by the new API.
      # According to the latest Version, Kubernetes v1.16 will adjust the API for the following 4 services.
## 
  | Categories | New | Old |
  | ------ | ------ | ------ |
  | NetworkPolicy | networking.k8s.io/v1 | extensions/v1beta1 |
  | PodSecurityPolicy | policy/v1beta1 | extensions/v1beta1 |
  | DaemonSet、Deployment、StatefulSet、ReplicaSet | apps/v1 | extensions/v1beta1、apps/v1beta1、apps/v1beta2 |
  | Ingress | networking.k8s.io/v1beta1 | extensions/v1beta1 |
