### 03-02 kubeapiserver
# 1. kubeapiserver에 api 요청하기
1. 클러스터 ca-cert 획득
- ~/.kube/config
- $ kubectl config view --minift --raw --output 'jsonpath={..cluster.certificate-authority-data}' | base64 -d > cluster-ca.crt
2. user cert key 획득
- $ kubectl config vie --minify --raw --output 'jsonpath={..user.client-certificate-data}' | base64 -d > user.crt
- $ kubectl config vie --minify --raw --output 'jsonpath={..user.client-key-data}' | base64 -d > user.key
3. curl 테스트
- $ curl --cacert cluster-ca.crt https://<kubernetes-server-url:6443>
> 인증 error
- $ curl --cacert cluster-ca.crt --cert user.crt --key user.key https://<kubernetes-server-url:6443>