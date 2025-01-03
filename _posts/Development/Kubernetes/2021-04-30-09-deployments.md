---
share: true
toc: true
categories: [Development, Kubernetes]
path: "_posts/development/kubernetes"
tags: [kubernetes, sre, devops]
title: "09. Deployments: Updating Applications Declaratively"
date: "2021-04-30"
github_title: "2021-04-30-09-deployments"
image:
  path: /assets/img/posts/development/kubernetes/k8s-09.jpeg
attachment:
  folder: assets/img/posts/development/kubernetes
---

![k8s-09.jpeg](/assets/img/posts/development/kubernetes/k8s-09.jpeg) _Rolling update of Deployments (출처: livebook.manning.com/book/kubernetes-in-action/chapter-9)_

### 주요 내용

- Deployment 를 사용하여...
  - 무중단 배포를 하는 방법
  - 롤백하는 방법

## 9.1 Updating applications running in pods

---

만약 서비스/앱을 업데이트하고 싶다면 2가지 방법이 있다.

1. Pod 를 모두 지운 후, 업데이트 된 pod 로 새롭게 시작
2. 새로운 pod 를 시작한 뒤 새로운 pod 가 요청을 처리할 준비가 되면 기존 pod 를 지운다.

두 방법 모두 장단이 있다. 1번의 경우 잠시 서비스가 중단된다는 문제가 있고, 2번의 경우 배포 중에 2가지 버전의 서비스가 동시에 존재하게 되므로 이를 잘 처리하기 위해서 서비스/앱 단에서 처리해줘야 한다. (하위 호환성)

### 9.1.1 삭제 후 새롭게 생성

ReplicationController / ReplicaSet 의 경우 pod template 을 수정할 수 있었다.

- Pod template 변경 (`kubectl apply -f ...`)
- Old pod 모두 삭제
- ReplicationController / ReplicaSet 이 삭제를 감지하고 새로운 pod template 으로 생성

당연히, rc/rs 가 삭제를 감지하고 pod 이 요청을 처리할 준비가 될 때까지 서비스가 중단된다.

### 9.1.2 새로운 pod 생성 후 기존 pod 삭제

#### 한 번에 새로운 pod 로 교체하기

이 방법을 사용할 때 고려해야할 점이 있다면, 새로운 pod 를 생성한 뒤 삭제하는 것이므로 평소보다 2배 많은 pod 가 존재하게 되어 리소스를 많이 사용하게 된다. (이를 버틸 수 있는 리소스가 있어야 한다)

이 방법도 간단하다. Pod 앞에 Service 가 붙어있을 것이므로, Service 의 label selector 롤 고쳐주면 된다.

- 새로운 pod template 을 사용하는 ReplicaSet 생성
- ReplicaSet 이 만든 pod 들이 모두 준비되었는지 확인
- Service 의 label selector 를 교체 (`kubectl set selector ...`)

이 방법은 *blue-green deployment* 라고 한다.

#### 롤링 업데이트

Rolling update 방식에서는 pod 를 조금씩 교체한다.

이를 수동으로 하는 경우에는 rc/rs 의 replica 수를 조절해야한다. 기존의 rc/rs 에서는 replica 수를 줄이고, 새로운 rc/rs 에서는 replica 수를 늘려야 한다. 이 때 Service 의 pod selector 가 새롭게 생성되는 pod 들도 포함할 수 있어야 한다.

롤링 업데이트는 수동으로 하면 실수할 확률이 매우 높으므로, 자동화하는 것이 좋다.

## 9.2 Performing an automatic rolling update with a ReplicationController

---

> 아래에 소개되는 방법은 이제는 사용하지 않는 방법이다!

### 9.2.1 Initial version 실행하기

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: kubia-v1
spec:
  replicas: 3
  template:
    metadata:
      name: kubia
      labels:
        app: kubia
    spec:
      containers:
      - image: luksa/kubia:v1
        name: nodejs
