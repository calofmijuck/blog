---
share: true
toc: true
categories: [Development, Kubernetes]
path: "_posts/development/kubernetes"
tags: [kubernetes, sre, devops]
title: "10. StatefulSets: Deploying Replicated Stateful Applications"
date: "2021-05-17"
github_title: "2021-05-17-10-statefulsets"
image:
  path: /assets/img/posts/development/kubernetes/k8s-10.jpeg
attachment:
  folder: assets/img/posts/development/kubernetes
---

![k8s-10.jpeg](/assets/img/posts/development/kubernetes/k8s-10.jpeg) _A stateful pod may be rescheduled to a different node, but it retains the name, hostname, and storage. (출처: https://livebook.manning.com/book/kubernetes-in-action/chapter-10)_

### 주요 내용

- StatefulSet 에 대한 이해
- Stateful application 배포하기
- DNS SRV record 를 이용한 peer discovery

## 10.1 Replicating stateful pods

---

도입 질문: 각 pod replica 가 하나씩 PV 를 갖게 할 수는 없을까?

ReplicaSet 의 pod template 에서 PVC (PersistentVolumeClaim) 를 사용하게 되면, 모든 replica 가 하나의 PV (PersistentVolume) 을 참조하게 된다.

그래서 *하나의* ReplicaSet 으로는 distributed data store 를 만들 수 없다. 지금까지 살펴본 API object 만으로는 조금 복잡하다.

### 10.1.1 Running multiple replicas with separate storage for each

Pod 의 replica 들을 만들고 싶은데 각각이 자신만의 storage volume 을 갖게 하려면 어떻게 해야하는가?

#### Pod 직접 만들어서 띄우기

Kubernetes 의 철학에 어긋난다.

#### One ReplicaSet per pod instance

각 ReplicaSet 의 replica count 를 1로 설정하고, 각 ReplicaSet 이 PVC 를 사용하도록 하면 여러 개의 pod 가 생성되며 각각이 자신만의 volume 을 갖도록 구성할 수 있다.

직접 pod 를 만들어 띄우는 것과 비교하면 automatic scheduling 이 지원되므로 좀 더 낫지만, scaling 을 수동으로 해야한다는 점에서 불편하다.

#### Multiple directories in the same volume

또 하나의 방법으로는, pod 들이 하나의 PV 를 사용하되, PV 내의 서로 다른 폴더를 사용하도록 하는 방법이 있다.

한편 이렇게 하면 ReplicaSet 이 알아서 pod 를 생성하므로, 각 pod 에게 PV 내의 어떤 폴더를 저장 공간으로 사용하라고 임의로 지정할 수 없다. (물론 앱 내부적으로 폴더를 선택하는 로직을 추가 구현할 수 있긴 하겠지만...)

### 10.1.2 Providing a stable identity for each pod

저장 공간 뿐만 아니라, 어떤 애플리케이션의 경우 장기간 안정적으로 유지되는 state 가 필요할 수 있다. 특히 IP 와 hostname 같은 네트워크 정보의 경우 pod 이 rescheduling 되면 바뀌게 된다. 대표적으로 distributed stateful applications 의 경우 이러한 네트워크 정보가 바뀌면 클러스터 내의 정보를 수시로 업데이트 해줘야 하는 오버헤드가 발생한다.

#### Dedicated service for each pod instance

위 문제를 회피하는 방법으로는 service 를 생성하여 IP 를 할당하고 각 pod 가 service endpoints 에 추가되도록 하면 된다.

이 방법도 그리 깔끔하지 않다. 각 pod 들은 자신이 어떤 service 를 통해 expose 되어있는지 알 수 없고 (자신의 stable IP 를 모른다), 다른 pod 들에게 자신의 IP 를 알릴 수 없다.

Kubernetes 에서는 이러한 문제를 **StatefulSet** 으로 해결한다.

## 10.2 Understanding StatefulSets

---

이런 경우, ReplicaSet 보다는 StatefulSet 을 사용하여 각 pod 들이 stable 한 이름과 상태를 갖도록 한다.

### 10.2.1 StatefulSets vs ReplicaSets

#### Pets vs Cattle 비유

우리는 앱 인스턴스를 pet 처럼 대하려는 경향이 있다. 이름을 부여하고, 각 인스턴스를 개별적으로 고려한다. 하지만, 보통 인스턴스를 cattle 로 대하고 개별적인 인스턴스에는 특별히 관심을 주지 않는 것이 나을 때가 많다. 이렇게 하면 문제가 있는 인스턴스를 고민 없이 교체할 수 있다.

상태가 없는 앱의 경우 cattle 과 같다. 인스턴스 하나에 문제가 생겨도, 새로 만들어서 교체해버리면 겉으로는 차이가 보이지 않는다. 반면, 상태가 있는 앱의 경우 pet 과 같다. 인스턴스에 문제가 생기면, 마치 pet 이 사라졌다고 티나지 않게 대체가 불가능하듯, 인스턴스도 대체가 불가능하다. 대체하려면 기존의 인스턴스와 정확히 똑같은 상태를 가지고 있어야 한다.

#### StatefulSets vs ReplicaSets / ReplicationControllers

그래서 ReplicaSet 이나 ReplicationController 에 의해 관리되는 replica 들은 cattle 에 가깝다. 상태를 가지고 있지 않으므로 언제든 교체될 수 있다.

반면 상태를 갖는 (stateful) 앱의 경우 pod 인스턴스 가 죽으면 새로운 인스턴스 를 생성할 때 전의 인스턴스와 동일한 이름, 네트워크 정보와 상태를 가져야 한다. *StatefulSet 을 사용하면 이런 일이 가능해진다.*

StatefulSet 은 pod 들이 자신의 정보와 상태를 유지하면서 reschedule 될 수 있도록 보장해준다. 이뿐만 아니라 ReplicaSet 때처럼 replica count 를 지정할 수 있어 scaling 이 가능하며, pod template 을 지정할 수도 있다.

ReplicaSet 과 다른 점은 StatefulSet 이 생성한 pod 들은 서로의 exact copy 가 아니라는 점이다. 각 pod 마다 자신만의 volume 을 가질 수도 있고, 생성될 때마다 예측하기 쉬운 (그리고 안정적인) 정보를 갖는다.

> 여기서 예측하기 쉽다는게...?

### 10.2.2 Providing a stable network identity

StatefulSet 을 이용해서 pod 를 생성하면 zero-based index 를 기반으로 번호가 주어진다. 그 번호를 이용해 pod 의 이름과 hostname 을 가져오고, 각 pod 에 storage 를 붙일 수 있게 된다.

이름에 번호가 주어지기 때문에 이름은 예측하기 쉬우며, 랜덤한 이름이 부여될 때보다 잘 정리되어 있다.

#### Governing service

상태를 갖는 (stateful) pod 의 경우, 서로 통신할 때 특정 상태를 갖는 특정 pod 에서 작업을 요청하게 되므로 hostname 으로 reference 가 가능해야한다.

그래서 StatefulSet 을 사용하면 StatefulSet 에 대응하는 governing headless service 를 생성하여 각 pod 에 네트워크 정보를 부여할 것이 강제된다. 이렇게 하면 service 를 통해 각 pod 는 DNS entry 를 갖게 되며, 클러스터 내의 다른 리소스에서 hostname 을 확인할 수 있게 된다. (FQDN 사용)

#### Replacing lost pets

StatefulSet 으로 관리되는 pod 중 하나가 사라지면, StatefulSet 이 자동으로 reschedule 해준다. 이 때 ReplicaSet 과는 달리, 새로 생긴 pod 는 사라진 pod 와 동일한 이름과 hostname 을 갖도록 생성된다.

#### Scaling a StatefulSet

Scale up 하는 경우, replica count 에 맞게 index 값이 이름에 부여된다. 반면 scale down 하는 경우 index 가 가장 큰 pod 부터 삭제하므로, 어떤 pod 가 삭제될지 사전에 알 수 있다.

몇몇 stateful 애플리케이션은 빠른 scale down 을 잘 처리하지 못하는 경우가 있어 StatefulSet 에서는 한 번에 하나씩만 scale down 할 수 있다.

> Distributed data store 의 경우 여러 개의 pod 가 한꺼번에 삭제되면, 데이터가 유실될 수 있다. 하나씩만 scale down 을 지원하게 되면 데이터의 copy 가 추가로 없는 경우 다른 곳에 백업을 해둘 수 있게 된다.

이러한 이유로 만약 임의의 인스턴스가 정상이 아니면, StatefulSet 의 scale down 은 불가능하다.

> Scalue up 할 때도 1개씩 하는 것 같다!

### 10.2.3 Providing stable dedicated storage to each stateful instance

위에서 pod 의 정보를 안정적으로 유지하는 방법을 살펴봤는데, 저장공간은 어떻게 하는가? 특히 rescheduling 되는 경우에도 잘 처리해줘야 한다.

당연히 stateful pod 와 연결된 storage 는 persistent 해야 하고 pod 에 종속적이지 않아야한다.

한편 PVC 의 경우 PV 와 일대일로 대응되기 때문에 StatefulSet 의 pod 들은 각각 다른 PVC 를 reference 해야 하는 상황이 된다. StatefulSet 은 이 문제도 해결해준다.

#### Volume claim template

StatefulSet 이 PVC 도 만들어 주는데, volume claim template 을 사용하면 pod 를 새로 띄울 때 자연스럽게 PVC 를 pod 에 연결해준다.

#### PVC 의 생성과 삭제 이해

StatefulSet 을 scale up 하는 경우 pod 와 PVC 가 새롭게 생기지만, scale down 하는 경우 pod 만 삭제하고 PVC 는 그대로 둔다. 만약 PVC 를 지워버리면 PV 는 recycle 되거나 삭제되어 내용이 유실된다.

StatefulSet 을 사용하는 경우는 말 그대로 state 가 중요하므로, pod 가 지워졌더라도 PV 안의 데이터는 중요하다. 그래서 PV 를 free 하려면 PVC 를 수동으로 지워줘야 한다.

#### PVC reattach

또한 scale down 이후 PVC 가 유지되기 때문에, 다시 scale up 하는 경우 같은 PVC 를 새로운 pod 에 연결하여 예전 pod 가 삭제되기 전 PV 의 상태를 그대로 사용할 수 있게 된다. 실수로 scale down 해도 원상 복구가 가능하다.

### 10.2.4 Understanding StatefulSet guarantees

StatefulSet 들이 무엇을 보장해주는지 살펴본다.

#### Implications of stable identity and storage

만약 Kubernetes 가 새로운 pod 를 생성했는데 기존 pod 가 사실 삭제되지 않았다면, 같은 정보를 가진 2개의 pod 가 동시에 존재하는 상황이 생길 수도 있다. 같은 저장소를 사용하므로 각 pod 내의 프로세스가 같은 파일에 write 할 수도 있다.

#### StatefulSet's at-most-one semantics

따라서 Kubernetes 에서는 같은 정보를 가지고 같은 PVC 에 bind 된 stateful pod 가 동시에 2개 이상 존재하지 않도록 특별히 주의한다. 즉 StatefulSet 은 stateful pod 가 최대 1개만 존재하도록 보장해야 하며, 이를 *at-most-one* semantics 라고 한다.

따라서 StatefulSet 의 입장에서는 새로운 pod 를 생성하기 전에 생성할 pod 의 정보와 같은 정보를 가진 pod 가 없음을 **확신**할 수 있어야 한다. 이 때문에 노드에 문제가 생기는 경우 처리 방법이 크게 달라진다.

뒤에서 더 자세히 살펴보고, 우선 StatefulSet 을 생성하는 방법부터 살펴본다.

## 10.3 Using a StatefulSet

---

### 10.3.1 Creating the app and container image

실습에 사용할 앱은 POST 요청을 받으면 request body 를 `/var/data/kubia.txt` 에 저장하고, GET 요청을 받으면 hostname 과 `/var/data/kubia.txt` 의 내용을 돌려준다.

### 10.3.2 Deploying the app through a StatefulSet

StatefulSet 을 이용해 앱을 배포하기 위해서는 PV 와 governing service 를 만들어줘야 한다.

#### PV 생성하기 (Without dynamic provisioning)

실습에서는 replica 를 3개 만들 것이므로 PV 를 3개 만들어야 한다.

```yaml
kind: List
apiVersion: v1
items:
- apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: pv-a
  spec:
    capacity:
      storage: 1Mi
    accessModes:
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Recycle
    hostPath:
      path: /tmp/pv-a
- apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: pv-b
  spec:
    capacity:
      storage: 1Mi
    accessModes:
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Recycle
    hostPath:
      path: /tmp/pv-b
- apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: pv-c
  spec:
    capacity:
      storage: 1Mi
    accessModes:
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Recycle
    hostPath:
      path: /tmp/pv-c
```

`List` 로 만들어서 여러 object 를 한꺼번에 생성할 수 있다.

#### Governing service

Headless service 도 하나 생성한다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia
spec:
  clusterIP: None   # headless
  selector:
    app: kubia
  ports:
  - name: http
    port: 80
```

#### StatefulSet 생성

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: kubia
spec:
  serviceName: kubia
  replicas: 2
  selector:
    matchLabels:
      app: kubia        # app=kubia label 이 있어야 선택
  template:
    metadata:
      labels:
        app: kubia
    spec:
      containers:
      - name: kubia
        image: luksa/kubia-pet
        ports:
        - name: http
          containerPort: 8080
        volumeMounts:
        - name: data
          mountPath: /var/data      # volume mount 위치
  volumeClaimTemplates:         # PVC template
  - metadata:
      name: data
    spec:
      resources:
        requests:
          storage: 1Mi
      accessModes:
      - ReadWriteOnce
```

ReplicaSet, Deployment 와 다른 점은 `volumeClaimTemplate` 가 있다는 점이다. `data` 라는 이름의 volume claim template 을 만들어서 pod 이 생길 때마다 PVC 를 하나씩 생성하고 pod 에 연결해준다.

Replica 수를 2로 설정했는데, pod 은 하나씩 생성된다. 이는 race condition 을 방지하기 위해서이다.

```
$ kubectl get pod
NAME      READY   STATUS    RESTARTS   AGE
kubia-0   1/1     Running   0          3m37s
kubia-1   1/1     Running   0          3m29s
```

Pod 이름 뒤에 zero-based index 가 추가되어있는 것도 확인 가능하다.

```
$ kubectl get pvc
NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-kubia-0   Bound    pvc-9ea59858-ecc9-4549-ba87-4db46e8f1f79   1Mi        RWO            standard       98s
data-kubia-1   Bound    pvc-7a58e057-beb9-40a8-adc2-8c46043fbce2   1Mi        RWO            standard       92s
```

또 PVC 가 생성된 것도 확인할 수 있다.

> 분명 앞에서 PV 생성해 뒀는데 그걸 안 쓰고 minikube 가 알아서 PV 를 새로 만들어서 사용해 버린다.

### 10.3.3 Playing with your pods

Pod 에 접속을 해보자. 단 service 가 headless 하므로 각 pod 에 직접 접속해야한다.

보통은 pod 안에 들어가서 `curl` 을 때려보거나 포트 포워딩을 하겠지만 이번에는 API server 를 사용하여 pod 로 proxy 해본다.

#### Communicating with pods through the API server

다음 URL 로 보내면 특정 pod 로 요청을 보낼 수 있게 된다.

```
<apiServerHost>:<port>/api/v1/namespaces/default/pods/<pod-name>/proxy/<path>
```

한편 API server 에 요청을 보내는 것은 인증 절차 등이 필요해 복잡하다는 것을 알고 있다. 그래서 `kubectl proxy` 를 이용해서 요청을 보낸다. 이제 아래와 같이 하면 `kubia-0` pod 에 요청을 보낼 수 있다.

```
$ curl localhost:8001/api/v1/namespaces/default/pods/kubia-0/proxy/
You've hit kubia-0
Data stored on this pod: No data posted yet
```

이제 POST 요청을 보내본다.

```
$ curl -X POST localhost:8001/api/v1/namespaces/default/pods/kubia-0/proxy/ \
    -d "This is a request for kubia-0 pod." 
Data stored on pod kubia-0
```

다시 GET 요청을 보내보면 응답이 정상적으로 온다.

```
$ curl localhost:8001/api/v1/namespaces/default/pods/kubia-0/proxy/
You've hit kubia-0
Data stored on this pod: This is a request for kubia-0 pod.
```

한편, `kubia-1` pod 에는 아무것도 저장되어 있지 않다.

```
$ curl localhost:8001/api/v1/namespaces/default/pods/kubia-1/proxy/
You've hit kubia-1
Data stored on this pod: No data posted yet
```

확실히 각 pod 가 state 를 갖는다는 사실을 확인할 수 있다.

#### Deleting a stateful pod

이제 pod 를 한 번 지워보고 rescheduling 될 때 같은 storage 에 연결되는지 확인할 것이다.

먼저 `kubectl delete pod kubia-0` 를 입력한 뒤 pod 가 삭제될 때까지 대기한다. 조금 기다리면 StatefulSet 이 새로 pod 를 생성하는 것을 확인할 수 있다.

> Pod 삭제 명령을 입력했는데 엄청 오래걸리는 것은 기분 탓인가, 아니면 stateful pod 라서 오래걸리는 건가... 1분 20초 정도 걸렸다.

같은 storage 에 연결되었는지 확인하기 위해 GET 요청을 보내본다.

```
$ curl localhost:8001/api/v1/namespaces/default/pods/kubia-0/proxy/
You've hit kubia-0
Data stored on this pod: This is a request for kubia-0 pod.
```

이전과 같은 정보를 저장하고 있는 것을 확인했다.

#### Exposing stateful pods through a non-headless service

Service 를 만들어서 expose 해본다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia-public
spec:
  selector:
    app: kubia
  ports:
  - port: 80
    targetPort: 8080
```

이 service 는 `ClientIP` service 이므로 클러스터 안에서만 접근이 가능하다. 앞에서와 마찬가지로 API server 를 proxy 로 이용해 접속할 수 있다.

```
$ curl localhost:8001/api/v1/namespaces/default/services/kubia-public/proxy/
You've hit kubia-3
Data stored on this pod: No data posted yet
```

다만 random 한 pod 에서 응답을 보내준다.

## 10.4 Discovering peers in a StatefulSet

---

마지막으로 살펴볼 내용은 peer discovery 이다. StatefulSet 의 pod 이 다른 pod 를 발견할 수 있어야 한다. 물론 API server 를 사용할수도 있겠지만, 애플리케이션이 직접 요청을 보내게 되기 때문에 Kubernetes 에 종속되게 된다. 다른 방법이 필요하며, 여기서는 SRV record 를 사용할 것이다.

#### SRV records

특정 서비스를 제공하는 hostname 과 port 를 알려주기 위한 record 이다.

실행한 pod 들의 SRV record 를 확인하기 위해 `dig` 를 사용할 것이다.

```
$ kubectl run -it srvlookup --image=tutum/dnsutils --rm \
    --restart=Never -- dig SRV kubia.default.svc.cluster.local
```

조금 기다리면 다음과 같이 결과가 나온다.

```
;; ANSWER SECTION:
kubia.default.svc.cluster.local. 30 IN	SRV	0 25 80 kubia-0.kubia.default.svc.cluster.local.
kubia.default.svc.cluster.local. 30 IN	SRV	0 25 80 kubia-1.kubia.default.svc.cluster.local.

;; ADDITIONAL SECTION:
kubia-0.kubia.default.svc.cluster.local. 30 IN A 172.17.0.3
kubia-2.kubia.default.svc.cluster.local. 30 IN A 172.17.0.6
```

`ANSWER SECTION` 을 보면 service 뒤에 있는 2개의 pod 에 대한 record 가 나온다. 각 pod 는 A record 도 가지고 있으며, `ADDITIONAL SECTION` 에 적혀있다.

그러므로 pod 가 StatefulSet 내의 다른 pod 정보를 얻고 싶다면, SRV DNS lookup 한 번을 해주면 된다.

> 참고로 결과에 있는 SRV record 의 순서는 임의로 변경될 수 있다.

### 10.4.1 Implementing peer discovery through DNS

이제 pod 들이 서로 통신하도록 하려고 한다.

이제 client 는 `kubia-public` service 를 이용해서 요청을 보낼 것인데, 그러면 요청이 임의의 pod 로 전달될 수 있다. 클러스터에 pod 가 여러 개 있으므로 데이터를 여러 개 저장할 수 있으나, 요청이 모든 pod 에 한 번씩 도달할 때까지 요청을 계속 반복해야 한다.

따라서 pod 가 요청을 받으면 다른 pod 로부터 정보를 모두 받아 데이터를 돌려주도록 할 것이다. 이를 위해 SRV DNS lookup 을 하고, 각 record 에 요청을 보내 데이터를 받아온 다음 돌려주면 된다.

(구현체는 생략)

### 10.4.2 Updating a StatefulSet

`kubectl edit statefulset <NAME>` 을 이용하면 된다. 실습을 위해 replica count 는 3으로, container 도 새롭게 구현한 container 로 교체해줄 것이다.

수정이 완료되면 자동으로 rollout 이 진행된다.

```
$ kubectl rollout status statefulset kubia
Waiting for 1 pods to be ready...
Waiting for partitioned roll out to finish: 2 out of 3 new pods have been updated...
Waiting for 1 pods to be ready...
partitioned roll out complete: 3 new pods have been updated...
```

### 10.4.3 Trying out your clustered data store

이제 테스트를 해본다. 먼저 데이터를 POST 한다. 요청은 *service* 로 보내야 한다.

```
$ curl -X POST localhost:8001/api/v1/namespaces/default/services/kubia-public/proxy/ \
    -d "Good morning"                   
Data stored on pod kubia-1

$ curl -X POST localhost:8001/api/v1/namespaces/default/services/kubia-public/proxy/ \
    -d "Good afternoon" 
Data stored on pod kubia-0

$ curl -X POST localhost:8001/api/v1/namespaces/default/services/kubia-public/proxy/ \
    -d "Some random data" 
Data stored on pod kubia-2
```

이제 데이터를 읽어온다.

```
$ curl localhost:8001/api/v1/namespaces/default/services/kubia-public/proxy/          
You've hit kubia-2
Data stored in the cluster:
- kubia-2.kubia.default.svc.cluster.local: Some random data
- kubia-0.kubia.default.svc.cluster.local: Good afternoon
- kubia-1.kubia.default.svc.cluster.local: Good evening
```

Pod 가 직접 peer discovery 를 진행하므로 scaling 에도 유연하게 대처할 수 있다.

## 10.5 Understanding how StatefulSets deal with node failures

---

10.2.4 에서 *at-most-one* semantics 를 설명하면서 StatefulSet 은 같은 상태를 가진 pod 를 2개 만들어서는 안된다고 했었다. 한편 노드가 작동을 중단하는 경우 Kubernetes 는 그 노드에 있던 리소스의 정보를 알 수 없게 된다. Pod 가 실제로 작동을 중지한 것인지, 아니면 접근이 가능한데 그냥 Kubelet 이 노드의 상태를 master 에 보고하는 것을 중단했을 수도 있다.

그래서 StatefulSet 은 클러스터 관리자가 알려주기 전까지는 대체 pod 를 생성하지 않게된다. 클러스터 관리자는 pod 를 완전히 삭제하거나, 노드를 삭제할 수도 있다.

마지막으로 한 노드의 네트워크가 유실되는 상황에서 StatefulSet 의 동작을 살펴볼 것이다.

### 10.5.1 Simulating a node's disconnection from the network

minikube 에서는 안된다! GKE 의 이야기이다.

노드에 ssh 로 접속해서 `sudo ifconfig eth0 down` 을 입력하면 네트워크를 죽일 수 있다!

이제 노드의 네트워크 인터페이스가 꺼졌기 때문에 노드의 Kubelet 은 Kubernetes API server 와 연결할 수 없다. Pod 들이 잘 동작하고 있다고 보고할 수 없게 되는 것이다.

조금 기다린 뒤 `kubectl get nodes` 를 해보면 노드가 `NotReady` 상태로 변경된다. 그래서 `kubectl get pod` 를 하면 `STATUS` 가 `Unknown` 으로 변경된다.

#### Status 가 Unknown 이면 어떤 일이 일어나는가

만약 노드의 네트워크가 금방 다시 연결되면 pod 상태를 다시 보고할 것이므로 `Ready` 상태가 된다. 반면 오랜 시간동안 (이 시간은 설정 가능하다) `Unknown` 상태이면 Kubernetes control plane 에서 pod 를 자동으로 삭제한다.

Kubelet 이 pod 이 삭제 명령을 받으면 삭제를 시작하여 `Terminating` 으로 변경되는데, 지금 상황에서는 네트워크가 유실되었으므로 Kubelet 이 pod 삭제 명령을 알 수 없다. 그래서 pod 는 게속 실행 중이게 된다.

`kubectl describe pod <NAME>` 으로 유실된 노드의 pod 를 살펴보면 `Terminating` 으로 변경되어있고, `Reason: NodeLost` 라고 적혀있다. 단, 이는 어디까지나 control plane 의 관점이며, 실제로 노드 안에서 pod 는 정상적으로 돌아가고 있다.

### 10.5.2 Deleting the pod manually

Rescheduling 을 하려면 수동으로 삭제해줘야 한다. 한편 `kubectl delete pod` 명령으로는 삭제가 불가능하다. 네트워크가 유실되었기 때문에 이 명령이 전달되지 않는다. 강제 삭제해야한다.

**이 명령은 노드가 더 이상 동작하지 않는 다는 것을 확신할 때만 입력해야 한다.**

```bash
$ kubectl delete pod <NAME> --force --grace-period 0
```

이렇게 `--force`, `--grace-period 0` 옵션을 모두 줘서 삭제해야 한다. 원래는 Kubelet 이 컨테이너가 모두 중단되었고, 삭제가 완료되었다고 알려줘야 하지만 이 옵션을 주게 되면 기다리지 않고 즉시 삭제한 것으로 처리한다.

---

## Discussion & Additional Topics

### Pets vs Cattle Analogy

- http://cloudscaling.com/blog/cloud-computing/the-history-of-pets-vs-cattle/
