# WSL2에서 k3s 환경 구성하기



기본적으로 WSL2, Docker Desktop, Ubuntu가 설치된 환경에서 설치를 진행한다.

설치가 되어있지 않다면 <u>[PC Setting](./01-PC-setting.md)</u> 을 우선 수행한다.



## 1) 교육 교재 Download

해당 Git에서는 교육에 필요한 정보들을 제공한다.

```bash
# 교육 정보 clone
git clone https://github.com/hiviv/elastic-edu.git
```



## 2) K3S Single Mode 구성

* k3s - Lightweight Kubernetes
  * 경량의 쿠버네티스로 kubernetes의 절반정도 되는 용량으로 패키지 제공
  * [K3s - Lightweight Kubernetes | K3s](https://docs.k3s.io/kr/)



```bash
$ sudo passwd
New password:
Retype new password:
passwd: password updated successfully

# root 권한으로 수행
$ su

# root 권한이 아닌 user도 실행할 수 있도록 --write-kubeconfig-mode 644 옵션을 추가
$ curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644

# 확인
$ kubectl version
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.6+k3s1", GitCommit:"bd04941a294793ec92e8703d5e5da14107902e88", GitTreeState:"clean", BuildDate:"2023-09-20T23:05:58Z", GoVersion:"go1.20.8", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v5.0.1
Server Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.6+k3s1", GitCommit:"bd04941a294793ec92e8703d5e5da14107902e88", GitTreeState:"clean", BuildDate:"2023-09-20T23:05:58Z", GoVersion:"go1.20.8", Compiler:"gc", Platform:"linux/amd64"}

# 확인
$ kubectl get nodes -o wide
NAME   STATUS   ROLES                  AGE     VERSION        INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION                      CONTAINER-RUNTIME
ion    Ready    control-plane,master   3h21m   v1.27.6+k3s1   172.28.245.240   <none>        Ubuntu 22.04.2 LTS   5.15.90.1-microsoft-standard-WSL2   containerd://1.7.6-k3s1.27

```



## 3) kubeconfig 설정

일반 User가 직접 kubctl 명령 실행을 위해서는 kube config 정보(~/.kube/config) 가 필요하다.

k3s 를 설치하면 /etc/rancher/k3s/k3s.yaml 에 정보가 존재하므로 이를 복사한다. 또한 모든 사용자가 읽을 수 있도록 권한을 부여 한다.

```sh
## root 로 실행
$ su

$ ll /etc/rancher/k3s/k3s.yaml
-rw-r--r-- 1 root root 2961 Sep 27 09:50 /etc/rancher/k3s/k3s.yaml

# 모든 사용자에게 읽기권한 부여 (k3s 설치 시 --write-kubeconfig-mode 644 옵션으로 설치한 경우 이미 권한이 부여되어있음)
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



## 4) helm install

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
-rwxr-xr-x 1 1001 docker 50597888 May 11 01:35 linux-amd64/helm*

# move
$ mv linux-amd64/helm /usr/local/bin/helm

# 확인
$ ll /usr/local/bin/helm*
-rwxr-xr-x 1 1001 docker 50597888 May 11 01:35 /usr/local/bin/helm*

# 일반유저로 복귀
$ exit

# 확인
$ helm version
version.BuildInfo{Version:"v3.12.0", GitCommit:"c9f554d75773799f72ceef38c51210f1842a1dea", GitTreeState:"clean", GoVersion:"go1.20.3"}

# 혹시 아래 메시지가 뜨면 권한조정한다.
# WARNING: Kubernetes configuration file is group-readable. This is insecure.
$ chmod 600 ~/.kube/config-multi

```



## 5) alias 정의

```sh
# user 권한으로 alias 등록

$ vi ~/.bashrc
alias k='kubectl'
alias kk='kubectl -n kube-system'
alias kpes='kubectl get pods -n es'
alias kgs='kubectl get service'
alias kgd='kubectl get deployment'
alias kgds='kubectl get daemonset'
alias kall='kubectl get all'

## alias 를 적용하려면 source 명령 수행
source ~/.bashrc

```



## 6) K9S Setup

kubernetes Cluster를 관리하기 위한 kubernetes cli tool 을 설치해 보자.

* [K9s - Manage Your Kubernetes Clusters In Style (k9scli.io)](https://k9scli.io/)

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




