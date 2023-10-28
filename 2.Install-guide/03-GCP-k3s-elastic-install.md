# GCP에서 Single Node로 Elastic 설치하기



## 1) 교육 교재 Download

해당 Git에서는 교육에 필요한 정보들을 제공한다.

```bash
# 교육 정보 clone
git clone https://github.com/hiviv/elastic-edu.git
```



## 2) GPC VM 생성하기

> elasticsearch, kibana 각각 2GB 정도 메모리가 필요하므로 여유롭게 8GB 메모리로 선택하여 VM을 생성한다.

* 시작하기 > vm 만들기 또는 ≡ > Compute Engine > VM 인스턴스 클릭
* 새 VM 인스턴스 생성
  * 리전 선택 : '23.10 기준 us-central1 (아이오와), us-west1 (오리건)이 가장 저렴
  * 실습에서는 많은 데이터를 사용하지 않으므로 어떤 리전을 선택하던 무관.
* 용도 : master node
* 머신유형 : e2-standard-2(vCPU 2개, 8GB 메모리)
* 부팅디스크

  * OS : Ubuntu 22.04 LTS 
    * x86/64, amd64 jammy image built on 2023-04-29, supports Shielded VM features
  * disk : 100GB

* 방화벽 : HTTP 트래픽 허용, HTTPS 트래픽 허용 체크



* 설정 화면
  <img src="assets\image-20231019233648453.png">

  <img src="assets\image-20231019234157016.png">

  <img src="assets\image-20231019235328527.png">

  <img src="assets\image-20231019234957278.png">

  <img src="assets\image-20231019235214995.png">





## 3) 방화벽 규칙 만들기

반드시 expose 한 port 로 방화벽을 열어줘야 한다.

* 메뉴 : VPC네트워크 > 방화벽 > VPC 방화벽 규칙

* 방화벽 규칙 만들기

  * 규칙명 : allow-es

  * 로그 : 사용안함

  * 우선순위 : 1000

  * 트래픽방향 : 수신

  * 일치시작업 : 허용

  * 대상 : 네트워크의 모든 인스턴스

  * 소스필터 : IPv4범위

  * 소스 IPv4범위

    * 0.0.0.0/0     <-- 모두 열자.

  * 대상필터 : IPv4범위

  * 대상 IPv4범위 : 0.0.0.0/0    <-- 모든 소스 를 다 허용

  * port

    * tcp : 32275 <-- Node expose port 명시

      

## 4) K3S Single Mode 구성



```bash
# 비밀번호 설정
$ sudo passwd

# root 권한으로 수행
$ su

$ curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644
or
$ curl -sfL https://get.k3s.io | sh -

# 확인
$ kubectl version
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.6+k3s1", GitCommit:"bd04941a294793ec92e8703d5e5da14107902e88", GitTreeState:"clean", BuildDate:"2023-09-20T23:05:58Z", GoVersion:"go1.20.8", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v5.0.1
Server Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.6+k3s1", GitCommit:"bd04941a294793ec92e8703d5e5da14107902e88", GitTreeState:"clean", BuildDate:"2023-09-20T23:05:58Z", GoVersion:"go1.20.8", Compiler:"gc", Platform:"linux/amd64"}

# 확인
$ kubectl get nodes -o wide
NAME             STATUS   ROLES                  AGE    VERSION        INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
elastic-single   Ready    control-plane,master   115s   v1.27.6+k3s1   10.178.0.2    <none>        Ubuntu 22.04.3 LTS   6.2.0-1014-gcp   containerd://1.7.6-k3s1.27
```



## 5) kubeconfig 설정

### 

일반 User가 직접 kubctl 명령 실행을 위해서는 kube config 정보(~/.kube/config) 가 필요하다.

k3s 를 설치하면 /etc/rancher/k3s/k3s.yaml 에 정보가 존재하므로 이를 복사한다. 또한 모든 사용자가 읽을 수 있도록 권한을 부여 한다.

