### ch02-10 kubeadm를 통해 kubernetes 구축하기
# 1. 구글 클라우드에 대한 사전 설정
> gcloud init
> (계정 설정)

> (set project)default project, 새로운 demo project: my-projet-for-demo
> default Compute region: asia-northeast3-a(서울)
> gcloud config list

> (set network)gcloud networks create fastcampus-k8s --subnet-mode custom
> gcloud compute networks subnets create k8s-nodes \
> --network fastcampus-k8s \
> --range 10.240.0.0/24

> (set firewall) gcloud compute firewall-rules create fastcampus-k8s-allow-internal \
> --allow tcp,udp,icmp,ipip \
> --network fastcampus-k8s \
> --source-range 10.240.0.0/24

> gcloud firewall-ruels create fastcampus-k8s-allow-external \
> --allow tcp:22,tcp:6443,icmp \
> --network fastcampus-k8s \
> --source-ranges 0.0.0.0/0

> (public ip 확인) gcloud compute addresses list
> gcloud create addresses create fastcampus-k8s
>> gcloud compute addresses list

> (set cloud nat)
> gcloud compute routers create k8s-router \
  --network fastcampus-k8s \
  --region asia-northeast3 

> gcloud compute routers nats create k8s-nat \
  --router-region asia-northeast3 \
  --router k8s-router
  --nat-all-subnet-ip-ranges \
  --auto-allocate-nat-external-ips


### ch02-11 Master Node 구축
# 1. 인스턴스 구축
> gcloud compute instances create controller-2
for i in 0 1 2; do gcloud compute instances create controller-${i} \
 --async \
 --boot-disk-size 200GB \
 --can-ip-forward \
 --image-family ubuntu-2004-lts \
 --image-project ubuntu-os-cloud \
 --machine-type e2-standard-2 \
 --private-netowrk-ip 10.240.0.1${i} \
 --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
 --subnet k8s-nodes \
 --tags fastcampus-k8s,controller
 done
>> gcloud compute instances list

# 2. node에 접속하여 kubeadm을 통한 설정
> gcloud compute ssh controller-0 ~2
> sudo apt update
> sudo apt -y install apt-transport-https
> (gcloud cli&key)curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
> (쿠버네티스 저장소)echo "deb https://apt.kubernets.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
> sudo apt update
>> 에러시 여러번 수행
> sudo apt -y install vim git curl wget kubelet kubeadm kubectl
> (cf,버전명시버전) sudo apt -y install vim git curl wget kubelet=1.21.11-0 kubeadm=1.21.11-0 kubectl=1.21.11-0
> (hold 강제로 update 되는것을 방지)sudo apt-mark hold kubelet kubeadm kubectl
> (버전확인) kubectl version --client && kubeadm version (1.23.5)
# 3. 쿠버네티스 설치전 설정 옵션
> (node서버는 메모리 swap 옵션 disable/삭제) sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
> sudo swapoff -a

# 4. containerD 설치
> sudo tee /etc/moduels-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
> sudo modprobe overlay
> sudo modprobe br_netfilter
>

> (k8s 필요한 설정 enable) sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward =1 
EOF
> (최신화)sudo sysctl --system

> (containerd 설치시 필요한 패키지 설치)
> sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
> (공개키 다운로드) curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
> (apt 저장소 추가) sudo apt-add-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
> sudo apt update
> (containerd install) sudo apt install -y containerd.io
>> systemctl status containerd

# 5. containerd 설정
> sudo su -
> mkdir -p /etc/containerd
> containerd config default>/etc/containerd/config.toml
> sudo systemctl restart containerd
> sudo systemctl enable containerd
> systemctl status containerd

> (containerd c group 설정)
> vi /etc/containerd/config.toml
> SystemdCgroup = true
> sudo systemctl restart containerd
> sudo systemctl enable containerd

# 6. kubelet 실행
> sudo systemctl enable kubelet
> sudo systemctl start kubelet
> sudo systemctl status kubelet
>> 정상기동 안됨: 기반 설정이 안되있어서
> sudo systemctl stop kubelet
> (추후 kubeadm을 통해서 활성화)


# 7. 기타 노드 설정
> (퍼블릭 클라우드 별 cloud control manager, 어떤 클라우드 사용할 것인지 설정) 
> cat << EOF | sudo tee /etc/kubernetes/cloud-config
[Global]
project-id  = <project-id>
EOF
>> project-id 확인 gcloud config list
>> cat /etc/kubernetes/cloud-confi

# 8. 구글 클라우드 설정
> (Loadbalancer 설정 - externalip)
> (lb 방화벽 설정) 
> gcloud compute firewall-rules create fw-allow-network-lb-health-chekcs \
  --network=fastcampus-k8s \
  --action=ALLOW
  --direction=INGRESS
  --source-ranges=35.191.0.0./16,209.85.152.0/22,209.85.204.0/22 \
  --target-tages=allow-network-lb-health-checks \
  --rules=tcp

> (컨트롤러 노드 등록)
> gcloud compute instance-groups unmanaged create k8s-master \
  --zone=asia-northeast3-a
> gcloud compute instance-groups unmanaged add-instances k8s-master \
  --zone=asia-nrotheast3-a \
  --instances=controller-0

