



# Elastic Stack Install



##  1) Elastic Stack 구성도

본 교육에서는 Elasticsearch, kibana, fleet server, elastic agent 를 설치하여 데이터 수집 및 시각화를 구성한다.

<img src="D:\GitHub\hiviv\elastic-edu\1.Install-guide\assets\fleet-server-cloud-deployment.png" align="left">

| name                                   | desc                                                         |
| :------------------------------------- | :----------------------------------------------------------- |
| Elasticsearch (Store, Search, Analyse) | 검색 및 분석 엔진으로 수립한 데이터를 인덱스에 저장하고 검색이 가능하도록 제공한다. |
| Kibana (Explore, Visualize, Engage)    | Elastic stack을 쉽게 사용하기 위한 툴로 데이터 분석, 시각화, Elastic Stack에 대한 모니터링 및 설정 관리를 위해 사용한다. |
| Integration (Connect, Collect, Alert)  | 기본 Beats 제품군 및 Logstash를 포함하여 다양한 솔루션과의 Integration 기능을 제공한다. |
| Elastic Agent                          | 로그 및 메트릭, 기타 데이터 유형에 대한 모니터링을 위해 각 host 서버에 설치하여 데이터를 수집한다. |
| Fleet Server                           | Fleet Server는 배포된 Elastic Agent와 통신하여 각 Agent의 policy를 관리한다. |

설치 순서는 다음과 같다.

* Elasticsearch
* Kibana
* Elastic Agent
* Fleet Server



## 2) Elasticsearch 설치하기

