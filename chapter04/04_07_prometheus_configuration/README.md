# ch04-07 [실습] Prometheus 설정

## prometheus 설정파일
kube-prometheus/manifests/prometheus-*.yaml파일을 참고한다.
```
cd kube-prometheus/manifests
ls prometheus-*.yaml -alh
prometheus-clusterRole.yaml
prometheus-clusterRoleBinding.yaml
prometheus-podDisruptionBudget.yaml
prometheus-prometheus.yaml
prometheus-prometheusRule.yaml
prometheus-roleBindingConfig.yaml
prometheus-roleBindingSpecificNamespaces.yaml
prometheus-roleConfig.yaml
prometheus-roleSpecificNamespaces.yaml
prometheus-service.yaml
prometheus-serviceAccount.yaml
prometheus-serviceMonitor.yaml
```

## prometheus 서비스 설정
prometheus-service.yaml 는 기본적으로 ClusterIP로 설정되어있는데 외부접속을 위해 LoadBalancer나 NodePort로 설정
kubectl get svc -n monitoring prometheus-k8s

## prometheus-prometheus.yaml
prometheus CR를 설정한다

- Prometheus CR을 통해 prometheus를 어떻게 배포할지, 프로메테우스에 적용할 서비스모니터, 파드모니터, 룰 CR의 레이블을 설정한다.
- 프로메테우스 어플리케이션을 배포할 때 replica 개수, 사용하는 컨테이너 이미지, request memory, security context등을 설정할 수 있다.

## 예시) ServiceMonitor Namespace Selector 설정
prometheus가 모니터링 타겟을 특정 네임스페이스의 서비스모니터만 사용하도록 변경
Service Discovery > $ kubectl get serviceMonitor -A
```
prometheus-prometheus.yaml
  serviceMonitorNamespaceSelector:
    matchLabels:
      monitor: enabled
```
> $ kubectl get ns --sohw-labels
> $ kubectl apply -f ~
```
kubectl logs -n monitoring prometheus-operator-7ddc6877d5-ksbmt -f
kubectl logs -n monitoring prometheus-k8s-0
```
> Service Discovery에 아무런 target이 없어지는 것을 확인할 수 있다

monitoring할 것을 label 추가
```
kubectl label ns monitoring monitor=enable
```
> operator log에 설정 반영된 것을 확인할 수 있다
> Service Discovery에 아무런 target이 추가된 것을 확인할 수 있다


## prometheus Rule
prometheus에서 alert이나 recording rule을 적용하기 위해서는 prometheusRule CR을 생성해야한다. manifests폴더에 `*-prometheusRule.yaml` 파일들을 보면 kube-prometheus에서 제공하는 prometheusRule CR 내용을 볼 수 있는데, 새로 CR을 생성하여 Rule을 추가하거나, 기존 Rule CR파일을 수정할 수 있다.


## prometheus serviceMonitor
serviceMonitor CR을 설정한다.
prometheus에서 기본적으로 모니터링하는 대상을 쿠버네티스 서비스를 사용하여 검색한다. manifests폴더에 `*-serviceMonitor.yaml` 파일들을 보면 kube-prometheus에서 제공하는 serivce monitor CR 내용을 볼 수 있는데, 새로 CR을 생성하여 service monitor를 생성하거나, 기존 serivce monitor CR파일을 수정할 수 있다.

# 마무리
`kube-prometheus`의 `Prometheus-operator`는 prometheus, prometheusRule, serviceMonitor Custrom Resource를 사용하고 있다. 따라서 프로메테우스를 설정하기 위해서는 
해당 리소스를 변경하고, 이를 kubectl로 적용하여 환경설정이 변경되는지`prometheus-operator`와 `prometheus` pod 로그를 통해 확인할 수 있다.