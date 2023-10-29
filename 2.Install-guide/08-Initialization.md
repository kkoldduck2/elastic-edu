# 초기화 하기



## 1) k3s 초기화

```bash
$ /usr/local/bin/k3s-uninstall.sh
$ sudo rm -rf /opt
$ rm -rf .kube
```



## 2) Namespace 및 권한 삭제

```bash
# es namespace를 삭제한다.
$ kubectl delete namespace es

# 권한 정보를 삭제한다.
$ kubectl delete serviceaccount elastic-agent -n es
$ kubectl delete RoleBinding elastic-agent elastic-agent-kubeadm-config -n es
$ kubectl delete ClusterRoleBinding elastic-agent -n es
$ kubectl delete ClusterRole elastic-agent
```



## 3) Elastic Agent 삭제

 ```bash
# powershell 을 관리자 권한으로 실행
$ C:\"Program Files"\Elastic\Agent\elastic-agent.exe uninstall
$ del C:\"Program Files"\Elastic
 ```

