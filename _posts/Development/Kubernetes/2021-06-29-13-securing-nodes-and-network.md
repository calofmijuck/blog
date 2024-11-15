---
share: true
toc: true
categories: [Development, Kubernetes]
path: "_posts/development/kubernetes"
tags: [kubernetes, sre, devops]
title: "13. Securing Cluster Nodes and the Network"
date: "2021-06-29"
github_title: "2021-06-29-13-securing-nodes-and-network"
image:
  path: /assets/img/posts/development/kubernetes/k8s-13.jpeg
attachment:
  folder: assets/img/posts/development/kubernetes
---

![k8s-13.jpeg](/assets/img/posts/development/kubernetes/k8s-13.jpeg) _A pod with hostNetwork: true uses the node's network interfaces instead of its own. (출처: https://livebook.manning.com/book/kubernetes-in-action/chapter-13)_

### 주요 내용

- 노드의 default Linux namespace 사용하기
- 컨테이너/Pod 단위로 권한 및 네트워크를 제어하여 보안 수준 높이기

컨테이너는 독립적인 환경을 제공한다고 하긴 했지만, 공격자가 API server 에 접근하게 되면 컨테이너에 무엇이든 집어넣고 악의적인 코드를 실행할 수 있고, 이는 실행 중인 다른 컨테이너에 영향을 줄 수도 있다!

## 13.1 Using the host node's namespaces in a pod

---

컨테이너는 별도의 linux namespace 에서 실행된다고 했었다.

### 13.1.1 Using the node's network namespace in a pod

시스템과 관련된 작업 (노드 레벨의 자원을 확인/수정하는 등) 을 하는 pod 의 경우 노드의 default namespace 에서 실행되어야 한다.

예를 들어, 별도의 네트워크 namespace 를 갖지 않고 (가상 네트워크 어댑터를 사용하지 않고), 호스트의 네트워크 어댑터를 사용하고 싶다면 `hostNetwork` 의 값을 `true` 로 해서 pod 를 실행하면 된다.

그러면 pod 는 노드의 네트워크 인터페이스에 접근할 수 있게 되고, pod 에는 별도의 IP 주소가 부여되지 않게 된다. Pod 내부에서 특정 포트에 bind 된 프로세스가 있다면, pod 의 포트가 곧 노드의 포트이므로 노트의 포트에 bind 되게 된다.

참고로 Kubernetes Control Plane 에 있는 컴포넌트들은 `hostNetwork` 옵션을 사용하여 pod 를 실행한다.

### 13.1.2 Binding to a host port without using the host's network namespace

위 경우에서는 노드의 네트워크 어댑터에 붙었지만, `hostPort` 값을 설정하면 노드의 특정 포트에 bind 하면서도 자신만의 네트워크 namespace 를 가질 수 있게 된다.

이렇게 했을 때 NodePort service 와의 차이점은, hostPort 의 경우 노드로 들어오는 요청을 직접 포워딩 해주는 반면, NodePort service 는 요청을 받아서 endpoint 중 임의의 (같은 노드가 아닐 수 있음) pod 로 포워딩 해준다는 점이다. 또 hostPort 의 경우 해당 노드에서만 포워딩이 일어나지만, NodePort service 의 경우 모든 노드에서 포워딩이 일어난다.

여러 프로세스가 하나의 포트에 bind 될 수 없기 때문에, Scheduler 도 이를 반영하여 scheduling 을 해준다. 만약 모든 노드의 포트가 사용 중이어서 bind 가 불가능하면 한 pod 는 pending 상태로 남아있게 된다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubia-hostport
spec:
  containers:
  - image: luksa/kubia
    name: kubia
    ports:
    - containerPort: 8080   # 컨테이너의 8080 포트를
      hostPort: 9000        # 노드의 9000번 포트와 bind
      protocol: TCP
