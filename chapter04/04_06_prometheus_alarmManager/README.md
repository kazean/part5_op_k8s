# Ch04_06. [실습] Prometheus AlarmManager을 통한 알람 수신

## AlertManager
### 기존 알람 확인
```
Status > Rule 다양한 Rule이 등록되어있음
ls manifest/*prometheusRule*
$ kubectl get prometheusRule -n monitoring
```

## AlertManager 설정
```
port: 9093
alarm UI 제공
설정: manifests/alertmanager-secret.yaml
```

## Slack (receiver) 설정
```
- 채널 추가: fastcampus
- webhook plugin 추가
> 설정 및 관리 > 앱 관리 : 수신 웹 후크 > 추가 > 채널 선택
- 웹후크 URL
$ vi alertmanager-secret.yaml
"receivers"
- "name": "Slack"
  "slack_configs" 
  - "channel": "#fastcampus"
  - "api_url": <추가!>
... 추가
"route"
  "routes"
$ kubectl apply -f ~
```
> $ kubectl get secret -n monitoring
> $ kubectl get secret -n monitoring alertmanager-main-generated -o yaml
>> encoding 된 secret > decoding 
 
## 테스트용 Rule 설정
`manifests/kubernetesControlPlane-prometheusRule.yaml`에서 `CPUThrottlingHigh` 참고
```
vi test-prometheusRule.yaml
spec:
  groups:
  - name: test
    rules:
    - alert: CPUThrottlingHighTest
      annotations:
        description: '{{ $value | humanizePercentage }} throttling of CPU in namespace
          {{ $labels.namespace }} for container {{ $labels.container }} in pod {{
          $labels.pod }}.'
        runbook_url: https://runbooks.prometheus-operator.dev/runbooks/kubernetes/cputhrottlinghigh
        summary: Processes experience elevated CPU throttling.
      expr: |
        sum(increase(container_cpu_cfs_throttled_periods_total{container!="", namespace="default" }[5m])) by (container, pod, namespace)
          /
        sum(increase(container_cpu_cfs_periods_total{}[5m])) by (container, pod, namespace)
          > ( 25 / 100 )
      for: 1m
      labels:
        severity: critical

kubectl apply -f test-prometheusRule.yaml
```
> $ kubectl get prometheusrule -n monitoring
>> test-rule
> Prometheus  UI에서 확인

## 부하 발생
```
php-apache pod 생성
$ kubectl run -i -tty load-generator --rm --image=busybox --restart=Never ....
```
> grafana를 통해 cpu부하 확인 및 prometheus에서 alertManager를 통해 alarm 생성