---
apiVersion: v1
kind: Service
metadata:
  name: kubia
spec:
  type: NodePort        # minikube 인 관계로 NodePort Service 이용
  selector:
    app: kubia
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30123
```

ReplicationController 는 3개의 pod 를 생성할 것이고, Service 가 해당 pod 들을 관리할 것이다.

> minikube 를 사용해 NodePort Service 를 생성하게 되면 minikube 의 IP 주소로 요청을 보내야 한다. `minikube ip` 로 확인하면 된다. 혹은 `minikube service list` 를 입력하면 포트까지 포함된 URL 이 출력된다.

### 9.2.2 `kubectl` 로 rolling update 하기

> Note. 현재 `kubectl` 에 존재하지 않는 명령어이다.

```bash
$ kubectl rolling-update kubia-v1 kubia-v2 --image=luksa/kubia:v2
```

ReplicationController `kubia-v1` 을 `kubia-v2` 로 업데이트하고, 이미지를 `luksa/kubia:v2` 를 사용하겠다는 의미이다. 명령을 실행하면 ReplicationController `kubia-v2` 가 생성되고 롤링 업데이트가 진행된다.

#### 동작 과정

`kubectl` 은 `kubia-v1` ReplicationController 의 pod template 을 그대로 가져와서 image 만 바꿔치기한 후 `kubia-v2` 를 만든다.

또한 label 에 `deployment=...` 이 추가된 것을 확인할 수 있다. 만약 label 이 변경되지 않는다면 같은 label selector 를 갖는 ReplicationController 가 2개 존재하게 될 것이므로 문제가 생긴다. (새로 생긴 rc 가 동작할 필요가 없어진다)

그래서 `kubectl` 은 ReplicationController 에 label selector 를 추가하고, rc 가 관리하고 있던 pod 의 label 에 `deployment=...` 을 추가해준다.

Label 변경이 끝나면 본격적으로 scaling 이 시작되고, pod 는 하나씩 교체된다. (rc 의 replica 수를 하나씩 감소/증가 시키는 것이다)

모든 pod 의 업데이트가 끝나면 기존의 rc 는 삭제된다.

> label 도 원래대로 돌아오나 조금 궁금하긴 한데, 존재하지 않는 명령이라 확인이 불가능하다.

### 9.2.3 `kubectl rolling-update` is now obsolete

- 사용자가 생성한 object 들을 Kubernetes 가 직접 수정하는 것이 좋지 않다.
  - 위에서 `deployment=...` label 이 추가되는 것처럼
- `kubectl` *client* 가 모든 작업을 수행한다.
  - `kubectl rolling-update ... --v 6` 을 하면 로그를 볼 수 있다.
  - 로그를 보면 `kubectl` 이 API 서버에 보내는 요청을 하나씩 확인할 수 있다.
  - Client 가 작업을 수행하게 되면 중간에 네트워크 연결이 유실되면 업데이트가 멈춰버린다.
- 이 방법은 imperative 하다.
  - **Kubernetes 를 사용할 때는 어떤 목표 도달 상태에 대해서 알려만 주고, 그것을 Kubernetes 가 알아서 수행하도록 해야한다.**
  - 반면 `rolling-update` 과정에서는 replica 수를 하나씩 조절한다. (좋지 않음)

> `--v 6` 하면 로깅 레벨을 높여서 `kubectl` 이 API 서버에 보내는 요청을 볼 수 있다.

## 9.3 Using Deployments for updating apps declaratively

---

> Declarative 와 imperative 가 서로 상반되는 개념인듯 하다.

앱 배포 및 업데이트 위해 사용하는 object 이다. Deployment 를 생성하게 되면 내부적으로 ReplicaSet 을 자동으로 생성하며, ReplicaSet 이 pod 들을 관리해주게 된다.

> ReplicationController 는 쓰지 말라고 공식 문서에 나와있다.

Deployment 를 사용하면 rc/rs 와 같은 lower-level construct 들을 사용하지 않고도 업데이트를 쉽게 할 수 있게 된다. Deployment 에 목표 도달 상태를 정해주기만 하면 나머지는 Kubernetes 가 알아서 한다.

### 9.3.1 Deployment 생성

ReplicationController 와 상당히 비슷하다. Deployment 에도 label selector, replica count, pod template 가 있으며, 추가로 배포 전략을 정해주는 strategy 필드를 갖고있다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubia           # 이름에 버전 정보가 들어갈 필요없다!
spec:
  replicas: 3
  template:
    metadata:
      name: kubia
      labels:
        app: kubia
    spec:
      containers:
      - image: luksa/kubia:v1
        name: nodejs
  selector:
    matchLabels:
      app: kubia
```