```

이 기능은 주로 시스템과 관련된 pod 를 expose 할 때 사용한다. (DaemonSet)

### 13.1.3 Using the node's PID and IPC namespaces

호스트의 네트워크 namespace 를 사용할 수 있었던 것처럼 `hostPID`, `hostIPC` 값을 `true` 로 설정해 주면 노드의 PID 와 IPC namespace 를 사용하게 된다. `spec` 아래에 넣어주면 된다.

## 13.2 Configuring the container's security context

---

`securityContext` property 를 이용하면 보안과 관련된 기능들을 pod 과 내부 컨테이너에 설정할 수 있다.

#### Security Context

Security context 를 설정하면 다양한 것들이 가능하다.

- 컨테이너 안의 프로세스가 어떤 user 로 실행할지 명시하기
- 컨테이너가 root 로 실행되는 것을 막기
- 컨테이너가 privileged mode 로 실행되도록 하기 (노드의 커널에 접근 가능)
- 권한을 상세하게 조정하기
- 프로세스가 컨테이너의 파일시스템에 write 하는 것을 막기

#### Running a pod without specifying a security context

Security context 를 기본값으로 하고 pod 를 실행해본다.

```bash
$ kubectl run pod-with-defaults --image alpine --restart Never -- /bin/sleep 999999
```

이제 컨테이너가 실행 중인 user 와 group 을 살펴보면,

```
$ kubectl exec pod-with-defaults -- id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
```

모두 root 로 실행 중인 것을 확인할 수 있다.

### 13.2.1 Running a container as a specific user

다른 user 로 pod 를 실행하려면, `securityContext.runAsUser` property 값을 설정하면 된다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-as-user-guest
spec:
  containers:
  - name: main
    image: alpine
    command: ["/bin/sleep", "999999"]
    securityContext:
      runAsUser: 405        # guest
```

Pod 를 생성한 뒤 `id` 명령을 실행해 보면 `guest` user 로 실행된 것을 확인할 수 있다.

```
$ kubectl exec pod-as-user-guest -- id
uid=405(guest) gid=100(users)
```

### 13.2.2 Preventing a container from running as root

root 가 아닌 임의의 사용자로 실행되더라도 무관하다면, root 로 실행하지 못하게 막을 수 있다.

> Scheduler 가 새롭게 pod 를 띄울 때는 registry 에서 image 를 pull 받을 것이다. 만약 공격자가 image registry 에 접근 권한을 얻어서 같은 tag 를 가졌지만 root 로 실행하는 image 를 push 하게 되면 악의적인 목적을 가진 컨테이너가 그대로 실행될 위험이 있다.

컨테이너는 물론 호스트의 시스템과 분리되어 있지만, 프로세스를 root 권한으로 실행하는 것은 권장되지 않는다. 대표적으로 폴더를 mount 하는 경우, root 로 실행하게 되면 모든 권한을 다 갖게 된다.

root 로 실행을 막기 위해서는 `securityContext.runAsNonRoot` 를 `true` 로 설정하면 된다.

### 13.2.3 Running pods in privileged mode

어떤 경우에는 pod 가 모든 권한을 부여받아야 할 때도 있다. 예를 들어 kube-proxy pod 의 경우 노드의 `iptables` 를 변경해야 service 를 동작시킬 수 있게 된다.

이 경우 `securityContext.privileged` 의 값을 `true` 로 설정하면 된다. 그러면 노드의 커널에 모든 접근 권한을 갖게 된다.

### 13.2.4 Adding individual kernel capabilities to a container

당연히, 모든 권한을 주는 것 보다는 필요한 권한만 주는 것이 훨씬 안전할 것이다. Linux 에서는 kernel *capability* 로 권한을 관리한다.

예를 들어, 컨테이너에서는 보통 시간을 설정할 수 없다.

```
$ kubectl exec -it pod-with-defaults -- date +%T -s "12:00:00"
date: can't set date: Operation not permitted
```

만약 이 권한을 주고 싶다면 `SYS_TIME` 을 설정해 주면 된다. `securityContext.capabilities.add` 아래에 추가해준다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-add-settime-capability
spec:
  containers:
  - name: main
    image: alpine
    command: ["/bin/sleep", "999999"]
    securityContext:
      capabilities:
        add:            # 권한을 추가한다
        - SYS_TIME
```

> 시간이 변경 가능한지 확인하고 싶었으나, minikube 내부에서 NTP daemon 이 시간을 원래대로 돌려줬다. 컨테이너 내의 `sh` 에서 직접 `date +%T -s "12:00:00"; date` 를 하고 나니 변경된 것을 확인할 수 있었다.

### 13.2.5 Dropping capabilities from a container

만약 특정 권한을 빼앗고 싶다면 `securityContext.capabilities.drop` 아래에 추가해주면 된다.

### 13.2.6 Preventing processes from writing to the container's filesystem

보안상 프로세스가 컨테이너의 파일시스템보다는 mounted volume 에 write 하도록 하는 것이 좋다. 파일시스템을 read only 로 설정하려면 `securityContext.readOnlyRootFilesystem` 을 `true` 로 설정하면 된다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-readonly-filesystem
spec:
  containers:
  - name: main
    image: alpine
    command: ["/bin/sleep", "999999"]
    securityContext:
      readOnlyRootFilesystem: true      # write 는 불가능
    volumeMounts:
    - name: my-volume
      mountPath: /volume
      readOnly: false                   # volume 에는 write 가능
  volumes:
  - name: my-volume
    emptyDir:
```

