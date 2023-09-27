# Single mode로 Elastic Stack 설치하기



## 1) local 연결을 위한 SSH Key 생성

### (1) SSH key 생성

#### ssh keygen

```sh
$ mkdir -p ~/.ssh/gcp-setup/${username}
$ cd ~/.ssh/gcp-setup/${username}

# key 는 아래 경로로 만들자.
# /home/${username}/.ssh/gcp-setup/${username}

$ ssh-keygen -t rsa -b 4096 -C ${username}

Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): /home/${username}/.ssh/gcp-setup/${username}
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/${username}/.ssh/gcp-setup/${username}
Your public key has been saved in /home/${username}/.ssh/gcp-setup/${username}.pub
The key fingerprint is:
SHA256:RUWm8TG2Wb5sAnv8XzSa3Ig2GJ/2jXO7BygG2tPwYk0 ktdseduuser
The key's randomart image is:
+---[RSA 4096]----+
|          ooB .  |
|         . * B   |
|          + + .  |
|        o.E+ . . |
|       oSO. +.+..|
|      . = X.=+*..|
|       . = O =.o.|
|          o o.ooo|
|             oo+=|
+----[SHA256]-----+


## 비밀번호 : [emtpy]


$ ls -ltr
-rw-r--r-- 1 root root  737 May 13 15:01 ktdseduuser.pub
-rw------- 1 root root 3369 May 13 15:01 ktdseduuser

```



#### ssh 확인

```sh
$ cd ~/song/gcp-setup/sshkey/manager

$ ll
-rw------- 1 root root 3369 May 13 15:01 ktdseduuser
-rw-r--r-- 1 root root  737 May 13 15:01 ktdseduuser.pub


$ cat ktdseduuser
-----BEGIN OPENSSH PRIVATE KEY-----
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
rhLa+cnsPK0AAAALa3Rkc2VkdXVzZXI=
-----END OPENSSH PRIVATE KEY-----


$ cat ktdseduuser.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDVhqICvHjqGjvJ5lyedivtpjomZZU20qtO8+KGTWxDtyyxX5ho14fxNSA0T9zukpbvZ0rH0K38nAcRvaCBtZxbWGk4mxshBxBO1xi4gr/Igi81+7s92ByuJuiZVXformhx/5zfma16aIKOdPr0ffUIpjr6ocZQce9RwoR5pGYn3/QioCJSsDNbDE3leWs9ppC28u2PwmMq9ElnjbtHlL0yxAM5uLPVheWaF5e5x4266IptpJNELphTxeFzC2JIy2Sc5V0O/H728Str+ialszP+Ghv7WuwEc/E7eMT7dGflVy+j/5kcLgxREoxxns8JEDDTZ1QORmjOC9fpOuUTZwdFI6ppNscdIbQZ3x9vWEbPrtaJNAOl3gkYyyLiEnzTt7iImC+5NJ0t3qp2TDcrxc56Ig/drLHP1+Wnir9Ga2z3JAvBoMm88tBnBM3XqsO0r8bee59S85h68XYo2NwURPmYeggS/38jeoYKIwi3GAzJx12c450hSIxL21H6kSCLrZFn1YfPXwfVRtr7t4uo4Dpom9Q2AIR71onyu8REGl1umMPa6vO1MCGJf5yZKUxMiNzjqMWFgD/axOGkE1CtW7/5lmfzJoEkUfBHW4hZ3iPLC8q7ij7Rfvsz4eCbEOp8sK4olEwIohLNL0mqyJyzR1h06Zu3gMS/IOcCjmtw+r3qQQ== ktdseduuser



```





## 2) K3S 구성(Single mode)

### (1) master node

```sh
# root 권한으로 수행
$ su

$ curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644

or

$ curl -sfL https://get.k3s.io | sh -

# 확인
$ kubectl version
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.5+k3s1", GitCommit:"7cefebeaac7dbdd0bfec131ea7a43a45cb125354", GitTreeState:"clean", BuildDate:"2023-05-27T00:05:40Z", GoVersion:"go1.19.9", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v4.5.7
Server Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.5+k3s1", GitCommit:"7cefebeaac7dbdd0bfec131ea7a43a45cb125354", GitTreeState:"clean", BuildDate:"2023-05-27T00:05:40Z", GoVersion:"go1.19.9", Compiler:"gc", Platform:"linux/amd64"}


# IP/ token 확인
$ cat /var/lib/rancher/k3s/server/node-token
K10f74ce1e1f309271e78114c63d51d5936249e3d379faf1c5c7b2269218f2f9220::server:459b5947077d6e612074e998ff769dd8


# 확인
$ kubectl get nodes -o wide
NAME        STATUS   ROLES                  AGE   VERSION        INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME
bastion03   Ready    control-plane,master   47s   v1.26.5+k3s1   10.158.0.29   <none>        Ubuntu 22.04.2 LTS   5.19.0-1022-gcp   containerd://1.7.1-k3s1


```



## 2) 기타 설정

### (1) kubeconfig 설정

일반 User가 직접 kubctl 명령 실행을 위해서는 kube config 정보(~/.kube/config) 가 필요하다.

