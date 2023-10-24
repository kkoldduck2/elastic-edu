# Getting started



## 1) GCP 계정 생성

GCP(Google Cloud Platform)는 총 90일 간 $300 무료 크레딧을 제공하며 본 강의는 GCP의 무료 크레딧을 사용하여 진행한다.

* 접속 정보 : [cloud.google.com](https://cloud.google.com)  -> 오른쪽 상단 무료로 시작하기 버튼 클릭

<img src="assets\image-20231019224242526.png">

* 로그인 계정 선택

<img src="assets\image-20231019224434037.png">



* 계정정보 입력

<img src="assets\image-20231019225444624.png">

* 결제 정보 입력 (카드번호만 제대로 입력하면 바로 가입 완료)

<img src="assets\image-20231019225613485.png">

<img src="assets\image-20231019225730300.png">


## 2) VM 인스턴스 생성

#### Master node 1대, Worker node 2대로 구성



##### 1.  Master node용 VM 생성

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



* 설정 화면
<img src="assets\image-20231019233648453.png">

<img src="assets\image-20231019234157016.png">

<img src="assets\image-20231019235328527.png">

<img src="assets\image-20231019234957278.png">

<img src="assets\image-20231019235214995.png">