k8s 환경에서는 helm chart를 이용하면 다양한 솔루션들을 손 쉽게 설치할 수 있다. (Elastic 공식 Helm Chart : https://github.com/elastic/helm-charts)

23.10.20 기준 8.5.1 Release 제공중이며, 본 교육에서는 공식 helm chart를 수정하여 v8.9.2, minimal size로 설치한다. (최소 Mem 4GB 이상 필요)

다음 명령어로 elastic helm repo를 가져온 후, elastic-edu git에서 가져온  values.yaml을 이용하여 배포한다. namespace는 es로 설정하였다.

```bash
# elastic 공식 helm repo 추가
$ helm repo add elastic https://helm.elastic.co

# minimal하게 수정한 elasticsearch value 파일로 배포하기
$ cd elastic-edu/helm/
$ helm install elasticsearch -f ./elasticsearch-values.yaml elastic/elasticsearch -n es --create-namespace &

# log 확인
$ kubectl logs -f elasticsearch-master-0 -n es
{"@timestamp":"2023-09-23T10:08:50.087Z", "log.level": "WARN", "message":"This node is a fully-formed single-node cluster with cluster UUID [I5WGB0BBS3KARCnW4666qw], but it is configured as if to discover other nodes and form a multi-node cluster via the [discovery.seed_hosts=[elasticsearch-master-headless]] setting. Fully-formed clusters do not attempt to discover other nodes, and nodes with different cluster UUIDs cannot belong to the same cluster. The cluster UUID persists across restarts and can only be changed by deleting the contents of the node's data path(s). Remove the discovery configuration to suppress this message.", "ecs.version": "1.2.0","service.name":"ES_ECS","event.dataset":"elasticsearch.server","process.thread.name":"elasticsearch[elasticsearch-master-0][scheduler][T#1]","log.logger":"org.elasticsearch.cluster.coordination.Coordinator","elasticsearch.cluster.uuid":"I5WGB0BBS3KARCnW4666qw","elasticsearch.node.id":"F6nUdmCnTDeHjpcIPukrdQ","elasticsearch.node.name":"elasticsearch-master-0","elasticsearch.cluster.name":"elasticsearch"} => 설치 끝

# port forward
$ kubectl port-forward -n es elasticsearch-master-0 9200:9200 &

# curl로 접속 확인
$ curl https://localhost:9200 -k -u 'elastic:new1234!'

```

### [참고] Pod 상태가 ImagePullBackOff 인 경우

``` bash
# 현상
$ kubectl get pods --namespace=es
NAME                     READY   STATUS                  RESTARTS   AGE
elasticsearch-master-0   0/1     Init:ImagePullBackOff   0          15m

# 다음 명령으로 에러 원인 확인
$ kubectl describe pod elasticsearch-master-0 -n es

Events:
  Type     Reason            Age                From               Message
  ----     ------            ----               ----               -------
  Warning  FailedScheduling  22m                default-scheduler  0/1 nodes are available: 1 pod has unbound immediate PersistentVolumeClaims. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.
  Normal   Scheduled         22m                default-scheduler  Successfully assigned es/elasticsearch-master-0 to docker-desktop
  Normal   Pulling           14m (x4 over 22m)  kubelet            Pulling image "docker.elastic.co/elasticsearch/elasticsearch:8.9.2"
  Warning  Failed            12m (x4 over 20m)  kubelet            Failed to pull image "docker.elastic.co/elasticsearch/elasticsearch:8.9.2": rpc error: code = Unknown desc = context deadline exceeded
  Warning  Failed            12m (x4 over 20m)  kubelet            Error: ErrImagePull
  Warning  Failed            12m (x7 over 20m)  kubelet            Error: ImagePullBackOff
  Normal   BackOff           2m (x33 over 20m)  kubelet            Back-off pulling image "docker.elastic.co/elasticsearch/elasticsearch:8.9.2"

# docker pull 명령으로 직접 이미지를 다운로드 한다.
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.9.2

# pod가 Ready 상태가 됐는지 다시 확인한다.
$ kubectl get pods --namespace=es
NAME                     READY   STATUS    RESTARTS   AGE
elasticsearch-master-0   1/1     Running   0          38m

# 로그 확인
$ kubectl logs -f elasticsearch-master-0 -n es

```





## 3) Kibana 설치하기

elasticsearch에 이어 kibana도 수정한 values.yaml을 이용하여 설치한다. namespace는 동일하게 es로 설정하였다.

```bash
# kibana value 파일로 배포하기
$ helm install kibana elastic/kibana -f ./kibana-values.yaml -n es &

# log 확인
$ kubectl logs -f svc/kibana-kibana -n es
[2023-09-23T10:06:10.897+00:00][INFO ][plugins.synthetics] Installed synthetics index templates => 설치 끝

# 상세 정보 확인
$ kubectl get pods -n es
NAME                            READY   STATUS    RESTARTS   AGE
elasticsearch-master-0          1/1     Running   0          3h41m
kibana-kibana-695f748b6-rpxwl   1/1     Running   0          3h36m

# 위에서 확인한 kibana pod 이름을 넣어 상세정보 확인
$ kubectl describe pod/kibana-kibana-695f748b6-rpxwl -n es

# port forward
$ kubectl port-forward -n es svc/kibana-kibana 5601:5601 &

$ curl -v http://localhost:5601


```



## 4) Fleet Server 설치하기

Elastic Agent를 관리하기 위한 Fleet Server를 설치한다.

``` bash
$ kubectl create -f elastic-agent-role-clusterrole-serviceaccount.yaml
```



* 브라우저에서 kibana에 접속한다.
  * http://localhost:5601 (elastic/new1234!)

<img src="assets\20231022_223021.png" align="left">



* "Explore on my own"을 선택한다.

<img src="assets\20231022_223312.png" align="left">

* 왼쪽 상단 ☰ > Management > Fleet를 선택한다.



<img src="assets\20231022_230337.png" align="left">



* "Add Fleet Server"를 클릭한다.

<img src="assets\20231022_230927.png" align="left">

* Advanced 선택
* ① Name
  * fleet-server-policy
* Create policy 선택

<img src="assets\20231022_231511.png" align="left">



* ② Quick start 유지, 
* ③ Name
  * fleet-server
* URL
  *  https://localhost:8220
  * "Add host" 선택

<img src="assets\20231022_231716.png" align="left">



* ④ Generate service token 클릭 후 생성된 token 복사

<img src="assets\20231022_232423.png" align="left">

* fleet-server.yaml 파일에 복사한 token 값 수정

<img src="assets\20231022_235215.png" align="left">

```bash
# token 값 수정 후 저장
$ vi fleet-server.yaml

# fleet server 설치
$ kubectl create -f fleet-server.yaml
```

* 잠시 기다리면 자동으로 Fleet Server와 Connect 된다.

<img src="assets\20231023_145002.png" align="left">



* 새로고침을 하여 Fleet 메뉴에서 정상적으로 조회가 되는지 확인한다. 



<img src="assets\20231023_153751.png" align="left">



* Setting 메뉴 > Outputs 를 수정한다.

<img src="assets\20231023_154337.png" align="left">



* Hosts
  * https://elasticsearch-master:9200

* Advanced YAML configuration
  * ssl.verification_mode: none

<img src="assets\20231023_013026.png" align="left">





## 5) Elastic Agent 설치하기



#### 1. kube-state-metrics 설치

kubernetes 메트릭 수집을 위한 kube-state-metrics를 설치한다.

```bash
$ git clone https://github.com/kubernetes/kube-state-metrics.git
$ kubectl apply -k kube-state-metrics
```



#### 2. Elastic Agent 설치

<img src="assets\20231023_162528.png" align="left">



*  Add agent

  * default 입력 후 Create policy 클릭

    <img src="assets\20231023_163119.png" align="left">

    

  * Enroll in Fleet 선택

    <img src="assets\20231023_163305.png" align="left">

    

  * Kubernetes 선택 > FLEET_ENROLLMENT_TOKEN 값 복사

    <img src="assets\20231023_170026.png" align="left">

  * elastic-agent-managed-kubernetes.yml 파일 수정

    ```bash
    # 복사한 FLEET_ENROLLMENT_TOKEN 값 수정
    $ vi elastic-agent-managed-kubernetes.yml
    
    # elastic-agent 설치를 위한 namespace 생성
    $ kubectl create namespace es-ds
    
    # elastic-agent daemonset 생성
    $ kubectl apply -f elastic-agent-managed-kubernetes.yml -n es-ds
    ```

* Kibana로 돌아가면 자동으로 Agent 가 추가된 것을 확인할 수 있다.

  <img src="assets\20231023_171352.png" align="left">







##### 수동으로 Elastic agent 설치하는 방법 (미 사용)

데이터 수집을 위해서는 수집을 원하는 host에 elastic agent를 설치한다.

``` bash
# elastic agent 다운로드
$ curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.9.2-linux-x86_64.tar.gz
# 압축 해제
$ tar xzvf elastic-agent-8.9.2-linux-x86_64.tar.gz
$ cd elastic-agent-8.9.2-linux-x86_64
$ sudo ./elastic-agent install \
  --fleet-server-es=http://localhost:9200 \
  --fleet-server-service-token=AAEAAWVsYXN0aWMvZmxlZXQtc2VydmVyL3Rva2VuLTE2OTc5ODcxNzAwOTA6bkFFNnZVczZSOVNpcDZuZ1dxUUU1UQ \
  --fleet-server-policy=76e1a640-70e7-11ee-91f5-835a208b6035 \
  --fleet-server-port=8220
[sudo] password for jay:
Elastic Agent will be installed at /opt/Elastic/Agent and will run as a service. Do you want to continue? [Y/n]:y


```





