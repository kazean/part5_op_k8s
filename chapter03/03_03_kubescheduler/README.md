### 03-03 kube-scheduler
# 1. 스케줄러 필터링: Pod Constraint
- Taints(Node), Toleration(POD)
- Node Selector
- Node Affinity
- nginx pod 
> spec.container.resources.requests/limits
> ..cpu, memory
1. request, cpu 정상배포 & 초과 배포

2. Taints, Toleration
- deployment.yaml
> spec.replicas: 3
- node에 taint 명령어
> $ kubectl taint nodes worker-0 color=red:NoSchedule
> $ kubectl taint nodes worker-1 color=red:NoSchedule
> $ kubectl taint nodes worker-2 color=red:NoSchedule
> $ kubectl apply -f deployment.yaml
>> pod: Pending, 3 node had taint
>> toleration이 없다면 스케줄링 되지 않는다

- deployment.yaml toleration추가
> spec.template.spec.tolerations
>   key: "color"
>   operator: "Equal"
>   value: "blue"
>   effect: "NoSchedule"
>> 배포되지 않는다.
- value: "blue" > "red"
>> 정상배포