---
share: true
toc: true
categories: [Development, Kubernetes]
path: "_posts/development/kubernetes"
tags: [kubernetes, sre, devops]
title: "08. Accessing Pod Metadata and Other Resources from Applications"
date: "2021-04-18"
github_title: "2021-04-18-08-accessing-pod-metadata"
image:
  path: /assets/img/posts/development/kubernetes/k8s-08.jpeg
attachment:
  folder: assets/img/posts/development/kubernetes
---

![k8s-08.jpeg](/assets/img/posts/development/kubernetes/k8s-08.jpeg) _Using the files from the default-token Secret to talk to the API server (출처: https://livebook.manning.com/book/kubernetes-in-action/chapter-8)_

### 주요 내용

- Pod 나 컨테이너의 metadata 를 전달하는 방법
- 컨테이너 안의 앱이 Kubernetes API 와 통신하고 클러스터의 리소스를 생성/수정하는 방법

## 8.1 Passing metadata through the Downward API

---

앱의 config data 는 pod 생성 전에 결정되기 때문에 환경 변수나 configMap, secret volume 을 이용해 전달할 수 있었다.

하지만 pod 가 시작 되어야 알 수 있는 정보들도 있다. (Pod IP, 실행 중인 노드의 이름, pod 자체의 이름 등) 이런 정보를 얻기 위해서 **Downward API** 가 존재하며, 이 API 를 호출하면 pod 나 실행 환경과 관련된 정보를 얻을 수 있다. 얻은 정보는 환경 변수로 전달되거나, `downwardAPI` 볼륨을 이용하면 파일로 전달된다.

### 8.1.1 사용 가능한 metadata

Downward API 를 이용하면 pod 의 metadata 를 pod 내부의 프로세스(컨테이너에도)에 전달할 수 있다.

현재 다음 정보들을 컨테이너에게 넘겨줄 수 있다.

- Pod 의 이름, IP, namespace, 노드 이름, label, annotation
- The name of the [service account](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/) the pod is running under
- 각 컨테이너의 CPU, 메모리 자원 요청과 최대 할당량

> service account 는 pod 가 API server 와 통신할 때 authentication 을 위해 활용하는 계정이다.

위 항목들 중에서 pod labels, annotation 은 volume 을 통해서만 전달될 수 있다.

### 8.1.2 환경 변수를 이용해 metadata 가져오기

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: downward
spec:
  containers:
  - name: main
    image: busybox
    command: ["sleep", "9999999"]
    resources:
      requests:
        cpu: 15m
        memory: 100Ki
      limits:
        cpu: 100m
        memory: 6Mi # 책은 4Mi 인데 limit 의 최솟값이 6Mi 여야 한다고 해서 수정
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name  # yaml 파일의 값에서 가져오기
    - name: POD_NAMESPACE
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace
    - name: POD_IP
      valueFrom:
        fieldRef:
          fieldPath: status.podIP
    - name: NODE_NAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName
    - name: SERVICE_ACCOUNT
      valueFrom:
        fieldRef:
          fieldPath: spec.serviceAccountName
    - name: CONTAINER_CPU_REQUEST_MILLICORES
      valueFrom:
        resourceFieldRef:           # CPU, 메모리의 경우 resourceFieldRef 를 사용
          resource: requests.cpu
          divisor: 1m               # divisor 를 정의해서 원하는 단위로 값을 얻을 수 있다
    - name: CONTAINER_MEMORY_LIMIT_KIBIBYTES
      valueFrom:
        resourceFieldRef:
          resource: limits.memory
          divisor: 1Ki
```

위와 같이 `.spec.containers.env` 아래에 환경 변수를 설정할 수 있다.

> `1m` 은 1 milli-core 를 의미한다. 1/1000 core. 또한 Ki 는 키비바이트로, 1024KiB = 1MiB 이다. (binary byte 라고 생각하면 된다. Prefix 간의 간격이 1000이 아닌 1024배이다.)

Pod 를 실행하고 나서 `kubectl exec downward -- env` 를 통해 환경변수를 확인할 수 있다.

> `kubectl exec [POD] [COMMAND]` 는 deprecated 되었다고 한다. `kubectl exec [POD] -- [COMMAND]` 를 사용하라고 한다.

### 8.1.3 Passing metadata through files in a downward API volume

만약 metadata 를 파일로 얻고 싶다면 `downwardAPI` volume 을 정의하고 컨테이너에 마운트해야 한다.

환경 변수와 마찬가지로 metadata field 를 명시적으로 지정해줘야 컨테이너 안의 프로세스가 사용할 수 있다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: downward
  labels:
    foo: bar
  annotations:
    key1: value1
    key2: |
      multi
      line
      value
spec:
  containers:
  - name: main
    image: busybox
    command: ["sleep", "9999999"]
    resources:
      requests:
        cpu: 15m
        memory: 100Ki
      limits:
        cpu: 100m
        memory: 4Mi
    volumeMounts:                   # downwardAPI volume 마운트하기
    - name: downward
      mountPath: /etc/downward
  volumes:
  - name: downward
    downwardAPI:
      items:                        # 설정한 경로의 파일에 필요한 정보가 저장된다
      - path: "podName"
        fieldRef:
          fieldPath: metadata.name
      - path: "podNamespace"
        fieldRef:
          fieldPath: metadata.namespace
      - path: "labels"
        fieldRef:
          fieldPath: metadata.labels
      - path: "annotations"
        fieldRef:
          fieldPath: metadata.annotations
      - path: "containerCpuRequestMilliCores"
        resourceFieldRef:
          containerName: main
          resource: requests.cpu
          divisor: 1m
      - path: "containerMemoryLimitBytes"
        resourceFieldRef:
          containerName: main           # volume 을 사용할 땐 containerName 이 들어간다
          resource: limits.memory
          divisor: 1
```

Pod 를 만들어서 마운트가 잘 되었는지 확인해 본다.

```
$ kubectl exec downward -- ls -lL /etc/downward
total 24
-rw-r--r--    1 root     root           134 Apr 17 06:31 annotations
-rw-r--r--    1 root     root             2 Apr 17 06:31 containerCpuRequestMilliCores
-rw-r--r--    1 root     root             7 Apr 17 06:31 containerMemoryLimitBytes
-rw-r--r--    1 root     root             9 Apr 17 06:31 labels
-rw-r--r--    1 root     root             8 Apr 17 06:31 podName
-rw-r--r--    1 root     root             7 Apr 17 06:31 podNamespace
```

> `-L` (`--dereference`) 옵션: when showing file information for a symbolic link, show information for the file the link references rather than for the link itself

#### label 과 annotation 은 volume 으로만 expose 가능한 이유

더불어 label 과 annotation 은 pod 이 생성되고 변경이 가능하기 때문에, 환경 변수로 expose 하게 되면 값이 변경됐을 때 업데이트할 방법이 없다. 반면 volume 을 사용하게 되면 변경시 파일은 업데이트 된다. (그러므로 환경 변수로 내보내는 것을 막아둔 듯)

#### Volume 사용시 컨테이너 이름 명시

추가로 환경 변수 때와는 달리 volume 을 사용하는 경우에는 `resourceFieldRef.containerName` 필드가 있어야 하는데, volume 은 pod 레벨에서 사용하는 리소스이므로 어떤 컨테이너의 metadata 를 가져오는 것인지 명시해야 값을 가져올 수 있다.

#### Volume 사용시 얻는 장점

같은 pod 내에서 한 컨테이너의 metadata 를 다른 컨테이너에게 보여줄 수 있다는 장점이 있다.

#### Downward API 를 사용해야할 때

Downward API 를 사용하는 것은 간단하다. Shell script 로 환경 변수를 설정하는 등의 수고로움을 덜어줄 것며, 애플리케이션이 Kubernetes 에 의존하지 않게 할 수 있다. 만약 환경 변수 값을 이용해서 동작하는 앱이라면 Downward API 가 유용할 것이다.

## 8.2 Talking to the Kubernetes API server

---

Downward API 도 다양한 정보를 제공하지만 이로는 부족할 수 있다. (다른 pod/클러스터의 정보가 필요하거나) 그렇다면 Kubernetes API server 와 직접 통신하여 원하는 값을 얻어야 한다.

### 8.2.1 Exploring the Kubernetes REST API

우선 API 서버의 URL 을 알아보려면 `kubectl cluster-info` 를 입력하면 된다.

```
$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.49.2:8443
KubeDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

API 서버는 HTTPS 를 사용하므로, 인증 없이는 직접 요청을 보낼 수 없다.

#### kubectl proxy 를 이용하여 요청하기

`kubectl proxy` 를 이용하면 로컬에서 HTTP 요청을 받아서 Kubernetes API 서버로 요청을 전달해 주고, 인증도 알아서 처리해준다. 더불어 매 요청마다 서버의 인증서를 확인하여 MITM attack 을 막고 실제 서버와 통신할 수 있도록 해준다.

```
$ kubectl proxy
Starting to serve on 127.0.0.1:8001
```

이제 `http://localhost:8001` 에 접속해보면 `paths` 가 잔뜩 있는 것을 확인할 수 있다. 각 `paths` 는 리소스를 만들때 사용했던 `apiVersion` 나 API group 에 대응된다.

#### batch API group

Job 리소스의 API group 은 `/apis/batch` 이다.

```
$ curl http://localhost:8001/apis/batch
{
  "kind": "APIGroup",
  "apiVersion": "v1",
  "name": "batch",
  "versions": [
    {
      "groupVersion": "batch/v1",
      "version": "v1"
    },
    {
      "groupVersion": "batch/v1beta1",
      "version": "v1beta1"
    }
  ],
  "preferredVersion": {
    "groupVersion": "batch/v1",
    "version": "v1"
  }
}
```

`curl` 을 해보면, API 버전이 2개가 있고, 이 중 `batch/v1` 이 선호된다는 것을 알 수 있다.

```
$ curl http://localhost:8001/apis/batch/v1
{
  "kind": "APIResourceList",
  "apiVersion": "v1",
  "groupVersion": "batch/v1",
  "resources": [
    {
      "name": "jobs",
      "singularName": "",
      "namespaced": true,
      "kind": "Job",
      "verbs": [
        "create",
        "delete",
        "deletecollection",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "categories": [
        "all"
      ],
      "storageVersionHash": "mudhfqk/qZY="
    },
    {
      "name": "jobs/status",
      "singularName": "",
      "namespaced": true,
      "kind": "Job",
      "verbs": [
        "get",
        "patch",
        "update"
      ]
    }
  ]
}
```

`batch/v1` 으로 `curl` 을 해보면 해당 API group 에 있는 리소스 종류의 목록과 REST 엔드포인트를 돌려준다. 여기에 Job 이 있다. (`resources` 안에 이 그룹의 리소스 종류가 배열의 원소로 들어있다)

자세히 살펴보면 리소스의 이름과 관련된 `name`, `kind` 가 있고, `namespaced` 에서 namespace 의 적용 여부와 `verbs` 에서 수행할 수 있는 동작을 (생성, 삭제 등) 확인할 수 있다.

#### 클러스터 내 모든 Job 인스턴스 가져오기

`/apis/batch/v1/jobs` 에 GET 요청을 보내면 될 것이다.

#### 이름으로 특정 Job 인스턴스 가져오기

만약 `default` namespace 의 `my-job` 이란 이름의 Job 리소스에 대해 정보를 확인하고 싶다면 `/apis/batch/v1/namespaces/default/jobs/my-job` 으로 GET 요청을 보내면 된다. (이거 매우 불편해보입니다;)

결과를 보면 `kubectl get job my-job -o json` 과 동일함을 확인할 수 있다.

### 8.2.2 Pod 내부에서 API 서버에 요청하기

Pod 내부에서는 `kubectl` 이 없을 것이므로, `kubectl proxy` 를 사용할 수 없다. 그러므로 API 서버와 통신하기 위해서는 다음 3가지를 고려해야 한다.

- API 서버의 URL
- API 서버의 authenticity (MITM 아닌지 확인)
- API 서버에 인증하기

#### 서버와 통신할 pod 생성

우선 테스트를 위해 아무것도 안하는 pod 를 생성하고, 안에서 shell 을 실행해본다. Pod 내부에서 `curl` 이 가능해야 하므로, `curl` binary 가 있는 이미지로 컨테이너를 만든다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: curl
spec:
  containers:
  - name: main
    image: tutum/curl
    command: ["sleep", "9999999"] # 자라...
```

```bash
$ kubectl exec -it curl bash
```

로 shell 을 실행한다.

#### API 서버의 주소 알아내기

`kubernetes` 라는 이름으로 service 가 돌아가고 있으므로, 해당 IP 로 요청을 보내면 될 것이다. Service 의 주소는 환경 변수로 들어가므로, `env` 를 이용해 확인해주면 간단하게 해결된다.

```
root@curl:/# env | grep KUBERNETES_SERVICE
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT_HTTPS=443
```

사실 이렇게 하지 않아도 되고, 내부 DNS 서버에 entry 가 생성되므로, 그냥 `https://kubernetes` 로 요청을 보내도 된다. 하지만 이 경우에는 포트 번호를 알아야하긴 하므로 환경 변수에서 직접 찾아보거나, [DNS SRV record](https://en.wikipedia.org/wiki/SRV_record) 를 찾아보면 될 것이다.

> SRV record: specification of data in the DNS defining the location of servers for specified services. 여기서 location 이라 함은 hostname 과 port number 정도로 생각해도 괜찮을 것 같다.

한 번 `https://kubernetes` 로 요청을 보내보면 인증서가 없어서 실패한다.

```
root@curl:/# curl https://kubernetes
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: http://curl.haxx.se/docs/sslcerts.html

curl performs SSL certificate verification by default, using a "bundle"
 of Certificate Authority (CA) public keys (CA certs). If the default
 bundle file isn't adequate, you can specify an alternate file
 using the --cacert option.
If this HTTPS server uses a certificate signed by a CA represented in
 the bundle, the certificate verification probably failed due to a
 problem with the certificate (it might be expired, or the name might
 not match the domain name in the URL).
If you'd like to turn off curl's verification of the certificate, use
 the -k (or --insecure) option.
```

`-k` (`--insecure`) 옵션을 사용하면 이를 무시할 수 있지만 그래도 API 서버 측에서 막는다.

```
root@curl:/# curl https://kubernetes -k
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {

  },
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
  "reason": "Forbidden",
  "details": {

  },
  "code": 403
}
```

#### 서버가 진짜인지 확인하기

`default-token-*****` 라는 Secret 이 모든 컨테이너에 생성되므로, `/var/run/secrets/kubernetes.io/serviceaccount` 에 들어가서 확인해 본다. `ca.crt`, `namespace`, `token` 이렇게 총 3개의 파일이 있는데, `ca.crt` 파일을 이용할 것이다. 이 파일에는 서버의 인증서를 sign 한 CA 의 정보가 들어있다.

`curl` 의 `--cacert` 옵션을 이용해 `ca.crt` 를 CA 인증서로 넘겨주고 다시 `curl` 을 때려본다.

```
root@curl:/# curl --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt https://kubernetes
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {

  },
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
  "reason": "Forbidden",
  "details": {

  },
  "code": 403
}
```

`-k` 를 이용해서 인증서 체크를 건너뛰었을 때와 같은 결과가 나오는 것을 확인할 수 있다. 이제 서버에 인증을 하면 될 것이다.

그 전에 매번 `--cacert` 를 입력하기 귀찮으므로 `CURL_CA_BUNDLE` 환경 변수를 `ca.crt` 파일의 경로로 설정해주면 입력하지 않고도 요청을 보낼 수 있다.

```bash
$ export CURL_CA_BUNDLE=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

#### API 서버에 인증하기

인증을 위해서는 authentication token 이 필요하다. 이 token 은 `token` 파일에 있다. 우선 환경 변수로 token 값을 빼온 뒤, `curl` 을 때려본다.

```bash
$ TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
$ curl -H "Authorization: Bearer $TOKEN" https://kubernetes
```

> Role-based access control ([RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)) 이 실행되고 있는 클러스터에서는 위 `curl` 에서 에러가 난다. 일단 테스트를 위해서 임시적으로
>
> ```bash
> kubectl create clusterrolebinding permissive-binding \
> --clusterrole=cluster-admin \
> --group=system:serviceaccounts
> ```
>
> 를 입력하여 모든 serviceaccounts 에 admin 권한을 줄 수 있다 ㅋㅋㅋ;

#### 현재 pod 의 namespace 가져오기

조금 전에 `default-token` secret 에 `ls` 했을 때 `namespace` 파일이 있었다. 해당 파일의 내용을 읽어오면 된다.

```bash
$ NS=$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace)
$ curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/namespaces/$NS/pods
```

비슷한 방법으로 다른 API 를 이용해 `PUT` 등 요청을 보내 오브젝트 정보를 가져오거나 수정할 수 있다.

#### Pod 안에서 API 서버와 통신하기 요약

1. 앱은 API 서버의 인증서가 CA 에 의해 확인되었는지 확인해야한다. CA 의 인증서는 `ca.crt` 에 있다.
2. 앱은 요청을 보낼 때 header 에 `Authorization: Bearer $TOKEN` 을 추가해야 한다.
3. `namespace` 파일을 이용해 namespace 를 확인하고 CRUD 요청을 할 수 있다.

### 8.2.3 Ambassador container 를 이용해 API 서버 통신 간소화

HTTPS 는 너무 번거로울 수도 있으므로, 요청을 처리해줄 컨테이너를 하나 만들면 되지 않을까?

#### Ambassador 패턴

컨테이너가 직접 API 서버와 통신하지 않고, ambassador 컨테이너로 요청을 보내며, ambassador 는 proxy 역할을 해준다.

Ambassador 컨테이너는 secret 을 이용해서 인증을 처리하고 proxy 역할을 해준다. 더불어 pod 내의 컨테이너들은 네트워크 인터페이스를 공유하므로 localhost 의 한 포트로 접근할 수 있다.

#### curl pod with ambassador

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: curl-with-ambassador
spec:
  containers:
  - name: main
    image: tutum/curl
    command: ["sleep", "9999999"]
  - name: ambassador
    image: luksa/kubectl-proxy:1.6.2    # proxy 를 담당할 이미지
```