생성할 때 `--record` 옵션을 줘서 revision history 에 명령을 저장할 수 있도록 한다.

#### Deployment rollout

`kubectl get deployment`, `kubectl describe deployment` 로 Deployment 의 상태를 확인할 수도 있지만, `rollout` 을 사용해서 확인하는 방법이 있다. (이 목적으로 만들어졌다)

```
$ kubectl rollout status deployment kubia
deployment "kubia" successfully rolled out
```

#### 생성된 pod, ReplicaSet 확인

`kubectl get po` 로 만들어진 pod 들을 확인할 수 있고, `kubectl get rs` 를 해보면 ReplicaSet 이 생겼다는 것을 알 수 있다.

그리고 ReplicaSet 의 이름에는 pod template 의 hash 값이 들어가 있는데, 이렇게 하는 이유는 Deployment 가 직접 pod 를 제어하지 않고 ReplicaSet 이 하기 때문이다. Deployment 는 ReplicaSet 을 여러 개 만들기도 하는데, 이때 이미 존재하는 ReplicaSet 을 추가로 만들지 않고 그대로 사용할 수 있게 된다. (pod template 이 같다면 hash 값이 일치하므로 재사용이 가능하다.)

> 앞에서 `kubectl rolling-update` 를 사용할 때는 ReplicationController 에 label 이 추가되는 부분이 있었는데, 이는 같은 label selector 를 사용하는 여러 개의 ReplicationController 생성을 막기 위한 것이었다.
> 
> ReplicaSet 도 이 문제에서 자유롭지는 못할 것 같아서 확인해보니, label selector 에 `pod-template-hash` 가 있는 것을 확인할 수 있었다.
> 
> ```
> $ kubectl get po --show-labels                          
> NAME                     READY   STATUS    RESTARTS   AGE   LABELS
> kubia-74967b5695-bpqfm   1/1     Running   0          17s   app=kubia, pod-template-hash=74967b5695
> kubia-74967b5695-djsfb   1/1     Running   0          17s   app=kubia,pod-template-hash=74967b5695
> kubia-74967b5695-h7d2b   1/1     Running   0          17s   app=kubia,pod-template-hash=74967b5695
> ```
> 
> 실제로 pod template hash 값을 사용하나보다.

#### Service 생성

Service 를 생성하여 Deployment 가 생성한 pod 에 연결할 수 있도록 해준다.

> Service 랑 Deployment 는 따로 띄워야 하나보다.

### 9.3.2 Updating a Deployment

ReplicationController 를 사용할 때보다 훨씬 간단해졌다. YAML 에서 pod template 만 고쳐주면 나머지는 Kubernetes 가 알아서 해준다!

> ReplicationController 를 사용할 때는 새로운 rc 의 이름을 정해줘야 하고, 중간에 문제가 생기지는 않았는지 `kubectl rolling-update` 명령이 끝날 때까지 기다려야 했다.

#### Deployment strategies

배포 전략에는 2가지가 있다. 하나는 rolling update 방식이고 이 값이 기본값이다. 나머지 하나는 `Recreate` 로, 9.1.1 에서 다룬 것과 같이 pod 를 모두 삭제한 뒤 새로운 pod 를 띄우는 방식이다.

