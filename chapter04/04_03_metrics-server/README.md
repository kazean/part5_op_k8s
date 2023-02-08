# 04-01 metrics-server
- kubectl get nodes > v1.21.11
- gke controller: 3, workter: 4
- metrics-server github repo

## 1. metrics-server install 
1. github repo, metrics-server install
```
wget https://~ download > components.yaml
SA, Role, Service, Deployment, APISerice
$ kubectl apply -f conponents.yaml
> $ kubectl get po -n kube-system  -w
>> metrics-server
```

2. metrics-server insstructions
```
$ kubectl top node/pod
> $ kubectl top node, kubectl top pod -n kube-system (15s 마다 갱신)
```

3. api
```
$ kubectl get --raw "/apis/metrics.k8s.io/v1beta/nodes" | jq '.'
$ kubectl get --raw "/apis/metrics.k8s.io/v1beta/nodes/<node명>" | jq '.'
$ kubectl get --raw "/apis/metrics.k8s.io/v1beta/namespaces/kube-system/pods" | jq '.'
$ kubectl get --raw "/apis/metrics.k8s.io/v1beta/namespaces/kube-system/pods/<pod명>" | jq '.'
```

## 2. HPA(Horizontal Pod Autoscaler)
```
- kubenets 공식 문서: hpa example application.yaml (Deployment, Service)
> $ kubectl apply -f application.yaml
- autoscale set
> $ kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
>> $ kubectl get hpa -w
```

- 부하 증가시키기
```
> $ kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
- 부하 컨테이너 종료후 hpa 확인
> $ kubectl get hpa -w
> $ kubectl get deploy
```