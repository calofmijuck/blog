---
share: true
toc: true
categories: [Development, Kubernetes]
path: "_posts/development/kubernetes"
tags: [kubernetes, sre, devops]
title: "07. ConfigMaps and Secrets: Configuring Applications"
date: "2021-04-18"
github_title: "2021-04-18-07-configmaps-and-secrets"
image:
  path: /assets/img/posts/development/kubernetes/k8s-07.jpeg
attachment:
  folder: assets/img/posts/development/kubernetes
---

![k8s-07.jpeg](/assets/img/posts/development/kubernetes/k8s-07.jpeg) _Combining a ConfigMap and a Secret to run your fortune-https pod (출처: https://livebook.manning.com/book/kubernetes-in-action/chapter-7)_

거의 대부분의 앱은 설정(configuration)이 필요하다. 개발 서버, 배포 서버의 설정 사항 (접속하려는 DB 서버 주소 등)이 다를 수도 있고, 클라우드 등에 접속하기 위한 access key 가 필요하거나, 데이터를 암호화하는 encryption key 도 설정해야하는 경우가 있다. 이러한 경우에 해당 값들을 도커 이미지 자체에 넣어버리면 보안 상 취약하고, 또 설정 사항을 변경하는 경우 이미지를 다시 빌드해야하는 등 불편함이 따른다.

이번 장에서는 Kubernetes 에서 돌아가는 애플리케이션에 설정 사항을 넘겨주는 방법을 알아본다.

## 7.1 컨테이너화 된 애플리케이션 설정하기

---

보통 애플리케이션의 설정 사항을 관리할 때에는 configuration file 이 존재하게 된다. (`.properties`, `.env` 등)

그런데 Docker 를 사용하면, config file 을 컨테이너로 옮기는 명령이 Dockerfile 에 필요하게 되고, config file 을 수정하면 이미지를 다시 빌드해야하기 때문에 보통은 컨테이너에 환경 변수(environment variables)를 전달하는 방식으로 사용한다. 그리고 애플리케이션은 환경 변수를 조회하여 사용할 수 있다.

또는 6장에서 배운 volume 을 사용할 수도 있을 것이다.

애플리케이션 설정 사항을 전달하는 방법에는 크게 3가지가 있다.
- 컨테이너에 command line argument 전달하기
- 컨테이너마다 환경 변수 설정하기
- Volume 을 이용해서 config file mount 하기

## 7.2 컨테이너에 command line argument 전달하기

---

보통은 컨테이너 이미지에 정의된 기본 명령으로 이미지를 실행하지만, Kubernetes 에서는 해당 명령을 override 하여 다른 명령을 실행하도록 할 수 있다. 그래서 실행할 때 추가로 argument 를 전달할 수 있게 된다.

### 7.2.1 Defining the command and arguments in Docker

우선, 컨테이너에서 실행되는 명령은 두 부분: **command**, **arguments** 으로 나뉜다.

#### `ENTRYPOINT`, `CMD` 이해하기

Dockerfile 에서 다음 명령을 사용할 수 있다.

- `ENTRYPOINT`: 컨테이너가 시작할 때 실행할 파일
- `CMD`: `ENTRYPOINT` 를 실행할 때 전달할 argument

> **Note.** Dockerfile reference 를 보면 `CMD` 의 용법은 3가지가 있다고 한다.
> - `CMD ["executable","param1","param2"]` (*exec* form, this is the preferred form)
> - `CMD ["param1","param2"]` (as *default parameters to ENTRYPOINT*)
> - `CMD command param1 param2` (*shell* form)
> 
> Dockerfile 에는 하나의 `CMD` 만 존재할 수 있으며, **The main purpose of a CMD is to provide defaults for an executing container.** 라고 한다.
> *provide default* 라고 했기 때문에 이는 overriding 이 가능하다는 것이다.
> 
> `ENTRYPOINT` 를 사용하면 컨테이너가 실행될 때 `ENTRYPOINT` 에서 지정한 명령을 수행하고, `CMD` 도 마찬가지지만 `CMD` 의 경우 컨테이너 실행시 인자값을 주면 Dockerfile 의 `CMD` 를 override 하여 실행한다.
> 
> Reference 에서도 이 둘의 사용법을 설명해줬다.
> 
> Both `CMD` and `ENTRYPOINT` instructions define what command gets executed when running a container. There are few rules that describe their co-operation.
> - Dockerfile should specify at least one of `CMD` or `ENTRYPOINT` commands.
> - `ENTRYPOINT` should be defined when using the container as an executable.
> - `CMD` should be used as a way of defining default arguments for an `ENTRYPOINT` command or for executing an ad-hoc command in a container.
> - `CMD` will be overridden when running the container with alternative arguments.

#### shell form & exec form

`ENTRYPOINT`, `CMD` 를 사용할 때는 2가지 형태가 있다.

- `shell` form: `ENTRYPOINT node app.js`
- `exec` form: `ENTRYPOINT ["node", "app.js"]`

`shell` form 의 경우에는 컨테이너가 시작하면 shell 을 띄워서 안에서 명령을 실행한다. 반면 `exec` form 은 해당 명령이 바로 컨테이너의 프로세스가 된다.

그러므로 전자는 `shell` 프로세스를 별도로 실행하기 때문에, `exec` form 을 사용하는 것이 좋다.

#### 이미지에서 argument 설정하기

```dockerfile
ENTRYPOINT ["/bin/fortuneloop.sh"]
CMD ["10"]
```

위와 같이 설정하면 쉘 스크립트의 argument 로 `10` 을 전달할 수 있다.

또 `docker run` 할 때 override 할 수 있다.

```bash
$ docker run -it <IMAGE_NAME> 15
```

위와 같이 넘기면 `15` 를 argument 로 전달할 수 있다.

### 7.2.2 Kubernetes 에서 command, argument overriding

YAML 파일에서 override 할 수 있다.

```yaml
spec:
  containers:
  - image: some/image
    command: ["/bin/command"]
    args: ["arg1", "arg2", "arg3"]
```

`command`, `args` 필드는 컨테이너가 시작되면 수정할 수 없다.

## 7.3 컨테이너 환경 변수 설정하기

---

Pod 레벨에서 환경 변수를 설정하고 컨테이너가 이를 상속하게 하는 옵션은 존재하지 않는다.

### 7.3.1 컨데이너 정의에 환경 변수 설정하기

```yaml
spec:
  containers:
  - image: luksa/fortune:env
    env:
    - name: INTERVAL
      value: "30"
    name: html-generator
```

위와 같이 `env` 필드 안에 환경 변수의 이름과 값을 설정할 수 있다. 확인해 보면 pod 레벨에서 환경 변수를 설정하는 것이 아니라 컨테이너 별로 환경 변수를 설정해야 한다.

### 7.3.2 환경 변수 참조하기

이전에 정의된 환경 변수를 재사용할 수도 있다.

```yaml
env:
- name: FIRST_VAR
  value: "foo"
- name: SECOND_VAR
  value: "$(FIRST_VAR)bar"
```

더불어 `command`, `args` 필드의 값에서도 환경 변수를 참조하여 사용할 수 있다.

### 7.3.3 환경 변수 하드코딩의 단점

위와 같은 방법으로 하면 환경 변수를 하나씩 하드코딩하는 것인데, 그렇다면 개발용/배포용 pod 설정을 따로 해줘야 하는 불편함이 있다. Pod 설정을 재사용하지 못하는 것이다.

만약 pod 설정을 재사용하고 싶다면 config 와 pod 설정을 분리해야 하므로, 쿠버네티스에서는 ConfigMap 리소스를 제공한다.

## 7.4 ConfigMap 으로 설정 분리하기

---

앱 설정 사항을 만들 때 가장 많이 고려하는 부분은 자주 바뀌는 설정을 코드와 분리하는 것이다. (그래서 config file 도 만들고, 환경 변수도 쓰고...)

### 7.4.1 ConfigMap

ConfigMap 는 환경 변수를 key/value pair 로 저장하는 리소스이다. Value 에는 짧은 string 뿐만 아니라 config file 자체가 들어갈 수도 있다.

애플리케이션 입장에서는 ConfigMap 을 읽어올 필요도 없고 존재하는지 알 필요도 없다. 대신 ConfigMap 리소스의 정보는 volume 이 되어서 환경 변수나 파일로 컨테이너에 전달된다.

또한 pod 들이 ConfigMap 을 레퍼런스 할 때 이름으로 레퍼런스 하기 때문에, ConfigMap 을 같은 이름으로 여러 개 만들고 각각 다른 namespace 에 만들게 되면 pod YAML 파일을 하나만 만들어서 여러 namespace 에 재사용할 수 있게 된다.

(namespace 별로 이름은 같지만 다른 설정을 할 수 있다는 의미)

### 7.4.2 ConfigMap 만들기

`kubectl create configmap` 으로 만들면 된다.

#### Literal 으로부터 만들기

```bash
$ kubectl create configmap <NAME> --from-literal=<KEY>=<VALUE>
```

위 명령을 실행하면 ConfigMap 을 만들고 `<KEY>=<VALUE>` 로 설정이 생긴다. 여러 개를 만들고 싶다면 `--from-literal` 을 여러 개 입력해야한다. (불편)

만약 YAML 로 만들게 된다면 `.metadata.name` 에 ConfigMap 이름을 설정해 주고, 나머지는 `data` 에 key-value pair 로 넣어주면 된다.

#### 파일로부터 만들기

```bash
$ kubectl create configmap <NAME> --from-file=<FILENAME>
```

주의할 점은 `kubectl` 을 실행하는 경로 내에서 `<FILENAME>` (파일)을 찾는다는 점이다. 실행하게 되면 파일 이름을 key 로 해서 파일의 내용을 저장하게 된다. (key 도 임의로 설정할 수 있다)

여러 파일을 추가하는 경우에도 `--from-file` 을 여러 번 적어야 한다.

#### 디렉토리 내의 모든 파일 추가

`--from-file` 옵션을 이용할 때 디렉토리 경로를 넣어주면 된다. 대신 파일 이름이 유효한 key 값이어야 한다. (valid DNS subdomain)

#### 옵션 조합하기

ConfigMap 을 만들 때 위 옵션들을 조합해서 전부 사용할 수 있다.

### 7.4.3 ConfigMap entry 를 컨테이너의 환경 변수로 사용하기

`valueFrom` 을 사용한다.

```yaml
spec:
  containers:
  - image: luksa/fortune:env
    env:
    - name: INTERVAL        # 환경 변수를 만든다
      valueFrom: 
        configMapKeyRef:
          name: fortune-config      # fortune-config 라는 ConfigMap 에서
          key: sleep-interval       # sleep-interval 의 값을 가져온다
```

만약 한 컨테이너가 존재하지 않는 ConfigMap 을 reference 한다면 해당 컨테이너는 시작되지 않는다.

### 7.4.4 ConfigMap 의 모든 entry 를 환경 변수로 넘기기

`envFrom` 을 사용한다. (버전 1.6 이후)

```yaml
spec:
  containers:
  - image: some-image
    envFrom:
    - prefix: CONFIG_       # 모든 환경 변수가 CONFIG_ 라는 prefix 를 갖는다
      configMapRef:
        name: my-config-map
```

올바르지 않은 key 값이 있다면 무시된다.

### 7.4.5 ConfigMap entry 를 command-line argument 로 넘기기

ConfigMap 의 entry 를 직접 reference 해서 argument 로 사용할 수는 없고 (`args` 필드), 환경 변수를 생성한 뒤 해당 값을 reference 하는 방식으로 해야한다.

(생성하고 reference 하기!)

### 7.4.6 `configMap` volume 을 사용하여 ConfigMap entry 를 file 로 사용하기

보통 환경 변수나 argument 로 넘길 때에는 짧은 값을 사용하는데, (그런가...) ConfigMap 의 경우 파일 전체가 들어갈 수도 있다. 이 파일을 컨테이너가 사용할 수 있도록 하려면 `configMap` volume 을 사용해야한다.

`configMap` volume 을 사용하면 ConfigMap 의 모든 entry 가 파일로 공개된다. 컨테이너 내부의 프로세스는 해당 파일을 읽어와서 설정 값을 사용할 수 있다.

#### Volume 에 ConfigMap entry 넣기

```yaml
spec:
  containers:
  - image: nginx:alpine
    name: web-server
    volumeMounts:
    - name: config
      mountPath: /etc/nginx/conf.d
      readOnly: true
  volumes:
  - name: config
    configMap:
      name: fortune-config
```

위와 같이 해주면 `fortune-config` ConfigMap 에 있는 파일 전체를 `mountPath` 에 mount 해준다.

더불어 파일 전체가 아니라 일부만 mount 할 수도 있다. (ConfigMap 을 두개 만들어서 따로 마운트 하는 것은 불편하지 않을까)

```yaml
volumes:
  - name: config
    configMap:
      name: fortune-config
      items:
      - key: <KEY>
        path: <PATH>
```

위와 같이 하면 `<KEY>` 에 해당되는 값을 volume 의 `<PATH>` 에 저장한다. 주의할 점은 `path` 를 각각 입력해야한다는 점이다.

추가로, volume 을 사용할 때 디렉토리로 mount 하게 되면 컨테이너의 해당 디렉토리에 아무 파일도 없을 경우 문제가 되지 않지만, 파일이 존재했다면 해당 파일은 더 이상 사용할 수 없고 mount 한 볼륨의 내용으로 대체된다. 그래서 만약 `/etc` 와 같은 경로에 파일을 mount 하고 싶다면 컨테이너 자체의 `/etc` 가 날아가버리기 때문에 mount 하기 어렵다.

그래서 `subPath` 를 사용하면 volume 전체를 mount 하지 않고 한 디렉토리나 한 파일만을 mount 할 수 있다. 하지만 이렇게 하나의 파일만을 mount 하는 경우에는 파일이 업데이트 될 때 문제가 발생하는데 다음 섹션에서 알아본다.

#### `configMap` volume 의 파일 권한

Default 644. `defaultMode` 값을 수정하여 변경할 수 있다.

### 7.4.7 앱 재시작 없이 설정 업데이트하기

환경 변수나 command line argument 를 사용하면, 설정 값을 바꿀 때 앱을 재시작해야 한다. 반면 ConfigMap 을 사용하면 그럴 필요가 없어진다.

ConfigMap 을 수정하게 되면 ConfigMap 을 참조하고 있는 모든 volume 에서 파일이 수정된다. 그러면 앱이 파일의 변경을 감지하고 reload 하면 된다. (다만 오래 걸릴 수 있으니 주의)

#### ConfigMap 수정하기

`kubectl edit` 으로 수정하면 된다.

#### 파일의 업데이트는 atomic

쿠버네티스가 파일을 모두 업데이트 하기 전에 앱이 이를 감지하고 reload 하는 일은 일어나지 않는다. 쿠버네티스에서 파일의 업데이트는 atomic 하게 되어서, 업데이트가 한 번에 되고, 이는 symbolic link 를 사용해서 한다.

파일이 수정되면 수정 사항이 반영된 파일들을 만들어 한 디렉토리에 집어넣은 뒤 symbolic link 를 수정하여 해당 디렉토리 안을 참조하도록 한다.

#### 파일 하나만 mount 하면 업데이트 안 됨

(책이 집필될 시점에서는) Volume 전체가 아니라 한 파일만 mount 하는 경우 업데이트가 되지 않는다. 그래서 대안으로는 volume 전체를 다른 디렉토리에 마운트 한 뒤 필요한 위치의 파일에서 참조하도록 symbolic link 를 만들어주는 방법이 있다.

#### ConfigMap 업데이트시 주의할 점

ConfigMap 을 업데이트하는데 앱이 만약 설정이 변경된 것을 reload 하지 않는다면, pod 은 얼마든지 재시작 될 수 있으므로 일부 pod 는 수정되기 전의 설정을 그대로 가지고 있을 수도 있다.

그러므로 앱 자체에 reload 기능이 없다면, ConfigMap 을 수정하는 것은 좋은 방법이 아닐 수 있다.

만약 reload 기능이 있다면 ConfigMap 을 수정해도 괜찮으나 존재하는 instance 들 상에서 파일이 동시에 업데이트 되지 않을 수도 있으므로 잠시 sync 가 맞지 않을 수 있다는 점에 유의해야 한다.

(??? atomic 하게 된다고 했던 것 같은데...)

## 7.5 Secrets for sensitive data

---

Credential 의 경우 안전하게 전달되어야 한다.

### 7.5.1 Secrets

ConfigMap 과 비슷하다. 컨테이너에 환경 변수를 전달할 수도 있고, volume 을 통해 Secret 의 파일을 사용할 수도 있다.

쿠버네티스는 Secret 에 접근할 필요가 있는 pod 가 실행 중인 노드에서만 Secret 이 접근 가능하도록 한다. 그리고 Secret 은 메모리 상에만 존재하여 절대 물리적 저장소에 저장되지 않는다.

Secret 과 ConfigMap 중 고민이 될 때에는 다음 기준을 고려해본다.

- Non-sensitive, plain configuration data 이면 ConfigMap
- Sensitive, 특별한 관리가 필요한 정보가 존재한다면 Secret

### 7.5.2 Default token Secret

모든 pod 에는 기본적으로 `secret` volume 이 존재한다. `describe` 를 이용해서 내부 정보를 보면 `ca.crt`, `namespace`, `token` 이 있는데 이 정보를 이용하면 Kubernetes API server 에 요청을 직접 보낼 수 있다. (필요하다면)

### 7.5.3 Secret 생성

`kubectl create secret generic <NAME> --from-file=<FILE>`

설정 방법 자체는 ConfigMap 과 유사하다.

### 7.5.4 ConfigMap 과 Secret 차이

각 리소스를 생성하고 YAML output 을 비교해 보면 다르다. Secret 의 entry 들은 Base64 로 인코딩된 문자열이다. 반면 ConfigMap 의 경우 평문이 그대로 출력된다.

추가로 Secrets 의 경우에는 binary data 를 갖고 있을 수 있다. Base64 로 binary data 를 인코딩해서 넣을 수 있는 것이다.

만약 non-binary data 라면 `stringData` 필드를 이용해서 평문을 그대로 저장할 수도 있다.

Secret 을 컨테이너에서 읽게 되면 entry 는 복호화되어서 원본 값 그대로 파일에 쓰이게 된다. 환경 변수로 넘겨주는 경우에도 동일하다. 앱 자체에서 복호화 할 필요는 없다.

### 7.5.5 Pod 에서 Secret 사용하기

(생략)

`secret` volume 은 in-memory filesystem (tmpfs) 을 사용하여 Secret 의 파일들을 서장한다. 디스크에 직접 쓰지 않는다.

#### 환경 변수로 내보내기

환경 변수로 내보낼 때는 `valueFrom` 을 사용하고, ConfigMap 에서는 `configMapKeyRef` 를 사용했던 것과는 달리, `secretKeyRef` 를 사용하면 된다.

환경 변수로 내보내는 것이 가능은 하지만 추천되지는 않는다. 환경 변수는 다른 어딘가 로그나 에러 리포트에 의도치 않게 저장될 수도 있다. 또 child process 가 환경 변수를 모두 상속받게 되는데 child process 가 third-party app 이면 secret data 에 무슨 일이 생길지 알 수 없다.

Volume 사용이 추천된다.

### 7.5.6 Understanding image pull Secrets

때로 쿠버네티스 자체에서 credential 을 요구하는 경우가 있다.

책의 예시는 private image registry 에서 pull 받는 경우인데, `docker-registry` Secret 을 만들면 `.dockercfg` 파일이 생긴다 (`docker login` 명령을 실행하면 생성되는 파일과 동일) Secret 의 이름을 이용해 `.spec.imagePullSecrets.name` 에 이름을 전달해 주면 private repo 에서 pull 받을 수 있게 된다.

매번 이렇게 설정해 주기는 번거로우므로 12장에서 ServiceAccount 를 이용해 자동으로 추가하는 방법을 배운다.

---

## Discussion & Additional Topics

### What is a valid DNS subdomain?

- Alphanumeric characters, dashes, underscores and dots only. May contain optional leading dot.

아래는 쿠버네티스 공식 문서에서 가져온 내용:

#### DNS Subdomain Names

Most resource types require a name that can be used as a DNS subdomain name as defined in RFC 1123. This means the name must:

- contain no more than 253 characters
- contain only lowercase alphanumeric characters, '-' or '.'
- start with an alphanumeric character
- end with an alphanumeric character

### Resolution of key collisions when creating ConfigMaps?

- 그냥 애초에 생성이 안 되는 듯 하다.

```
$ kubectl create configmap test --from-literal=some=thing2 --from-literal=some=thing
error: cannot add key "some", another key by that name already exists in Data for ConfigMap "test"
```

### YAML 파일로 ConfigMap 을 만든다면 literal 을 제외한 다른 옵션은 어떻게 사용하는지?

- Literal 옵션의 경우 `key: value` 로 등록하면 되는 것이고,
- 파일 entry 를 넣는다면 `|` 를 이용해서 아래와 같이 하면 된다.

```yaml
data:
  some-key: |
    property1=value1
    property2=value2
    property3=value3
    property4=value4
```

### ConfigMap 에서 파일을 사용하는 경우???

- ConfigMap 에서 파일을 entry 로 넣는 경우 mount 해서 사용하는게 맞을 것 같아 보인다.
  - 안 그러면 불러왔는데 multi-line 일테니 직접 파싱 해서 사용 (??)
- Mount 를 잘 하면 `.properties` 나 `.env` 가 들어갈 위치에 config file 을 넣어줄 수 있지 않을까?
  - 그러면 애플리케이션은 해당 설정 파일을 읽으며 자연스럽게 동작!

### Mounted ConfigMaps are updated automatically

(공식 문서 발췌)

When a ConfigMap already being consumed in a volume is updated, *projected keys are eventually updated as well*. **Kubelet is checking whether the mounted ConfigMap is fresh on every periodic sync**.

However, it is using its *local ttl-based cache* for getting the current value of the ConfigMap. As a result, the total delay [from the moment when the ConfigMap is updated to the moment when new keys are projected to the pod] can be as long as kubelet sync period (1 minute by default) + ttl of ConfigMaps cache (1 minute by default) in kubelet.

You can trigger an immediate refresh by updating one of the pod's annotations.

> Note: A container using a ConfigMap as a `subPath` volume will not receive ConfigMap updates.

### Secret 종류에 generic, docker-registry 말고 뭐가 더 있나?

- [공식 문서](https://kubernetes.io/docs/concepts/configuration/secret/)에 자료가 많다!

| Builtin Type | Usage |
|--------------|-------|
| `Opaque`     |  arbitrary user-defined data |
| `kubernetes.io/service-account-token` | service account token |
| `kubernetes.io/dockercfg` | serialized `~/.dockercfg` file |
| `kubernetes.io/dockerconfigjson` | serialized `~/.docker/config.json` file |
| `kubernetes.io/basic-auth` | credentials for basic authentication |
| `kubernetes.io/ssh-auth` | credentials for SSH authentication |
| `kubernetes.io/tls` | data for a TLS client or server |
| `bootstrap.kubernetes.io/token` | bootstrap token data |

### Restrictions of Secrets

- Pod 생성 전에 Secret 를 만들어야 함
  - 없는 값을 참조하면 pod 가 시작하지 않는다
- Secret 은 namespace 를 가지므로, 같은 namespace 안에 있어야 참조할 수 있음
- 각 Secret 의 최대 용량은 1MB 이다.
  - API server, Kubelet 의 자원 낭비를 막기 위한 장치
  - 물론 작은 Secret 여러 개 만들 수 있는데 이것 까지도 통합적으로 용량 제한하는 기능은 계획 중

### More use cases of Secret

- https://kubernetes.io/docs/concepts/configuration/secret/#use-cases

### Hard link vs Symbolic link

- Hard link
  - 원본 파일과 동일한 `inode` 를 사용한다. 즉, **원본 파일을 가르키는** 링크이다. 그러므로 원본 파일이 삭제되어도 사용 가능하다.
- Symbolic link
  - 원본 파일의 **이름을 가르키는** 링크로, 원본 파일이 삭제되면 사용이 불가능하다.

### Where does ConfigMap data get stored?

- `etcd`
- https://stackoverflow.com/questions/53935597/where-does-configmap-data-gets-stored/53936061#53936061

---

후기: 이해는 되지만 잘 와닿지가 않는 느낌인데, 나중에 다시 읽어보겠습니다...