> 만약 다른 버전의 앱이 여러 개 동작하고 있는 것이 서비스 구조상 불가능하다면 `Recreate` 를 사용할 수 있을 것이다. 서비스가 잠시 중단되는 것은 마찬가지이다.

#### 실습을 위해 rolling update 천천히 하도록 설정

'Rolling' 이 잘 되고있는지 확인하기 위해 `minReadySeconds` 필드를 Deployment 에 추가한다. 필드 하나를 추가하는 것이므로 `kubectl patch` 를 사용한다.

```bash
$ kubectl patch deployment kubia -p '{"spec": {"minReadySeconds": 10}}'
```

> `minReadySeconds` 는 해당 pod 가 available 상태라고 판단하기 위한 최소한의 `READY` 시간이다. 이 때, 설정한 시간만큼 해당 pod 의 어떠한 컨테이너도 crash 하지 않고 살아있어야 한다.

#### Rolling update

이제 pod 에서 사용하는 이미지를 `luksa/kubia:v2` 로 변경하면 된다. `kubectl patch` 로도 할 수 있지만, `kubectl set image` 를 사용할 것이다.

```bash
$ kubectl set image deployment kubia nodejs=luksa/kubia:v2
```

`kubia` 이름을 가진 Deployment 의 `nodejs` 컨테이너가 사용하는 이미지를 `luksa/kubia:v2` 로 변경하라는 의미이다. 실행하면 Deployment 의 pod template 이 변경된다.

변경을 감지한 Kubernetes 가 rolling update 를 알아서 해준다.

#### Deployment 의 편리함

필드 하나를 바꿨는데 rolling update 가 된다! 이 작업은 `kubectl` client 가 한것이 아니고 Kubernetes control plane 에서 처리한 것이다.

기존 pod 를 하나씩 지우고 새로운 pod 를 하나씩 생성하는 전체적인 과정은 `rolling-update` 명령과 동일하다. 반면 업데이트가 끝난 다음에도 ReplicaSet 이 남아있게 되는데, 사용자는 Deployment 를 생성했지 ReplicaSet 을 직접 생성한 것이 아니므로 일단 고려하지 않아도 된다.

오히려 ReplicaSet 이 남아있기 때문에 이를 활용할 수 있게 된다.

### 9.3.3 Deployment 롤백하기

의도적으로 버그가 있는 버전을 배포해본다. Pod template 의 이미지를 변경한 후, `kubectl rollout` 명령을 이용해 rolling update 과정을 확인할 수 있다.

```
kubectl rollout status deployment kubia                                                          
Waiting for deployment "kubia" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "kubia" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "kubia" rollout to finish: 1 old replicas are pending termination...
deployment "kubia" successfully rolled out
```

`curl` 을 때려보면 에러가 발생하는 것을 확인할 수 있다.

롤백을 위해서는 다음과 같이 마지막 rollout 을 취소하면 된다.

```
$ kubectl rollout undo deployment kubia
deployment.apps/kubia rolled back
```

> `undo` 명령은 rollout 이 진행 중인 경우에도 사용할 수 있다. 새로 생성된 pod 들은 삭제되고 삭제되었던 pod 들은 다시 생성된다.

#### Rollout history 확인하기

롤백이 가능한 이유는 history 가 저장되고 있기 때문이다. ReplicaSet 안에 revision history 가 저장되고 있다. Rollout 이 완료되었을 때 ReplicaSet 이 삭제되지 않으므로, 임의의 revision 으로 롤백할 수 있게 된다.

History 확인은 `kubectl rollout history deployment <NAME>` 으로 한다.

```
$ kubectl rollout history deployment kubia
deployment.apps/kubia 
REVISION  CHANGE-CAUSE
1         kubectl create --filename=kubia-deployment-v1.yaml --record=true
3         kubectl create --filename=kubia-deployment-v1.yaml --record=true
4         kubectl create --filename=kubia-deployment-v1.yaml --record=true
```