```sh
## root 로 실행
$ su

$ ll /etc/rancher/k3s/k3s.yaml
-rw-r--r-- 1 root root 2961 Sep 27 09:50 /etc/rancher/k3s/k3s.yaml

# 모든 사용자에게 읽기권한 부여
$ chmod +r /etc/rancher/k3s/k3s.yaml

$ ll /etc/rancher/k3s/k3s.yaml
-rw-r--r-- 1 root root 2961 Sep 27 09:50 /etc/rancher/k3s/k3s.yaml

# 일반 user 로 전환
$ exit

## 사용자 권한으로 실행
$ mkdir -p ~/.kube

$ cp /etc/rancher/k3s/k3s.yaml ~/.kube/config

$ ll ~/.kube/config
-rw-r--r-- 1 jay jay 2961 Sep 27 09:53 /home/jay/.kube/config

# 자신만 RW 권한 부여
$ chmod 600 ~/.kube/config

$ ls -ltr ~/.kube/config
-rw------- 1 jay jay 2961 Sep 27 09:53 /home/jay/.kube/config

## 확인
$ kubectl version
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.6+k3s1", GitCommit:"bd04941a294793ec92e8703d5e5da14107902e88", GitTreeState:"clean", BuildDate:"2023-09-20T23:05:58Z", GoVersion:"go1.20.8", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v5.0.1
Server Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.6+k3s1", GitCommit:"bd04941a294793ec92e8703d5e5da14107902e88", GitTreeState:"clean", BuildDate:"2023-09-20T23:05:58Z", GoVersion:"go1.20.8", Compiler:"gc", Platform:"linux/amd64"}

```

root 권한자가 아닌 다른 사용자도 사용하려면 위와 동일하게 수행해야한다.



## 6) helm install

#### helm client download

```sh
# 개인 PC WSL
# root 권한으로 수행
$ su


## 임시 디렉토리를 하나 만들자.
$ mkdir -p ~/temp/helm/
  cd ~/temp/helm/

# 다운로드
$ wget https://get.helm.sh/helm-v3.12.0-linux-amd64.tar.gz

# 압축풀기
$ tar -zxvf helm-v3.12.0-linux-amd64.tar.gz

# 확인
$ ll linux-amd64/helm
-rwxr-xr-x 1 goworkj 123 50597888 May 10 16:35 linux-amd64/helm*

# move
$ mv linux-amd64/helm /usr/local/bin/helm

# 확인
$ ll /usr/local/bin/helm*
-rwxr-xr-x 1 goworkj 123 50597888 May 10 16:35 /usr/local/bin/helm*


# 일반유저로 복귀
$ exit

# 확인
$ helm version
version.BuildInfo{Version:"v3.12.0", GitCommit:"c9f554d75773799f72ceef38c51210f1842a1dea", GitTreeState:"clean", GoVersion:"go1.20.3"}

# 혹시 아래 메시지가 뜨면 권한조정한다.
# WARNING: Kubernetes configuration file is group-readable. This is insecure.
$ chmod 600 ~/.kube/config-multi

```



## 7) alias 정의

```sh
# user 권한으로 alias 등록

$ vi ~/.bashrc
alias k='kubectl'
alias kk='kubectl -n kube-system'
alias kpes='kubectl get pods -n es'
alias kd='kubectl delete'
alias kall='kubectl get all'

## alias 를 적용하려면 source 명령 수행
source ~/.bashrc

```



## 8) K9S Setup

kubernetes Cluster를 관리하기 위한 kubernetes cli tool 을 설치해 보자.

```sh
# root 권한으로

$ su
Password:

$ mkdir ~/temp/k9s
  cd  ~/temp/k9s

$ wget https://github.com/derailed/k9s/releases/download/v0.27.4/k9s_Linux_amd64.tar.gz
$ tar -xzvf k9s_Linux_amd64.tar.gz

$ ll
-rw-r--r-- 1  501 staff    10174 Mar 22  2021 LICENSE
-rw-r--r-- 1  501 staff    35702 May  7 16:54 README.md
-rwxr-xr-x 1  501 staff 60559360 May  7 17:01 k9s*
-rw-r--r-- 1 root root  18660178 May  7 17:03 k9s_Linux_amd64.tar.gz

$ cp ./k9s /usr/local/bin/

$ ll /usr/local/bin/
-rwxr-xr-x  1 root    root 60559360 Sep 27 10:06 k9s*


# 일반 사용자로 전환
$ exit 

# 실행
$ k9s

```