Pod 를 생성해보면, root 로 실행되었음에도 `/` 디렉터리에 write 가 안된다.

#### Setting options at the pod level

지금까지는 각 컨테이너마다 security context 를 지정했지만, pod 레벨에서도 `pod.spec.securityContext` property 를 이용해 설정할 수 있다. 모든 컨테이너가 적용 대상이 되고, 컨테이너 레벨에서 또 설정하게 되면 overriding 할 수 있다.

또한 pod 레벨에서는 추가로 사용할 수 있는 보안 기능이 있다.

### 13.2.7 Sharing volumes when containers run as different users

한 pod 내에서 volume 을 사용하게 되면 컨테이너 간에 데이터를 공유할 수 있다고 했다. 이게 가능했던 이유는 컨테이너가 모두 root 로 실행되어 읽기/쓰기 권한을 모두 갖고 있었기 때문이다. 한편 `runAsUser` 옵션을 사용하게 되면 volume 을 사용했을 때 둘 다 읽기/쓰기 권한이 없을 수도 있다.

Kubernetes 에서는 supplemental groups 를 제공하여 데이터 공유를 가능하게 해준다. `fsGroup`, `supplementalGroups` 옵션을 사용하면 된다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-shared-volume-fsgroup
spec:
  securityContext:              # 이 두 옵션은 pod 레벨에서 정의된다
    fsGroup: 555
    supplementalGroups: [666, 777]
  containers:
  - name: first
    image: alpine
    command: ["/bin/sleep", "999999"]
    securityContext:
      runAsUser: 1111           # 첫 번째 컨테이너는 user ID 1111
    volumeMounts:
    - name: shared-volume
      mountPath: /volume
      readOnly: false
  - name: second
    image: alpine
    command: ["/bin/sleep", "999999"]
    securityContext:
      runAsUser: 2222           # 두 번째 컨테이너는 user ID 2222
    volumeMounts:
    - name: shared-volume
      mountPath: /volume
      readOnly: false
  volumes:
  - name: shared-volume
    emptyDir:
```

Pod 를 생성하고 `id` 명령을 실행해본다.

```
$ id
uid=1111 gid=0(root) groups=555,666,777
```

user ID 는 `1111` 이고, group ID 는 `0` (root) 이지만 `555,666,777` 도 이 사용자와 엮여있는 것을 확인할 수 있다. `fsGroup` 을 `555` 로 설정했으므로, mount 된 volume 을 소유하고 있는 group ID 는 `555` 이다.

```
$ ls -l / | grep volume
drwxrwsrwx    2 root     555           4096 Jun 13 14:59 volume
```

Volume 안에 들어가서 파일을 생성하면 파일을 소유하고 있는 user ID 는 `1111` 이고 group ID 는 `555` 가 된다.

```
$ echo foo > /volume/foo
$ ls -l /volume
total 4
-rw-r--r--    1 1111     555              4 Jun 13 15:03 foo
```

보통 사용자가 파일을 만들게 되면 effective group ID 로 설정되는데, `fsGroup` 옵션을 이용하게 되면 volume 안에 파일을 만들 때 설정할 group ID 를 지정할 수 있다.

> `supplementalGroups` 에 대한 설명이 좀 부족하다. 단순히 user 와 엮인 추가 group ID 를 설정할 수 있다고만 적혀있다.

## 13.3 Restricting the use of security-related features in pods

---

클러스터 관리자는 PodSecurityPolicy 리소스를 이용해서 pod 의 보안과 관련된 기능들을 제한할 수 있다.

### 13.3.1 Introducing the PodSecurityPolicy resource

PodSecurityPolicy 리소스는 클러스터 레벨의 리소스로, 사용자들이 pod 를 생성할 때 사용할 수 있는 보안 관련 기능을 정의하기 위해 사용한다. PodSecurityPolicy 안의 규칙(policy)은 API server 에서 실행 중인 PodSecurityPolicy admission control plugin 에서 관리된다.

사용자가 pod 생성을 요청하게 되면, PodSecurityPolicy admission control plugin 이 pod 의 정의를 보고 validation 을 해준다. 만약 pod 의 정의가 PodSecurityPolicy 에 부합하면, etcd 에 저장되고, 그렇지 않으면 생성 요청이 거절된다. 추가로, 해당 플러그인이 직접 pod 리소스 정보를 변경할 수도 있다. (기본값 세팅 등)

#### Understanding what a PodSecurityPolicy can do

다음과 같은 작업을 제어할 수 있다.

- Pod 의 호스트 IPC/PID/네트워크 namespace 를 사용 제어
- Pod 가 bind 할 수 있는 호스트의 포트 제한
- 컨테이너를 실행할 user ID 제한
- Privileged 컨테이너 실행 가능 여부
- 커널 관련 작업 제어
- 컨테이너의 root 파일시스템 쓰기 제어
- 컨테이너를 실행할 파일시스템 group 제한
- Pod 가 사용할 수 있는 volume 종류 제한

앞에서 소개한 내용과 거의 비슷하다.

#### Examining a sample PodSecurityPolicy

```yaml
apiVersion: extensions/v1beta1
kind: PodSecurityPolicy
metadata:
  name: default