k3s 를 설치하면 /etc/rancher/k3s/k3s.yaml 에 정보가 존재하므로 이를 복사한다. 또한 모든 사용자가 읽을 수 있도록 권한을 부여 한다.

```sh
## root 로 실행
$ su

$ ll /etc/rancher/k3s/k3s.yaml
-rw------- 1 root root 2961 May 14 03:23 /etc/rancher/k3s/k3s.yaml

# 모든 사용자에게 읽기권한 부여
$ chmod +r /etc/rancher/k3s/k3s.yaml

$ ll /etc/rancher/k3s/k3s.yaml
-rw-r--r-- 1 root root 2961 May 14 03:23 /etc/rancher/k3s/k3s.yaml

# 일반 user 로 전환
$ exit




## 사용자 권한으로 실행

$ mkdir -p ~/.kube

$ cp /etc/rancher/k3s/k3s.yaml ~/.kube/config

$ ll ~/.kube/config
-rw-r--r-- 1 song song 2957 May 14 03:44 /home/song/.kube/config

# 자신만 RW 권한 부여
$ chmod 600 ~/.kube/config

$ ls -ltr ~/.kube/config
-rw------- 1 ktdseduuser ktdseduuser 2957 May 13 14:35 /home/ktdseduuser/.kube/config



## 확인
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.5+k3s1", GitCommit:"7cefebeaac7dbdd0bfec131ea7a43a45cb125354", GitTreeState:"clean", BuildDate:"2023-05-27T00:05:40Z", GoVersion:"go1.19.9", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v4.5.7
Server Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.5+k3s1", GitCommit:"7cefebeaac7dbdd0bfec131ea7a43a45cb125354", GitTreeState:"clean", BuildDate:"2023-05-27T00:05:40Z", GoVersion:"go1.19.9", Compiler:"gc", Platform:"linux/amd64"}

```

root 권한자가 아닌 다른 사용자도 사용하려면 위와 동일하게 수행해야한다.



### (4) alias 정의

```sh
# user 권한으로

$ cat > ~/env
alias k='kubectl'
alias kk='kubectl -n kube-system'
alias kall='kubectl get all'
alias kpes='kubectl get pods -n es'
alias kpk='kubectl get pods -n kibana'
alias kii='kubectl -n istio-ingress'

## alias 를 적용하려면 source 명령 수행
$ source ~/env

# booting 시 자동인식하게 하려면 아래 파일 수정
$ vi ~/.bashrc
...
source ~/env


```



### (5) helm install

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

# 압축해지
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


$ helm -n yjsong ls
NAME    NAMESPACE       REVISION        UPDATED STATUS  CHART   APP VERSION

```



#### [참고] bitnami repo 추가

- 유명한 charts 들이모여있는 bitnami repo 를 추가해 보자.

```sh
# test# add stable repo
$ helm repo add bitnami https://charts.bitnami.com/bitnami

$ helm repo list

$ helm search repo bitnami
# bitnami 가 만든 다양한 오픈소스 샘플을 볼 수 있다.
NAME                                            CHART VERSION   APP VERSION     DESCRIPTION
bitnami/airflow                                 14.1.3          2.6.0           Apache Airflow is a tool to express and execute...
bitnami/apache                                  9.5.3           2.4.57          Apache HTTP Server is an open-source HTTP serve...
bitnami/appsmith                                0.3.2           1.9.19          Appsmith is an open source platform for buildin...
bitnami/argo-cd                                 4.7.2           2.6.7           Argo CD is a continuous delivery tool for Kuber...
bitnami/argo-workflows                          5.2.1           3.4.7           Argo Workflows is meant to orchestrate Kubernet...
bitnami/aspnet-core                             4.1.1           7.0.5           ASP.NET Core is an open-source framework for we...
bitnami/cassandra                               10.2.2          4.1.1           Apache Cassandra is an open source distributed ...
bitnami/consul                                  10.11.2         1.15.2          HashiCorp Consul is a tool for discovering and ...
...

$ kubectl create ns yjsong

# 설치테스트(샘플: nginx)
$ helm -n yjsong install nginx bitnami/nginx

$ kubectl -n yjsong get all
NAME                         READY   STATUS              RESTARTS   AGE
pod/nginx-68c669f78d-wgnp4   0/1     ContainerCreating   0          10s

NAME            TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
service/nginx   LoadBalancer   10.43.197.4   <pending>     80:32754/TCP   10s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   0/1     1            0           10s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-68c669f78d   1         1         0       10s

# 간단하게 nginx 에 관련된 deployment / service / pod 들이 설치되었다.


# 설치 삭제
$ helm -n yjsong delete nginx

$ kubectl -n yjsong  get all
No resources found in yjsong namespace.
```







## 5) K9S Setup

kubernetes Cluster를 관리하기 위한 kubernetes cli tool 을 설치해 보자.

```sh
# root 권한으로

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
-rwxr-xr-x  1 root root 60559360 May 15 13:05 k9s*


# 일반 사용자로 전환
$ exit 

# 실행
$ k9s

```

