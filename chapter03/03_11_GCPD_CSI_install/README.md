### 03-11 GCPD(GCP Compute Persistent Disk) CSI 설치
- GCPD github
# 1. controller-0 node
1. init 설정 
- $ gcloud init -h
- $ gcloud init --no-browser --console-only

2. csi driver 설치 기본구조 생성
- mkdir go, go/bin, go/pkg, go/src
- export GOPATH=$HOME/go
- ~/go/src > git clone

- csi driver 설정
> export 환경변수들 설정 후
> ~/go/src/sigs.k8s.io/gcp-compute-persistent-disk-csi-driver/deploy/setup-project.sh 실행
>> SA 생성

3. deploy에 csi driver 적용
- ~/deploy/kubernetes
- export GCE_PD_DRIVER_VERSION=stable-master (수동생성과 자동생성과는 변수가 다름)
- deploy-driver.sh 실행
>> SA, daemonset, ... 설정됨
>> $ kubectl get po -n gce-pd-csi-driver

4. test yaml
- sc 생성
> ~/example/kubernetes
> $ kubectl get sc
> demo-zonal-sc.yaml
>> $ kubectl apply -f demo-zonal-sc.yaml

- volume 생성
> demo-pod.yaml
> pvc, pod
>> $ kubectl apply -f demo-pod.yaml