> (lb health check rules)
> gcloud compute health-checks create https k8s-controller-hc --check-interbal=5 \
  --enable-logging \
  --request-path=/healthz \
  --port=6443 \
  --region=asia-northeast3

> (backend service 구성: hc, instances group)
> gcloud compute backend-services cretae k8s-service \
  --protocol TCP \
  --health-checks k8s-controller-hc \
  --health-checks-region asia-northeast3 \
  --region asia-northeast3
> gcloud compute backend-service add-backend k8s-service \
  --instance-group k8s-master \
  --instance-group-zone asia-northeast3-a \
  --region asia-northeast3

> gcloud compute forwarding-rules create k8s-forwarding-rule \
  --load-balancing-scheme extenral \
  --region asia-northeast3 \
  --port 6443 \
  --address fastcampus-k8s \
  --backend-service k8s-service

# 9. kubeadm 실행: cluster 생성 및 설정 (controller-0)
> cat << EOF > kubeadmcfg.yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: InitConfiguration
nodeRegisteration:
  criSocket: "/run/containerd/containerd.sock"
  kubeletExtraArgs:
    cloud-provider: "gce"
    cloud-config: "/etc/kubernetes/cloud-config"
    cgroup-driver: "systemd"
---
apiVersion: kubeadm.k8s.io/v1beata2
kind: ClusterConfiguration
apiServer:
  extraArgs:
    cloud-provider: "gce"
    cloud-config: "/etc/kubernetes/cloud-config"
  extraVloumes:
  - name: cloud
    hostPath: "/etc/kubernetes/cloud-config"
    mountPath: "/etc/kubernetes/cloud-config"
controllerManager:
  extraArgs:
    cloud-provider: "gce"
    cloud-config: "/etc/kubernetes/cloud-config"
  extraVloumes:
  - name: cloud
    hostPath: "/etc/kubernetes/cloud-config"
    mountPath: "/etc/kubernetes/cloud-config"
networking:
  podSubnet: "192.168.0.0/16"
  serviceSubnet: "10.32.0.0/24"
controlPlaneEndpoint: "<ADD>:6443"
---
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
cgroupDriver: systemd
serverTLSBootstrap: true
EOF
> cat kubeadmcfg.yaml
>> gcloud compute addresses list : add를 controlplane endpoint에 설정  

> (kubeadm으로 k8s 설치) sudo kubeadm init --upload-certs --config kubeadmcfg.yaml
> mkidir -p $HOME/.kube
> sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
> sudo chown $(id -u):$(id -g) $/HOME/.kube/config
> kubectl get nodes

> (추가적인 노드 설정방법 desc)
> (controller-1,2) sudo kubeadm join ~~


### ch02-12 Worker Node 구축
# 1. worker instance 생성
> for i in 0 1 2; do gcloud compute instances create worker-${i} \
  --async \
  --boot-disk-size 200GB \
  --can-ip-forward \
  --image-family ubuntu-2004-lts \
  --image-project ubuntu-os-cloud \
  --machine-type e2-standard-2 \
  --metadata pod-cidr=10.200.${i}.0/24
  --private-netowork-ip 10.240.0.2${i}
  --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
  --subnet k8s-nodes \
  --tags fastcampus-k8s,worker
done
> gcloud compute instances list

# 2. 필요 package download
> gcloud compute ssh -worker-0,1,2
> sudo apt update
> sudo apt -y install apt-transport-https
> (gcloud cli&key)curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
> (쿠버네티스 저장소)echo "deb https://apt.kubernets.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
> sudo apt update
>> 실패시 여러번
> sudo apt -y install vim git curl wget kubelet kubeadm kubectl
> (hold 강제로 update 되는것을 방지)sudo apt-mark hold kubelet kubeadm kubectl
> (버전확인) kubectl version --client && kubeadm version (1.23.5)

# 3. swap 설정
> (node서버는 메모리 swap 옵션 disable/삭제) sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
> sudo swapoff -a

# 4. containerD 설치
> sudo tee /etc/moduels-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
> sudo modprobe overlay
> sudo modprobe br_netfilter

> (k8s 필요한 설정 enable) sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward =1 
EOF
> (최신화)sudo sysctl --system

> (containerd 설치시 필요한 패키지 설치)
> sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
> (공개키 다운로드) curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
> (apt 저장소 추가) sudo apt-add-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
> sudo apt update
> (containerd install) sudo apt install -y containerd.io
>> systemctl status containerd

# 5. containerd 설정
> sudo i
> mkdir -p /etc/containerd
> containerd config default>/etc/containerd/config.toml
> sudo systemctl restart containerd
> sudo systemctl enable containerd
> systemctl status containerd

> (containerd c group 설정)
> vi /etc/containerd/config.toml
> SystemdCgroup = true
> sudo systemctl restart containerd
> sudo systemctl enable containerd

# 6. kubelet 실행
> sudo systemctl enable kubelet
> kubeadm join ~~
>> (controller-0 ssh) kubectl get nodes

# 7. 서명된 인증서 활성화 작업 (controller-0)
> kubelet 인증서가 활성화가 되지 않아서 logs 안보임
- kubectl get csr (Pending)
> kubectl get csr|grep Pen|awk '{print "kubectl certificate approve "$1}'|bash
>> kubectl get csr (Pending > Approve)