특정 revision 으로 롤백하고 싶다면,

```bash
$ kubectl rollout undo deployment kubia --to-revision=1
```

과 같이 `--to-revision` 옵션을 사용하면 된다.

위에서 Deployment 생성시 `--record` 명령을 사용했기 때문에 `CHANGE-CAUSE` 에 revision history 가 상세하게 남게 된다.

> `--record` 를 사용하지 않고 Deployment 를 생성하면 history 에서 `CHANGE-CAUSE` 열이 `<none> ` 으로 표시된다.
> 
> ```
> $ kubectl rollout history deployment kubia
> deployment.apps/kubia 
> REVISION  CHANGE-CAUSE
> 1        \ <none\> 
> 2         \<none\> 
> ```
> 
> 또한, ReplicaSet 을 강제로 삭제하게 되면 그와 관련된 revision 도 함께 삭제된다.
> 
> ```
> $ kubectl rollout history deployment kubia
> deployment.apps/kubia 
> REVISION  CHANGE-CAUSE
> 2         \<none\> 
> ```
> 
> 그러므로 `kubectl rollout undo` 도 사용이 불가능하다.
> 
> ```
> $ kubectl rollout undo deployment kubia   
> error: no rollout history found for deployment "kubia"
> ```
> 
> 그냥 궁금해서 테스트했는데 뒤에 나와있다. ㅋㅋ

그렇다면 revision 이 생길 때마다 ReplicaSet 이 남아있게 되므로 이는 작업할 때 매우 불편할 것이다. 그래서 `revisionHistoryLimit` 옵션을 Deployment 에 주게 되면 해당 개수 만큼만 revision history 가 남는다.

> `revisionHistoryLimit` 을 0으로 설정하게 되면 ReplicaSet 이 자동으로 정리된다. 하지만 이 경우 롤백이 불가능하며, rollout 을 취소할 수 없게 된다.

> Deployment 를 수정하면서 과거의 ReplicaSet 을 다시 이용하게 되면 해당 ReplicaSet 에 있던 revision 기록은 삭제된다.

### 9.3.4 Controlling the rate of the rollout

앞에서 rolling update 를 할 때는 pod 를 하나씩 교체했다. 하지만 이것도 (rate) 설정할 수 있다.

#### `maxSurge` and `maxUnavailable`

- `maxSurge`: Replica count 보다 추가로 존재할 수 있는 pod 의 개수의 최댓값이다. 기본값은 25% (replica count 의 최대 25% 만 더 생성할 수 있다) 이다. 계산시 소수점은 올린다.
- `maxUnavailable`: Replica count 중 동작이 중지된 pod 의 개수의 최댓값이다. 기본값은 25% (replica count 의 최대 25% 만 unavailable 일 수 있다) 이다. 계산시 소수점은 버린다.

두 값 모두 백분율 값이 아닌 정수로도 설정할 수 있다.

> 테스트 할 때 replica count 는 3이었으므로, `maxSurge` 는 ceil(3 * 0.25) = 1 이었고, `maxUnavailable` 은 floor(3 * 0.25) = 0 이었기 때문에 rolling update 시 pod 를 최대 한 개 추가로 생성할 수 있었던 것이며, 3개의 pod 는 항상 동작하고 있어야 하므로 새로운 pod 가 생성된 다음에서야 기존 pod 를 삭제할 수 있었던 것이다.

`maxUnavailable` 이 1일 때 반드시 최대 1개만 unavailable 이라는 의미는 아니다. `Replica count - 1` 개가 available 해야한다는 의미이다. 그래서 만약 `maxSurge` 가 2 이고 replica count 가 3이라면, 최소 2개만 available 이면 되므로 rollout 시 기존 pod 를 즉시 하나 삭제하여 2개를 available 로 남기고, 새로운 pod 3개를 생성하게 된다. 생성한 즉시에는 당연히 unavailable 이므로 이 순간에는 unavailable pod 개수는 3이다.

