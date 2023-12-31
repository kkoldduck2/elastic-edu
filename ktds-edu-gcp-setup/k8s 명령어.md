# k8s 명령어





**3.8.4 namespace 유지하면서 그 안의 모든 pod 삭제** https://greencloud33.tistory.com/21

--all 옵션으로 간편하게 해당 namespace의 모든 pod를 삭제할 수 있다.

```sh
root@master001:~/k8s_in_action/03_pod# kubectl delete po --all
```

우선 전체 List 확인

```sh
kubectl get pods
```

 그 중 Completed 확인

```sh
kubectl get pod --field-selector=status.phase==Succeeded
```

 Completed pod들 삭제

```sh
kubectl delete pod --field-selector=status.phase==Succeeded
```

 에러 상태의 pod 삭제

```sh
kubectl delete pod --field-selector=status.phase==Failed
```

pod는 다음과 같이 중지,제거 할 수 있다.

 

 

**3.8.1 이름으로 삭제**

pod를 삭제하면 pod 안의 모든 container 또한 종료된다.

```csharp
root@master001:~/k8s_in_action/03_pod# kubectl delete pod kubia-gpu
pod "kubia-gpu" deleted
```

 

**3.8.2 Label Selector를 이용한 pod 삭제**

label을 지정하여 원하는 집합의 pod를 한번에 삭제할 수 있다.

```csharp
root@master001:~/k8s_in_action/03_pod# kubectl delete pod -l creation_method=manual
pod "kubia" deleted
pod "kubia-manual-v2" deleted
```

 

**3.8.3 namespace 삭제로 인한 pod 제거**

더 이상 사용하지 않는 namespace를 지우면 그 안에 있는 오브젝트도 함께 삭제된다.

```csharp
root@master001:~/k8s_in_action/03_pod# kubectl delete ns wglee-namespace
namespace "wglee-namespace" deleted
```

 

**3.8.4 namespace 유지하면서 그 안의 모든 pod 삭제**

--all 옵션으로 간편하게 해당 namespace의 모든 pod를 삭제할 수 있다.

```csharp
root@master001:~/k8s_in_action/03_pod# kubectl delete po --all
pod "kubia-manual" deleted
pod "kubia-manual2" deleted
pod "nginx-deployment-7d95987b64-4pk8v" deleted
pod "nginx-deployment-7d95987b64-kxr44" deleted
pod "nginx-deployment-7d95987b64-thfs6" deleted
pod "podtest" deleted
```

단, 해당 명령어를 실행 한 후에도 기존에 replicaset 혹은 deployment로 생성한 리소스는 지워지지 않는다.

이 경우 상위 object를 지워야 그로 인해 생성된 pod도 삭제 가능하다.

```csharp
root@master001:~/k8s_in_action/03_pod# kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7d95987b64-5cmxj   1/1     Running   0          97s
nginx-deployment-7d95987b64-6t5j9   1/1     Running   0          97s
nginx-deployment-7d95987b64-b9k8k   1/1     Running   0          97s

root@master001:~/k8s_in_action/03_pod# kubectl delete deployments nginx-deployment
deployment.apps "nginx-deployment" deleted

root@master001:~/k8s_in_action/03_pod# kubectl get pods
No resources found in default namespace.
```

 

**3.8.5 Namespace에서 모든 리소스 삭제**

모든 리소스(replicaset, pod, deployment 등)을 한번에 삭제할 수 있다. 하지만 secret 등의 리소스는 삭제되지 않는 것을 유의한다.

```csharp
root@master001:~# kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
kubia-79dbg    1/1     Running   0          17s
kubia-gpu      1/1     Running   0          72s
kubia-manual   1/1     Running   0          86s
kubia-mcllt    1/1     Running   0          17s
kubia-zg9kp    1/1     Running   0          17s

root@master001:~# kubectl get rc
NAME    DESIRED   CURRENT   READY   AGE
kubia   3         3         3       24s

root@master001:~# kubectl delete all --all
pod "kubia-79dbg" deleted
pod "kubia-gpu" deleted
pod "kubia-manual" deleted
pod "kubia-mcllt" deleted
pod "kubia-zg9kp" deleted
replicationcontroller "kubia" deleted
service "kubernetes" deleted
```