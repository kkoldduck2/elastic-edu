# Bitnami로 Elastic Stack 설치하기



참고 페이지 : https://soojae.tistory.com/88

https://github.com/bitnami/charts/tree/df139fd02a7ac5ab26b165d72fb9575a284e0304/bitnami/elasticsearch



* **bitnami 기본 이미지로 설치하기**

```bash
# bitnami helm add
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo -l bitnami/elasticsearch | head

# elasticsearch 설치하기
helm install elasticsearch oci://registry-1.docker.io/bitnamicharts/elasticsearch -n es

# port-forward
kubectl port-forward --namespace es svc/elasticsearch 9200:9200 &
curl localhost:9200

# kibana 설치하기
helm install kibana oci://registry-1.docker.io/bitnamicharts/kibana \
  --set "elasticsearch.hosts[0]=elasticsearch.es.svc.cluster.local" \
  --set elasticsearch.port=9200 \
-n es

# port-forward
kubectl port-forward --namespace es svc/kibana 5601:5601 &
http://localhost:5601

# 설치 정보 확인
kubectl get all -n es

# helm 삭제
helm delete kibana -n es
```


* **value.yaml 수정하여 설치하기**

```bash
# bitnami helm add
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo -l bitnami/elasticsearch | head

# bitnami value.yaml down
wget https://raw.githubusercontent.com/bitnami/charts/master/bitnami/elasticsearch/values.yaml
wget https://raw.githubusercontent.com/bitnami/charts/master/bitnami/kibana/values.yaml

# es 네임스페이스 생성
kubectl create namespace es

# values.yaml 파일을 이용하여 애플리케이션 배포
helm install elasticsearch -f ./bitnami/es-values.yaml \
  --set sysctlImage.enabled=true -n es --create-namespace bitnami/elasticsearch
  
kubectl port-forward -n es svc/elasticsearch 9200:9200 &

helm install kibana -f ./bitnami/kibana-values.yaml \
  --set sysctlImage.enabled=true -n es bitnami/kibana \
  --set "elasticsearch.hosts[0]=elasticsearch.es.svc.cluster.local" \
  --set elasticsearch.port=9200 \
-n es

kubectl port-forward -n es svc/kibana 5601:5601 &
http://localhost:5601

```