> `maxSurge`, `maxUnavailable` 은 replica count 에서 더하거나 빼는 값임을 기억하면 될 듯 하다.

### 9.3.5 Rollout 일시정지하기

Rollout 하는 도중 일부 pod 에만 업데이트를 적용하고 조금 지켜본 다음 정상 작동한다면 나머지에도 적용하고 싶을 수 있다. (canary release) 이런 경우 업데이트된 내용으로 pod 를 새로 띄우거나 할 필요 없이 rollout 과정을 잠시 중단하면 된다.

```
$ kubectl set image deployment kubia nodejs=luksa/kubia:v4
deployment.apps/kubia image updated

$ kubectl rollout pause deployment kubia
deployment.apps/kubia paused

$ kubectl rollout status deployment kubia
Waiting for deployment "kubia" rollout to finish: 1 out of 3 new replicas have been updated...
```

하나만 교체하고 나머지는 그대로인 상태에서 rollout 을 중지했다. `curl` 을 때려 확인해 보면 일부 요청은 새로운 버전의 pod 가 처리하고 있는 것을 확인할 수 있다.

새로운 버전을 그대로 배포해도 괜찮겠다면, rollout 을 재개한다.

```
$ kubectl rollout resume deployment kubia
deployment.apps/kubia resumed
```

#### 일시정지의 단점

새롭게 생성된 pod 가 어떤 비율이 되었을 때 자동으로 pause 해주는 기능은 없다. 사실 그래서 canary release 를 하는 올바른 방법은 새로운 Deployment 를 생성하여 적당히 scaling 하는 것이다.

#### Pause deployment

Deployment 를 일시정지 할 수 있다. Deployment 를 일시정지한 후 필요한 설정 변경을 한 뒤에 Deployment 를 resume 하면 rollout 이 진행된다.

> 왜 필요할까 생각해 보니, 여러 개의 설정을 변경하는 경우 설정을 변경할 때마다 rollout 이 trigger 될 수 있다는 문제가 있다. 이를 막기 위해 Deployment 를 pause 해둔 뒤 여러 설정을 변경하고 resume 하면 rollout 을 한 번만 수행하여 변경된 설정을 모두 반영할 수 있게 된다.

### 9.3.6 Blocking rollouts of bad versions

`minReadySeconds` 는 비정상적인 버전을 배포하는 것을 막기 위해 사용하는 옵션이다.

새로 생성된 pod 는 `READY` 상태로 `minReadySeconds` 만큼의 시간이 지나야 available 상태로 취급된다. 이 옵션이 유의미한 이유는 `maxUnavailable` 때문인데, available pod 가 일정 개수 이상 있어야 rolling update 를 할 수 있게 된다. Available pod 개수가 충족되지 않으면 rollout 이 진행될 수 없다.

> Pod 가 `READY` 상태가 되려면 모든 컨테이너에서 readiness probe 가 성공해야 한다.

실제로는 `minReadySeconds` 를 큰 값으로 설정해서, pod 가 `READY` 상태가 되어 실제 서비스 요청을 받고 나서도 오랜 시간동안 문제가 없는지 확인한 뒤 available pod 로 처리한다.

> 큰 값으로 설정하면 규모가 큰 서비스는 rolling 엄청 오래 걸리겠다.

#### 버그가 있는 버전에 readiness probe 적용하여 rollout

이번에는 컨테이너에 `readinessProbe` 를 설정할 것이다.

```yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: kubia
spec:
  replicas: 3
  minReadySeconds: 10       # READY 상태로 10초 있어야 한다
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0     # 3개는 항상 동작 중이도록 한다
    type: RollingUpdate
  template:
    metadata:
      name: kubia
      labels:
        app: kubia
    spec:
      containers:
      - image: luksa/kubia:v3
        name: nodejs
        readinessProbe:
          periodSeconds: 1    # 1초마다 GET 요청을 보낸다
          httpGet:
            path: /
            port: 8080
```