spec:
  hostIPC: false
  hostPID: false
  hostNetwork: false            # 호스트의 IPC, PID, 네트워크 namespace 사용 불가
  hostPorts:
  - min: 10000
    max: 11000                  # bind 가능한 포트는 10000~11000
  - min: 13000
    max: 14000                  # 13000~14000 포트도 허용
  privileged: false             # privileged 컨테이너 실행 불가능
  readOnlyRootFilesystem: true  # root 파일시스템은 읽기 전용
  runAsUser:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  seLinux:
    rule: RunAsAny              # 실행할 user 와 group 은 제한 없음
  volumes:
  - '*'                         # 모든 종류의 volume 사용 가능
```

### 13.3.2 Understanding `runAsUser`, `fsGroup` and `supplementalGroups` policies

앞 예제에서는 `RunAsAny` 를 사용했기 때문에 제약 조건이 없었지만, 제한하고 싶다면 `MustRunAs` 를 이용해서 ID 의 범위를 제한할 수 있다.

```yaml
runAsUser:
  rule: MustRunAs
  ranges:
  - min: 2
    max: 2
```

> 참고로 PodSecurityPolicy 리소스를 업데이트 하더라도 기존에 생성된 pod 에는 영향을 주지 않는다. Pod 를 생성하거나 수정할 때만 플러그인이 확인한다.

또한 root user 로의 실행을 막고 싶을 때는 `MustRunAsNonRoot` 를 사용하면 된다.

### 13.3.3 Configuring allowed, default, and disallowed capabilities

Linux 커널과 관련된 권한을 통제하고 싶을 때 capabilities 를 조작하면 됐었다. PodSecurityPolicy 에서는 `allowedCapabilities`, `defaultAddCapabilities`, `requiredDropCapabilities` 를 이용해 권한을 통제한다.

```yaml
apiVersion: extensions/v1beta1
kind: PodSecurityPolicy
metadata:
  name: default
spec:
  allowedCapabilities:
  - SYS_TIME
  defaultAddCapabilities:
  - CHOWN
  requiredDropCapabilities:
  - SYS_ADMIN
  - SYS_MODULE
```

`allowedCapabilities` 를 사용하면 pod 의 `securityContext.capabilities` 에 어떤 값들이 포함될 수 있는지 제한할 수 있게 된다.

`defaultAddCapabilities` 를 사용하면 pod 에 해당 capability 가 자동으로 추가된다.

`requiredDropCapabilities` 를 사용하면 pod 가 어떤 capability 를 가지지 않아야 하는지 제한할 수 있다. (해당 capability 를 drop 하는 것을 require 하는 것이다)

### 13.3.4 Constraining the types of volumes pods can use

최소한 `emptyDir`, `configMap`, `secret`, `downwardAPI`, `persistentVolumeClaim` 은 사용할 수 있게 해줘야 한다.

PodSecurityPolicy 가 여러 개 있으면, 각각에서 허용한 volume 종류의 합집합이 사용 가능한 volume 종류가 된다.

### 13.3.5 Assigning different PodSecurityPolicies to different users and groups

PodSecurityPolicy 를 만들었는데 해당 policy 가 전역에 영향을 준다면 이를 사용하기 어려울 것이다. 그러므로 RBAC 를 이용해 사용자마다 어떤 policy 가 할당되어 적용되는지 관리할 수 있다.

방법은 간단하다. PodSecurityPolicy 를 필요한 만큼 만들어 두고, ClusterRole 을 만들어 PodSecurityPolicy 를 reference 하도록 하는 것이다. 이제 ClusterRoleBinding 을 이용해 사용자나 group 에게 ClusterRole 을 bind 하면 적용된다.

```bash
$ kubectl create clusterrole <CLUSTER_ROLE_NAME> --verb=use \
  --resource=podsecuritypolicies --resource-name=<POD_SECURITY_POLICY_NAME>