#### k9s 명령어

네임스페이스 변경

```ruby
:namespace <namespcae_name>
```

다양한 오브젝트 모니터링

```ruby
:logs               # 모든 컨테이너 로그 보기
:services           # Service 모니터링
:deployments        # Deployment 모니터링
:statefulsets       # StatefulSet 모니터링
:daemonsets         # DaemonSet 모니터링
:configmaps         # ConfigMap 모니터링
:secrets            # Secret 모니터링
:pods               # Pod 모니터링
:no                # 노드 모니터링
```

터미널에서 컨테이너 쉘 접속

```ruby
shift + s    # 선택한 Pod에 대한 쉘 접속
:q : K9s 종료
:version : K9s 버전 정보 출력
:cluster-info : 클러스터 정보 출력
:nodes : 노드 정보 출력
:ns : 현재 네임스페이스 출력
:namespace <namespace> : 네임스페이스 변경
/ : 검색 모드로 변경
? : 정규식 검색 모드로 변경
ctrl+l : 화면 지우기
ctrl+c : 선택 취소
enter : 선택한 오브젝트 정보 출력
ctrl+enter : 선택한 오브젝트 YAML 출력
shift+enter : 선택한 오브젝트 로그 출력
shift+a : 새로운 오브젝트 생성
shift+e : 선택한 오브젝트 수정
shift+d : 선택한 오브젝트 삭제
shift+s : 선택한 Pod에 대한 쉘 접속
: : 명령 모드로 변경
:logs : 모든 컨테이너 로그 출력
:services : Service 정보 출력
:deployments : Deployment 정보 출력
:statefulsets : StatefulSet 정보 출력
:daemonsets : DaemonSet 정보 출력
:configmaps : ConfigMap 정보 출력
:secrets : Secret 정보 출력
:pods : Pod 정보 출력
:no : 노드 정보 출력
:portforwards <pod> : Pod 포트 포워딩
```



## 9) Elasticsearch 설치하기

helm chart를 이용하면 손 쉽게 설치할 수 있다. (Elastic 공식 Helm Chart : https://github.com/elastic/helm-charts)

23.09.27 기준 8.5.1 Release 제공중이며, 본 교육에서는 공식 helm chart를 수정하여 v8.9.2, minimal size로 설치한다. (최소 Mem 4GB 이상 필요)

다음 명령어로 elastic helm repo를 가져온 후, elastic-edu git에서 가져온  values.yaml을 이용하여 배포한다. namespace는 es로 설정하였다.

```bash
# elastic 공식 helm repo 추가
$ helm repo add elastic https://helm.elastic.co

# minimal하게 수정한 elasticsearch value 파일로 배포하기
$ cd elastic-edu/helm/
$ helm install elasticsearch -f ./elasticsearch-values.yaml elastic/elasticsearch -n es --create-namespace &

# log 확인
$ k logs -f elasticsearch-master-0 -n es

# port forward
$ kubectl port-forward -n es elasticsearch-master-0 9200:9200 &

# curl로 접속 확인
$ curl https://localhost:9200 -k -u 'elastic:new1234!'

```



## 10) Kibana 설치하기

elasticsearch에 이어 kibana도 수정한 values.yaml을 이용하여 설치한다. namespace는 동일하게 es로 설정하였다.

```bash
# kibana value 파일로 배포하기
$ helm install kibana elastic/kibana -f ./kibana-values.yaml -n es &

# log 확인
$ k logs -f svc/kibana-kibana -n es
$ k describe pod/kibana-kibana-99c648b97-pcb5m -n es

# port forward
$ kubectl port-forward -n es svc/kibana-kibana 5601:5601 &

$ curl -v http://localhost:5601

```



## 11) 브라우저에서 Kibana 접속하기

GCP의 external ip로 접속한다.

http://${GCP 외부 IP}:32275