`kubectl apply -f ...` 으로 Deployment 를 업데이트한다. 그리고 rollout status 를 확인해본다.

```
$ kubectl rollout status deployment kubia                 
Waiting for deployment "kubia" rollout to finish: 1 out of 3 new replicas have been updated...
```

더 이상 출력이 되지 않는다. `kubectl get po` 로 확인해 보면 pod 하나가 `READY` 상태가 아닌 것을 볼 수 있다.

```
$ kubectl get po                                            
NAME                     READY   STATUS    RESTARTS   AGE
kubia-86499d4c5f-pdfgt   0/1     Running   0          13s
kubia-bcf9bb974-9425g    1/1     Running   0          7m10s
kubia-bcf9bb974-9cq7f    1/1     Running   0          7m12s
kubia-bcf9bb974-k7cxf    1/1     Running   0          7m14s
```

`describe po` 로 확인해 보면 readiness probe 가 실패했다고 나와있으며, `curl` 로 확인해보면 과거 버전 pod 로만 요청이 가고있음을 확인할 수 있다.

당연히, available 상태가 되지 않았으므로 rollout 은 더 이상 진행될 수 없으며 (`maxUnavailable`) `READY` 상태가 아닌 pod 들은 Service Endpoints 에서 제거되기 때문에 과거 버전 pod 로만 요청이 가게 된다.

> Service Endpoint 에서 추가/제거 되는 기준은 pod 가 `READY` 인지 아닌지 이다. 우선 `curl` 로 요청을 계속 보내고 있는 상태에서 버그가 있는 버전으로 rolling update 를 해보니 중간에 에러가 난 요청들이 보이긴 했다.
> 
> 4번째 요청까지는 OK 응답이 돌아오므로, readiness probe 가 이 pod 는 `READY` 라고 했을 것이다. 그러면서 Service Endpoints 에 이 pod 의 IP 가 추가된 것이며, 내가 보낸 `curl` 요청을 받을 수 있었던 것이다.
> 
> Readiness probe 는 주기적으로 실행되므로, 어느 시점부터 실패했을 것이므로 `READY` 상태가 아니게 되고 unavailable 상태가 지속되면서 rollout 은 중단된 것이다.
> 
> 결국 중간에 요청이 비정상적으로 처리되는 경우가 있었다는 것은 실제 서비스에서 일부 사용자들은 오류를 겪을 수도 있다는 것인데 이것이 불가피한 것인지 아닌지는 상황에 따라 다를 것 같다. 책에서는 모든 pod 가 잘못된 pod 로 교체되는 것에 비하면 낫다고 하는데, 이렇게 생각해도 과연 괜찮을련지 ㅋㅋ

#### Rollout deadline

Rollout 이 10분간 진전이 없으면 failed 처리된다. `kubectl describe` 로 확인해 보면 `ProgressDeadlineExceeded` condition 이 발생해있다.

> `kubectl rollout status` 에서는 다음과 같이 에러가 발생한다.
> 
> ```
> $ kubectl rollout status deployment kubia
> error: deployment "kubia" exceeded its progress deadline
> ```

이 값 또한 `progressDeadlineSeconds` 로 설정할 수 있다.

#### Bad rollout 중단하기

`kubectl rollout undo` 를 이용해서 중단하면 된다.

> Note 에는 future version 들에서 자동으로 중단될거라고 하는데, 나는 자동으로 중단되지 않았다. Unavailable pod 가 1시간 넘게 남아있었다. 수동으로 `undo` 해주니 그제서야 `Terminating` 으로 변경되었다.

---

## Discussion & Additional Topics

### BlueGreenDeployment

- https://martinfowler.com/bliki/BlueGreenDeployment.html

### Canary Release

- https://martinfowler.com/bliki/CanaryRelease.html
  - 일부 사용자들에게만 new version 으로 서비스 하는 것