Pod 를 생성한 뒤 `kubectl exec -it curl-with-ambassador -c main -- bash` 를 이용해 접근한다.

> `-c` 는 pod 내의 어느 컨테이너에서 명령을 실행할지 정하는 옵션이다.

#### Ambassador 를 이용해 API 서버에 요청하기

`kubectl proxy` 는 8001 포트에 bind 되므로 `localhost:8001` 로 요청을 보내보면 잘 작동하는 것을 확인할 수 있다.

모든 인증 과정은 ambassador 컨테이너가 담당한다.

### 8.2.4 Client 라이브러리를 이용해서 API 서버에 요청하기

간단한 요청은 ambassador 컨테이너와 HTTP client 라이브러리를 이용해서 해결할 수 있겠지만, 복잡한 요청을 하게 된다면 Kubernetes API client 라이브러리를 사용하는 것이 좋다.

#### 존재하는 라이브러리

Supported by the API Machinery special interest group (SIG)

- Golang client, Python

User contributed client libraries

- Java. Node.js, PHP, Ruby, Clojure, Scala, Perl

당연히 라이브러리들은 HTTPS 인증도 처리해준다.

---

## Discussion & Additional Topics

### Mili-core 단위로 CPU 자원 할당을 어떻게 하는 것인지?

### Ambassador 컨테이너가 `kubectl proxy` 를 실행하는게 과연 맞는 방법일지?

- 컨테이너 안에 `kubectl` 설치해야 하는 것 같던데...
- [소스 링크](https://github.com/luksa/kubernetes-in-action/tree/master/Chapter08/kubectl-proxy)
