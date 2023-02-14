# 04-05 Prometheus와 Grafana를 통한 쿠버네티스 모니터링

## kube-prometheus 저장소
https://github.com/prometheus-operator/kube-prometheus

## Prometheus 저장소 복사
```
git clone https://github.com/prometheus-operator/kube-prometheus.git -b release-0.10
```

## Manifest description
```
manifest
manifest/setup
```

## 설치 
```
$ kubectl create -f manifest/setup/
```
> CRD, namespace 
```
$ kubectl create -f manifest
```
> others install
> $ kubectl get pods -n monitoring
>> alertmanager, blackbox, node-exporter, prometheus-adapter
> $ kubectl get svc -n monitoring

## svc external IP 연결
```
vi manifests/grafana-service.yaml
vi manifests/prometheus-service.yaml
# type 추가
type: LoadBalancer

kubectl apply -f manifests/grafana-service.yaml
kubectl apply -f manifests/prometheus-service.yaml
```
> $ kubectl apply -f ~
> $ kubectl get svc -n monitoring

## Prometheus UI & Metrics 확인
```
port: 9090
search: prometheus_http_requests_total
```
> graph

```
status > targets
```
> prometheus를 통해 모니터링 대상 (svc monitor)
>> ls manifests/*serviceMonitor* - alh
>> $ kubectl get servicemonitor node-exporter -n monitoring -o yaml
>> svc와 matchLabel(Selector)

node-exporte 메트릭을 확인해본다.

```
sudo ss -lp|grep 9100
curl http://localhost:9100/metrics
```

## Grafana
```
- port:3000
- 최초 ID/PW: admin/admin
- 다양한 Dashboard 확인
```


