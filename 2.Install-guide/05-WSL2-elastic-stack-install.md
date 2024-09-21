# Elastic Stack Install



##  1) Elastic Stack 구성도

본 교육에서는 Elasticsearch, kibana, fleet server, elastic agent 를 설치하여 데이터 수집 및 시각화를 구성한다.

<img src="assets\elastic-architecture.png">



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
* Fleet Server
* Elastic Agent



## 2) Elasticsearch 설치하기

k8s 환경에서는 helm chart를 이용하면 다양한 솔루션들을 손 쉽게 설치할 수 있다. (Elastic 공식 Helm Chart : https://github.com/elastic/helm-charts)

23.10.20 기준 8.5.1 Release 제공중이며, 본 교육에서는 공식 helm chart를 수정하여 v8.10.4, minimal size로 설치한다. (최소 Mem 4GB 이상 필요)

다음 명령어로 elastic helm repo를 가져온 후, elastic-edu git에서 가져온  values.yaml을 이용하여 배포한다. namespace는 es로 설정하였다.

```bash
# elastic 공식 helm repo 추가
$ helm repo add elastic https://helm.elastic.co

# minimal하게 수정한 elasticsearch value 파일로 배포하기
$ cd elastic-edu/2.Install-guide/helm
$ helm install elasticsearch -f ./elasticsearch-values.yaml elastic/elasticsearch -n es --create-namespace &

# 진행상황 확인
$ kubectl get pods --namespace=es -l app=elasticsearch-master -w

# log 확인
$ kubectl logs -f elasticsearch-master-0 -n es
{"@timestamp":"2023-10-24T08:51:23.736Z", "log.level": "INFO",  "current.health":"YELLOW","message":"Cluster health status changed from [RED] to [YELLOW] (reason: [shards started [[.apm-so 
=> 설치 끝

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
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.10.4

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

# 진행상황 확인
$ kubectl get pods --namespace=es -l release=kibana -w

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



#### Fleet Server란?

다양한 데이터 소스에서 데이터 수집을 위해 여러 종류의 Beats를 설치하고 각각의 설정을 하나씩 설정하고 Health check도 별도로 했던 불편사항을 개선하기 위해 데이터 수집기는 Elastic Agent 하나로 통합하고 Fleet Server를 통해 Agent 통합관리하여 Configuration 및 바이너리 자동업데이트를 지원합니다.

 <img src="assets/20231029_215753.png">



1. Agent policy가 생성되면 Kibana의 Fleet UI에서 해당 정책을 Elasticsearch의 Fleet 인덱스에 저장
2. Elastic Agent는 인증을 위해 생성된 token을 사용하여 Fleet Server에 인증 요청
3. Fleet Server는 Fleet 인덱스를 모니터링하고 Elasticsearch에서 새 에이전트 정책을 선택한 다음 해당 정책에 등록된 모든 Elastic 에이전트에 정책 전달
4. Elastic Agent는 정책의 구성 정보를 사용하여 데이터를 수집하고 Elasticsearch로 전송
5. Elastic Agent는 업데이트를 위해 Fleet Server에 체크인하여 연결을 유지
6. 정책이 업데이트되면 Fleet Server는 Elasticsearch에서 업데이트된 정책을 검색하여 연결된 Elastic Agent로 전송
7. Elastic Agent 상태 및 정책 롤아웃에 대해 Fleet과 통신하기 위해 Fleet 서버는 Fleet 인덱스에 업데이트



#### Fleet Server 설치

* Elastic Agent를 관리하기 위한 Fleet Server를 설치 전 Agent를 관리하기 위한 Role을 추가한다.

``` bash
# elastic agent를 관리하기 위한 다양한 role 추가
$ kubectl create -f elastic-agent-role-clusterrole-serviceaccount.yaml
```



* 브라우저에서 kibana에 접속한다.
  * http://localhost:5601 (elastic/new1234!)

 <img src="assets\20231022_223021.png">



* "Explore on my own"을 선택한다.

 <img src="assets\20231022_223312.png">

* 왼쪽 상단 ☰ > Management > Fleet를 선택한다.

 <img src="assets\20231022_230337.png">



* "Add Fleet Server"를 클릭한다.

<img src="assets\20231022_230927.png">

* Advanced 선택
* ① Name
  * fleet-server-policy
* Create policy 선택

 <img src="assets\20231022_231511.png">



* ② Quick start 유지, 
* ③ Name
  * fleet-server
* URL
  *  https://localhost:8220
  * "Add host" 선택

<img src="assets\20231022_231716.png">



* ④ Generate service token 클릭 후 생성된 token 복사

<img src="assets\20231022_232423.png">

* 다시 터미널로 돌아가서 위에서 복사한 Token 값으로 fleet-server.yaml 을 수정한다.

<img src="assets\20231022_235215.png">

```bash
# token 값 수정 후 저장
$ vi fleet-server.yaml

# tocket value로 마우스 커서를 옮긴 뒤 $를 입력하고 a를 입력 후 text를 지운다. 
# 마우스 오른쪽을 클릭하여 복사한 내용을 붙이고 :wq! 를 입력하여 저장한다.

# fleet server 설치 (설치 시 fleet server와 Integration server가 둘 다 설치된다.)
$ kubectl create -f fleet-server.yaml

# fleet server 설치 확인
$ k get service -n es
NAME                            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE
elasticsearch-master-headless   ClusterIP   None           <none>        9200/TCP,9300/TCP   15m
elasticsearch-master            ClusterIP   10.43.98.190   <none>        9200/TCP,9300/TCP   15m
kibana-kibana                   NodePort    10.43.89.61    <none>        5601:32275/TCP      12m
fleet-server                    ClusterIP   10.43.50.185   <none>        8220/TCP            12s

# Integration 설치 확인
$ kpes
NAME                                             READY   STATUS    RESTARTS   AGE
elasticsearch-master-0                           1/1     Running   0          16m
kibana-kibana-78dd9f6f96-47cv7                   1/1     Running   0          13m
integration-server-deployment-5ccfc6c9dc-6fwz5   1/1     Running   0          43s

# port forward
$ kubectl port-forward -n es svc/fleet-server 8220:8220 &
```

* 잠시 기다리면 자동으로 Fleet Server와 Connect 된다.

 <img src="assets\20231023_145002.png">



* 새로고침(F5)을 하여 Fleet 메뉴에서 정상적으로 조회가 되는지 확인한다. 



<img src="assets\20231023_153751.png">



* Elasticsearch, Kibana, Fleet Server까지 기본 설치 완료

 <img src="assets/install-status01.png">



* Setting 메뉴 > Outputs 에서 Fleet server → Elasticsearch 로 데이터 전송이 될 수 있게 설정한다.

 

 <img src="assets\20231023_154337.png">



* Name
  * fleet-es
* Hosts
  * https://elasticsearch-master:9200
* Advanced YAML configuration
  * ssl.verification_mode: none

 <img src="assets\20231023_013026.png">





## 5) Elastic Agent 설치하기





#### 1. Elastic Agent 설치

<img src="assets\20231023_162528.png">



* Add agent

  * ① Name : Agent policy 1 > Create Policy

  * ② Enroll in Fleet > Enroll in Fleet (recommended) > 수정 없음

    

   <img src="assets\20231025_231455.png">

  

  * ③ Install Elastic Agent on your hosts

  > GCP에서는 정상적으로 적용되나, Windows 환경에서는 메모리 부족 등으로 정상적으로 설치되지 않는다.
>
  > 하단 **2. Windows 환경에 Elastic Agent 설치하기**로 진행한다.

  * (Skip) ~~elastic-agent-managed-kubernetes.yml 파일 수정~~

  ```bash
  # 복사한 FLEET_ENROLLMENT_TOKEN 값 수정
  $ vi elastic-agent-managed-kubernetes.yml
  
  # elastic-agent daemonset 생성 (이때 FLEET_ENROLLMENT_TOKEN 값 수정 필요)
$ kubectl apply -f elastic-agent-managed-kubernetes.yml
  ```

  

  #### 2. Windows 환경에 Elastic Agent 설치하기

  * Powershell을 관리자 권한으로 실행
  
     <img src="assets\20231025_232634.png">

  

* ③ Install Elastic Agent on your hosts > Windows를 선택한다.

     <img src="assets\20231023_170026.png">

  

* Windows에 있는 내용을 복사하여 관리자:Powershell에서 실행한다.
  
  ```bash
  $ d:
  $ mkdir temp
  $ cd temp
  $ $ProgressPreference = 'SilentlyContinue'
  Invoke-WebRequest -Uri https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.10.4-windows-x86_64.zip -OutFile elastic-agent-8.10.4-windows-x86_64.zip
  
  $ Expand-Archive .\elastic-agent-8.10.4-windows-x86_64.zip -DestinationPath .
  $ cd elastic-agent-8.10.4-windows-x86_64
    
  # **** 중요!! Token 값 뒤에 --insecure 옵션을 추가한다.
  $ .\elastic-agent.exe install --url=https://localhost:8220 --enrollment-token=UVV5emFZc0JNMmpVM1h5WkR2OFg6b1ItdVZqWFZSVEtOT1dOQXU2c0M4UQ== --insecure
  
  $ Elastic Agent will be installed at C:\Program Files\Elastic\Agent and will run as a service. Do you want to continue? [Y/n]:y
  
  # 만약 --insecure 옵션을 추가 하지 않은 경우 elastic agent를 삭제 후 재 설치 한다.
  $ C:\"Program Files"\Elastic\Agent\elastic-agent.exe uninstall  
  ```
  
    <img src="assets\20231025_233200.png">



* Kibana로 돌아가면 자동으로 Agent 가 추가된 것을 확인할 수 있다.

  <img src="assets\20231023_171352.png">



* 몇 분 기다리면 Agent와 연동이 완료된다.

 <img src="assets\20231025_003851.png">



* Fleet > Settings > Outputs > Add output을 클릭한다.

 <img src="assets\20231025_235313.png">



* Name
  
  * agent-es
  
* Hosts

  * https://localhost:9200

* Advanced YAML configuration

  * ssl.verification_mode: none

* Save and apply settings

 <img src="assets\20231025_235904.png">



* Fleet > Agent policies > fleet-server-policy를 선택한다.

  

 <img src="assets\20231026_000854.png">



* fleet-server-policy > Settings를 선택한다.

 <img src="assets\20231026_001144.png">

* Output for integrations

  * fleet-es

* Output for agent monitoring

  * fleet-es

 <img src="assets\20231026_001454.png">



<img src="assets\20231026_111942.png">

 <img src="assets\20231026_112027.png">



#### 3. Elastic Agent 수집 정보 확인하기



* Fleet > Agent > Agent pilocy 1을 선택한다.

<img src="assets\20231026_112243.png">



* 추가 설정없이 기본적으로 System metric을 수집한다.

<img src="assets\20231026_112535.png">



* Discover에서 수집 정보 확인하기
  <img src="assets\20231026_112839.png">



#### 4. Discover 메뉴 살펴보기

* 자세한 내용은 [Discover 사용 가이드](../3.User-guide/01-Discover.md)를 참고한다.