$ kubectl create clusterrolebinding <CLUSTER_ROLE_BINDING_NAME> \
  --clusterrole=<CLUSTER_ROLE_NAME> --group=<GROUP_NAME>
```

> `kubectl` 에서 사용자를 추가하려면 `kubectl config set-credentials <NAME> --username=<USERNAME> --password=<PASSWORD> ` 를 입력하면 된다.

> 다른 사용자의 이름으로 리소스를 생성하려면 `kubectl --user <USERNAME> create` 를 하면 된다.

## 13.4 Isolating the pod network

---

앞서 살펴본 방법들은 pod 와 컨테이너 단에서 적용되는 보안 관련 설정을 살펴봤다. 이번에는 pod 사이의 네트워크 통신 측면에서 보안을 적용하는 방법을 알아본다.

네트워크 보안을 설정하기 위해서는 클러스터에서 사용하는 networking plugin 이 이를 지원해야한다. 만약 지원한다면, NetworkPolicy 리소스를 생성하여 네트워크를 분리시킬 수 있다.

NetworkPolicy 리소스를 사용하게 되면 ingress 와 egress 규칙을 설정할 수 있어 어떤 source 에서만 트래픽을 받을지, 어떤 destination 으로만 트래픽을 보낼지 제한할 수 있다.

### 13.4.1 Enabling network isolation in a namespace

원래 한 namespace 안의 pod 로는 아무나 접근할 수 있으므로, 이것부터 변경해야 한다.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector:  # 모든 pod 가 match 된다
```

이제 NetworkPolicy 를 특정 namespace 에 생성하면 그 누구도 pod 에 접근할 수 없게 된다.

### 13.4.2 Allowing only some pods in the namespace to connect to a server pod

클라이언트의 연결을 허용하려면 어떤 pod 가 연결할 수 있는지 명시적으로(explicitly) 적어야 한다.

예를 들어 DB 를 갖고있는 pod 가 실행 중인데, 이를 사용하는 웹 서버 이외의 접근은 막으려고 한다. 이런 경우 NetworkPolicy 에서 ingress 규칙을 설정하면 된다.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: postgres-netpolicy
spec:
  podSelector:          # app=database label 이 있는 pod 에 적용되는 규칙
    matchLabels:
      app: database
  ingress:
  - from:
    - podSelector:
        matchLabels:      # app=webserver label 이 있는 pod 로부터 들어오는 트래픽을 허용
          app: webserver
    ports:
    - port: 5432          # 개방된 포트
```

이렇게 설정하면 `app=webserver` label 을 가진 pod 이외에는 DB pod 에 접속이 불가능하며, 심지어 웹 서버 조차도 5432 포트 외의 포트에는 접속할 수 없게 된다.

> 실제로는 pod 에 직접 접속하지 않고 service 를 거칠 것이다. 이 경우에도 NetworkPolicy 의 적용을 받게 된다.

### 13.4.3 Isolating the network between Kubernetes namespaces

만약 다양한 namespace 로부터 트래픽을 받고 싶다면 namespace 에 label 을 붙여서 사용할 수도 있다.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: shoppingcart-netpolicy
spec:
  podSelector:
    matchLabels:
      app: shopping-cart
  ingress:
  - from:
    - namespaceSelector:      # tenant=manning 을 가진 namespace 에서 오는 트래픽을 허용
        matchLabels:
          tenant: manning
    ports:
    - port: 80
```

### 13.4.4 Isolating using CIDR notation

Label selector 를 사용하는 대신 CIDR block 으로 제어할 수도 있다.

```yaml
ingress:
- from:
  - ipBlock:
      cidr: 192.168.1.0/24    # 해당 block 의 트래픽만 허용
```

### 13.4.5 Limiting the outbound traffic of a set of pods

앞에서는 들어오는 트래픽 (inbound/ingress) 에 대한 제한이었지만, 나가는 트래픽 (outbound/egress) 도 제어할 수 있다.

```yaml
spec:
  podSelector:
    matchLabels:
      app: webserver
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - port: 5432
```

위와 같이 하면 `app=webserver` label 을 가진 pod 는 `app=database` 의 5432 포트로만 요청을 보낼 수 있게 된다.
