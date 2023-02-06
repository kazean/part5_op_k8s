### 03-09 Calico CNI 설치
- 워커노드까지 설치된 환경: worker node NotReady

# 1. Calico 설치
- calico yaml download 후 설치
- $ kubectl apply -f calico.yaml
>> configmap, CRD, ClusterRole, ClusterRoleBinding, RoleBinding Creating
>> kubectl get po -n kube-system (calico-kube-controller, calico-node, coredns)
>> kubectl get nodes > NodeReady, kubectl describe node <node>
