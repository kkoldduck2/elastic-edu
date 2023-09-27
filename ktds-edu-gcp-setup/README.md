# ktds-edu-gcp-setup
ktds 교육을 위한 GCP Setup (k8s, istio, kafka, redis)



#  1. K3S Setup ( [링크](./1.K3S-Setup.md) )

GCP에 Instance Group 을 생성하고 k3s 를 셋팅후 Load Balace, k9s 설정한다.

다수의 사용자들의 원할한 접근을 위해 GKE 대신 수작업으로 Kubernetes 를 설정한다.

* GCP 에 VM 생성
* K3S Setup
  * K3s 구성
  * K9s Setup
  * Bastion Server Setup
    * kubectl / kubeconfig / heml /podman
* GCP Load Balance Setup
  * K3S ingress(http)
  * kafka nodeport
* Helm Install



#  2. Istio Setup ([링크](./2.Istio-Setup.md))

* Helm 을 이용하여 Istio Setup
* Sample App Sidecar Inject
* 모니터링 tool(prometheus, grafana, kiali, jaeger) Setup



#  3. Kafka Setup ([링크](./3.Kafka-Setup.md))

Strimzi Kafka 를 Setup 한다.

* Strimzi Cluster Operator Setup
* Kafka Cluster Setup
* Accessing Kafka
  * internal / external
  * kafkacat
  * python
* Monitoring(Kafka Exporter, Prometheus, Grafana) Setup



#  4. Redis Setup ([링크](./4.Redis-Setup.md))

Redis 및 Redis Cluster 를 Setup 한다.

* P3X Redis UI tool 설치

* ACL 확인

* Java Sample





#  9. 사용자 실습 환경 설정 ([링크](./9.User-Setup.md))

사용자들의 실습환경에 맞도록 Namespace , Bastion Server 접속정보를 셋팅한다.








