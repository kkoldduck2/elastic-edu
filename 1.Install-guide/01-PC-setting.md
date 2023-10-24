# 1. 실습 환경 준비(개인PC)


## 1) mobaxterm
PC에서 ssh 접근을 할 수 있는 다양한 터미널(putty 등)이 있지만 본 실습에서는 mobax term 이라는 free 버젼 솔루션을 사용한다.

- download 위치
  
- 링크: https://download.mobatek.net/2202022022680737/MobaXterm_Installer_v22.0.zip
  
- mobaxterm 실행

  <img src="assets\image-20220601194018844.png">


## 2) wsl2

본 강의 에서는 개인 PC 에 wsl2 설치 후 진행예정이므로 PC에  WSL이 설치되어 있는지 확인이 필요하다.


### (1) 확인 하는 방법

command 창에서 wsl 명령으로 설치여부를 확인 할 수 있다.

```sh
> wsl -l -v 
```

<img src="assets\image-20220601193023175.png">


- 만약 version 이 1 이라면 아래와 같이 update 한다.
  - 참고링크
    - https://docs.microsoft.com/en-us/windows/wsl/install
    - https://docs.microsoft.com/ko-kr/windows/wsl/install-manual
  - PowerShell 실행

```sh
> wsl --install

> wsl --set-version Ubuntu 2

# 기본값으로 설정 변경해도 됨
> wsl --set-default-version 2

# 강제 재기동
> wsl -t Ubuntu

```



- linux 가 설정안되어 있다면

```sh
1. Microsoft Store를 열고 즐겨 찾는 Linux 배포를 선택
   - Ubuntu 20.04.1 LTS

2. 배포 페이지에서 "다운로드"를 선택

3. 사용자 계정 및 암호 생성

```





### (2) WSL 실행하는 방법

실행하는 방법은 아래와 같이 다양하다. 본인이 편한 방법을 선택하자.

1. cmd 창에서 바로실행
   - cmd 창에서 `wsl` 명령을 입력하면 바로 default linux 가 실행된다.
   - `wsl -u root` 명령으로 root 로 실행 할 수 있다.

<img src="assets\image-20220601193219422.png" style="margin:0 30px">


2. windows 터미널 으로 실행하는 방법
   - windows 터미널 설치 : https://docs.microsoft.com/ko-KR/windows/terminal/get-started
   
3. mobaxterm 에서 실행
   - session > WSL 실행

<img src="assets\image-20220601193859958.png" style="margin:0 30px">


## 3) docker desktop 확인

Container 환경 실습을 위해 docker desktop 설치가 필요하다

### (1) docker destktop 확인

우측 하단  docker desktop  아이콘에서 우클릭후 아래 그림 처럼 Docker Desktop is running 확인

<img src="assets\image-20220601192354841.png">



### (2) docker destktop install

설치되어 있지 않으면 아래와 같이 설치한다.

- 설치 가이드 위치
  - 링크: https://docs.docker.com/desktop/windows/install/



### (3) docker daemon 확인

docker 가 실행가능 곳에서 아래와 같이 version 을 확인하자.

```sh
$ $ docker version
Client: Docker Engine - Community
 Cloud integration: v1.0.29
 Version:           20.10.22
 API version:       1.41
 Go version:        go1.18.9
 Git commit:        3a2c30b
 Built:             Thu Dec 15 22:28:22 2022
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Desktop
 Engine:
  Version:          20.10.22
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.18.9
  Git commit:       42c8b31
  Built:            Thu Dec 15 22:26:14 2022
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.14
  GitCommit:        9ba4b250366a5ddde94bb7c9d1def331423aa323
 runc:
  Version:          1.1.4
  GitCommit:        v1.1.4-0-g5fd4c4d
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
  
```

Server version 을 확인할 수 있다면 정상 설치되었다고 볼 수 있다.


### (4) WSL2에서 도커 데스크탑 실행 설정

도커 데스크탑을 설치하고 설정 페이지의 **General** 탭에서 **Use the WSL2 based engine** 옵션을 체크해준다.

<img src="assets\cc2fa29ced0170be569fa2babb3f37ce853a4a6edaa393ae7d7e6cf0e734809e.m.png">

Resource -> WSL Integration 페이지로 이동해서 설정을 확인한다. 자신이 사용중인 WSL2 배포판이 맞는지 확인한다.

<img src="assets\2e6f6b874322977fd2a606fac1628628a42e0e6161aaecae4e0ca5891dda008d.m.png">

도커 데스크탑을 설치하고 정상적으로 설정되어있다면, 바로 WSL2 우분투 터미널에서 도커 명령어를 사용할 수 있다.

<img src="assets\d0f6a634419019be2cf954e7258932a9ea28afc6a058059f54e659104003fddf.m.png">