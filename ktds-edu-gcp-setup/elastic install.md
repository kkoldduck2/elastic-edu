# Elastic helm install



sudo k3s server --server https://10.42.0.1:6443 --token K1039de57483f87b03f41e018f7aff663ce796820be7a9321b9caa7abb63d56fe98::server:14c8e56b8420d561d630dccb51dd83fa & 

172.28.245.240

elastic 설치하기





https://nangman14.tistory.com/65

git clone https://github.com/elastic/helm-charts.git

- Install it: `helm install elasticsearch ./helm-charts/elasticsearch --set imageTag=8.5.1`



# GCP 설정하기

```bash
#master node 연결
export MASTER_TOKEN="K10689843edd92e9b972f10a0e9161aabd2c62645c0116679c28065cc574d7453f7::server:bcd343bbcfd96eebcec90ea679a6012c"
export MASTER_IP="10.182.0.2"

=> 안되서 수동 실행함

sudo k3s agent --server https://10.182.0.2:6443 --token K10689843edd92e9b972f10a0e9161aabd2c62645c0116679c28065cc574d7453f7::server:bcd343bbcfd96eebcec90ea679a6012c &



root@master:/home/goworkj# export MASTER_TOKEN="K10689843edd92e9b972f10a0e9161aabd2c62645c0116679c28065cc574d7453f7::server:bcd343bbcfd96eebcec90ea679a6012c"
root@master:/home/goworkj# export MASTER_IP="10.182.0.2"
root@master:/home/goworkj# sudo k3s server --server https://10.182.0.2:6443 --token K10689843edd92e9b972f10a0e9161aabd2c62645c0116679c28065cc574d7453f7::server:bcd343bbcfd96eebcec90ea679a6012c &


# 교육 정보 clone
git clone https://github.com/hiviv/elastic-edu.git
```



# Key 생성하기

```bash 
ssh-keygen -t rsa -b 4096 -C edu01
cat id_rsa.pub

메타데이터에 key 입력
```



# helm 설치하기





# 배포하기

배포하는 것 역시 너무 간단하다. 다음 명령어로 elastic helm repo를 가져온 후, 이전에 준비한 values.yaml을 이용하여 배포면 된다. namespace는 es로 설정하였다.

```bash
helm repo add elastic https://helm.elastic.co
helm install elasticsearch elastic/elasticsearch -n es --create-namespace

wget https://github.com/elastic/helm-charts/tree/main/elasticsearch/values.yaml
wget https://github.com/elastic/helm-charts/tree/main/kibana/values.yaml
```

```bash
# elasticsearch value 파일로 배포하기
$ cd elastic-edu/helm/
$ helm install elasticsearch -f ./elasticsearch-values.yaml elastic/elasticsearch -n es --create-namespace &
$ helm install elasticsearch elastic/elasticsearch -f ./elasticsearch-values.yaml -n es --create-namespace &

# log 확인
$ k logs -f elasticsearch-master-0 -n es

# port forward
$ kubectl port-forward -n es elasticsearch-master-0 9200:9200 &

# curl로 접속 확인
$ curl https://localhost:9200 -k -u 'elastic:new1234!'

# workernode1에서 실행
# kibana value 파일로 배포하기
# old helm install kibana ./helm-charts/kibana -n es
$ helm install kibana -f ./kibana-values.yaml elastic/kibana -n es --create-namespace &
$ helm install kibana elastic/kibana -f ./kibana-values.yaml -n es &

$ k logs -f svc/kibana-kibana -n es
$ k describe pod/kibana-kibana-99c648b97-pcb5m -n es

# port forward
$ kubectl port-forward --namespace es svc/kibana-kibana 5601:5601 &
$ curl https://localhost:5601

```


```bash
# PVC, Secret 수동 삭제
$ kubectl get pvc
$ kubectl patch pvc elasticsearch-master-elasticsearch-master-0  -p '{"metadata":{"finalizers":null}}' -n es
$ kubectl patch pvc elasticsearch-master-elasticsearch-master-1  -p '{"metadata":{"finalizers":null}}' -n es
$ kubectl patch pvc elasticsearch-master-elasticsearch-master-2  -p '{"metadata":{"finalizers":null}}' -n es
$ kubectl delete pvc elasticsearch-master-elasticsearch-master-0 -n es
$ kubectl delete pvc elasticsearch-master-elasticsearch-master-1 -n es
$ kubectl delete pvc elasticsearch-master-elasticsearch-master-2 -n es
$ kubectl get secret
$ kubectl delete secret elasticsearch-master-certs -n es
$ kubectl delete secret elasticsearch-master-credentials -n es

```

