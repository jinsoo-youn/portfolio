+++
author = "Jinsoo Youn"
title = "8&9장 - kube-api와 deployment"
date = "2022-08-01"
description = "kube-api의 REST API와 deployment"
tags = [
"kubernetes",
"study",
]
categories = [
"kubernetes",
"study",
]
series = ["Kubernetes-In-Action"]
aliases = ["Kubernetes-In-Action"]
image = "cover.png"
+++

쿠버네티스 api 서버의 rest API 명세와 deployment를 알아보자
<!--more-->

<aside>
📂 [https://velog.io/@niyu/k8s-access-api-server](https://velog.io/@niyu/k8s-access-api-server)
[https://velog.io/@niyu/k8s-deployment](https://velog.io/@niyu/k8s-deployment)

</aside>

어플리케이션이 다른 파드와 클러스터에 정의된 다른 리소스에 대해 알아야할 경우에는 API 서버와 직접 통신해야 한다.

컨테이너 내부에서 실행되는 어플리케이션이 쿠버네티스 API 서버와 통신해 클러스터에 배포된 리소스 정보를 얻는 방법에 대해 살펴보자.

# 로컬에서 API 서버와 통신

`kubectl proxy`는 로컬 시스템에서 HTTP 연결을 허용하고 k8s API 서버와 통신한다. 요청할 때마다 인증 토큰을 전달할 필요가 없다.

```
# 로컬 k8s 프록시 실행
$ kubectl proxy

Starting to serve on 127.0.0.1:8001
```

```
# 로컬 포트 8001에서 연결 수락
$ curl localhost:8001

{
  "paths": [
    "/api",
    "/api/v1",
    "/apis",
    "/apis/",
    "/apis/apps",
    "/apis/apps/v1",
    "/apis/batch",
    "/apis/batch/v1",
    "/apis/batch/v1beta1",
    ...
```

반환한 결과의 `paths`는 파드, 서비스 등과 같은 리소스를 생성할 때 리소스 정의에 지정한 API 그룹 및 버전에 해당한다. `/apis/batch/v1` 경로의 `batch/v1`은 잡(Job) 리소스의 버전으로 인식할 수 있다.

**🔎 리소스 API 탐색 (1)**

```
# 잡 리소스 API 탐색
$ curl <http://localhost:8001/apis/batch>

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

batch API 그룹에는 `v1`와 `v1beta1` 두 가지 버전이 있는 것을 확인할 수 있다. `preferredVersion` 항목을 통해 v1 버전을 권장하는 것을 알 수 있다.

**🔎 리소스 API 탐색 (2)**

```
$ curl <http://localhost:8001/apis/batch/v1>

{
  "kind": "APIResourceList",
  "apiVersion": "v1",
  "groupVersion": "batch/v1",
  "resources": [
    {
      "name": "cronjobs",
      "singularName": "",
      "namespaced": true,
      "kind": "CronJob",
      "verbs": ["create",  "delete","deletecollection","get","list","patch","update","watch"],
      "shortNames": ["cj"],
      "categories": ["all"],
      "storageVersionHash": "sd5LIXh4Fjs="
    },
    ...
    {
      "name": "jobs",
      "singularName": "",
      "namespaced": true,
      "kind": "Job",
      "verbs": ["create","delete","deletecollection","get","list","patch","update","watch"],
      "categories": ["all"],
      "storageVersionHash": "mudhfqk/qZY="
    },
    ...
  ]
}
```

반환된 목록은 API 서버에 공개된 REST 리소스를 설명한다. 리소스의 `name`과 `kind` 뿐만 아니라 리소스가 `namespace`인지 아닌지, 함께 사용할 수 있는 `verbs` 목록이 포함된다. "name: jobs"는 API에 "/apis/batch/v1/jobs" 엔드포인트가 있음을 알려준다. verbs 배열은 해당 엔드포인트를 통해 작업 리소스를 생성, 검색, 업데이트, 삭제할 수 있다고 말해준다.

**🔎 클러스터에 있는 모든 잡 인스턴스 목록 탐색**

클러스터 잡 리소스 [job.yaml](https://raw.githubusercontent.com/kubernetes/website/main/content/ko/examples/controllers/job.yaml)를 배포하고 클러스터에 있는 모든 잡 인스턴스 목록을 조회한다.

```
$ curl <http://localhost:8001/apis/batch/v1/jobs>

{
  "kind": "JobList",
  "apiVersion": "batch/v1",
  "metadata": {
    "resourceVersion": "4536"
  },
  "items": [
    {
      "metadata": {
        "name": "pi",
        "namespace": "default",
        ...
}
```

**🔎 이름으로 특정한 잡 인스턴스 탐색**

하나의 특정한 잡을 조회하려면 URL에 이름과 네임스페이스를 지정해야 한다.

```
$ curl <http://localhost:8001/apis/batch/v1/namespaces/default/jobs/pi>

{
  "kind": "Job",
  "apiVersion": "batch/v1",
  "metadata": {
    "name": "pi",
    "namespace": "default",
  ...
```

# 파드 내에서 API 서버 통신

프록시를 사용하지 않고 파드 내부에서 API 서버와 통신하려면, API 서버의 위치를 알아야하고 서버로 인증을 해야 한다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: curl
spec:
  containers:
    - name: main
      image: curlimages/curl
      command: ["sleep",  "9999999"]
```

오직 컨테이너에서 sleep 명령을 실행하는 `파드`를 생성한다.

```
# kubectl exec로 컨테이너에서 쉘을 실행
$ kubectl exec -it curl /bin/sh

/ $
```

curl를 사용해 해당 쉘 내에서 API 서버에 접근을 시도한다.

k8s API 서버의 IP와 포트를 찾아야 한다. k8s라는 서비스는 자동으로 namepsace에 노출돼 API 서버를 가리키도록 구성돼 있다.

```
# 서비스 조회
$  kubectl get svc

NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   92m
```

API 서버가 HTTPS의 기본 포트인 포트 443에서 수신 중임을 알 수 있다.

```
# API 서버에 접속
/ $ curl <https://kubernetes>

curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: <https://curl.se/docs/sslcerts.html>
```

API 서버에 접속 시 인증이 없어서 에러가 나는 것을 알 수 있다.

모든 파드에는 자동으로 연결된 `secret` 볼륨이 있다. default-token 시크릿은 각 컨테이너의  /var/run/secrets/kubernetes.io/serviceaccount/에 마운트된다.

```
# 해당 디렉토리에 파일을 나열해 시크릿 내용 조회
/ $ ls /var/run/secrets/kubernetes.io/serviceaccount/

ca.crt     namespace  token
```

시크릿에는 `ca.crt`, `namespace`, `token` 세 개의 항목이 있다. `ca.crt` 파일은 k8s API 서버 인증서에 서명하는데 사용된 인증기관(CA)의 인증서를 보유하고 있다. `token` 파일은 서버에 인증하는데 이용된다. `namespace` 파일은 파드가 실행중인 네임스페이스가 포함돼 있다.

```
# 내부 API 서버 호스트 이름을 가리킨다
APISERVER=https://kubernetes

# 서비스어카운트(ServiceAccount) 토큰 경로
SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount

# 이 파드의 네임스페이스를 읽는다
NAMESPACE=$(cat ${SERVICEACCOUNT}/namespace)

# 서비스어카운트 베어러 토큰을 읽는다
TOKEN=$(cat ${SERVICEACCOUNT}/token)

# 내부 인증 기관(CA)을 참조
CACERT=${SERVICEACCOUNT}/ca.crt

# --cacert 옵션을 통해 인증서 지정 및 Authorization 헤더에 토큰 지정하여 API를 탐색
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/apis/batch/v1

{
  "kind": "APIResourceList",
  "apiVersion": "v1",
  "groupVersion": "batch/v1",
  "resources": [
    {
      "name": "cronjobs",
      "singularName": "",
      "namespaced": true,
      "kind": "CronJob",
   ...

# 파드 탐색
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1/namespaces/$NAMESPACE/pods

{
  "kind": "PodList",
  "apiVersion": "v1",
...
```

요청의 Authorization HTTP 헤더 안에 토큰을 전달한다. API 서버는 토큰을 인증된 것으로 인식하고 적절한 응답을 반환한다.

![https://velog.velcdn.com/images/niyu/post/bf00823e-0bd4-4f30-9ace-9b636fccace7/image.png](https://velog.velcdn.com/images/niyu/post/bf00823e-0bd4-4f30-9ace-9b636fccace7/image.png)

> API 서버의 인증서가 ca.crt 파일에 있는 certificate 기관에 의해 서명됐는지 여부를 확인한다.
>
- 어플리케이션은 `token` 파일에서 무기명 토큰과 함께 Authorization 헤더를 보내 자신을 인증한다.
- `namespace` 파일은 파드의 네임스페이스 안에 있는 API 객체에 대해 CRUD 작업을 수행할 때 네임스페이스를 API 서버로 전달하는 데 사용한다.

## 클라이언트 라이브러리를 사용해 API 서버와 통신

파드에서 k8s API를 사용하는 가장 쉬운 방법은 클라이언트 라이브러리 중 하나를 사용하는 것이다. 이 라이브러리들은 API 서버를 자동으로 감지하고 인증할 수 있다.

**공식 클라이언트 라이브러리**

- [Golang Client](https://github.com/kubernetes/client-go/)
- [Python Client](https://github.com/kubernetes-client/python/)

**사용자 제공 클라이언트 라이브러리**

참고 : [https://kubernetes.io/ko/docs/reference/using-api/client-libraries/](https://kubernetes.io/ko/docs/reference/using-api/client-libraries/)

# Swagger (스웨거)

스웨거 API 프레임워크는 Web API 문서화를 위한 도구이다. k8s API 서버는 /swaggerapi 에서 스웨거 API 정의를 공개하고, /swagger.json에서  OpenAPI 스펙을 제공한다.

스웨거 UI를 통해 REST API를 탐색할 수 있다. API 서버를 `--enable-swagger-ui=true` 옵션으로 실행하면 이 기능을 활성화할 수 있다.

UI를 활성화한 후에는 웹 브라우저에서 `https://<api server>:<port>/swagger-ui` url을 통해 확인할 수 있다.

![https://velog.velcdn.com/images/niyu/post/3e29031f-d3c6-4d5b-8794-ba6824cae336/image.png](https://velog.velcdn.com/images/niyu/post/3e29031f-d3c6-4d5b-8794-ba6824cae336/image.png)

![https://velog.velcdn.com/images/niyu/post/a1947dc7-0c3a-47ca-bd9d-68d3b673daa3/image.png](https://velog.velcdn.com/images/niyu/post/a1947dc7-0c3a-47ca-bd9d-68d3b673daa3/image.png)

**References**

- [책 | Kubernetes in Action 8장 애플리케이션에서 파드 메타 데이터와 그 외의 리소스에 접근하기](http://www.yes24.com/Product/Goods/89607047)
- [Kubernetes | 파드 내에서 쿠버네티스 API에 접근](https://kubernetes.io/ko/docs/tasks/run-application/access-api-from-pod/)

---

파드가 어플리케이션의 첫 번째 버전을 실행하고 이 이미지의 태그가 v1이라고 가정할 때, 이후에 새로운 버전의 어플리케이션을 개발해 v2로 태그가 지정된 새 이미지를 이미지 스토리지에 푸시한다면 모든 파드를 새로운 버전으로 바꾸는 것이 좋다.

이때 파드를 생성한 후에는 해당 파드를 만들 때 사용했던 기존 파드의 이미지를 이전 파드를 제거하고 새로운 이미지를 실행하는 새로운 파드로 교체해야 한다.

파드에서 실행 중인 어플리케이션을 어떻게 업데이트할까? 🧐

# 파드에서 실행 중인 어플리케이션 업데이트

![https://velog.velcdn.com/images/niyu/post/14c9f7df-8830-4b7f-8b90-98c02fa4c03d/image.png](https://velog.velcdn.com/images/niyu/post/14c9f7df-8830-4b7f-8b90-98c02fa4c03d/image.png)

k8s에서 실행되는 어플리케이션 기본 구성은 위와 같다. 클라이언트가 파드에 액세스하기 위해 서비스가 존재하고, 레플리카셋이 파드를 관리한다.

## 오래된 파드를 삭제하고 새로운 파드로 교체

기존의 모든 파드를 먼저 삭제한 후 새로운 파드를 시작한다.

![https://velog.velcdn.com/images/niyu/post/1065a7d1-cb8f-42e1-ae06-10de2fa11941/image.png](https://velog.velcdn.com/images/niyu/post/1065a7d1-cb8f-42e1-ae06-10de2fa11941/image.png)

레플리케이션컨트롤러의 파드 템플릿은 언제든지 업데이트될 수 있다. 레플리케이션컨트롤러가 새 인스턴스를 만들면 업데이트된 파드 템플릿을 사용해 인스턴스를 만든다.

**버전 v1의 파드 세트를 관리하는 레플리케이션컨트롤러가 있는 경우 버전 v2 이미지를 참조하도록 파드 템플릿을 수정한 후 이전 파드 인스턴스를 삭제한다.** 레플리케이션컨트롤러는 해당 라벨 셀렉터와 일치하는 파드를 발견하지 못하고 새로운 인스턴스를 스핀업시킨다.

하지만 이 방법을 사용하면, **어플리케이션을 사용할 수 없는 짧은 시간이 발생한다.** 오래된 파드를 새로운 파드로 변경하는 데 있어서 잠깐의 다운타임을 허용할 수 있는 경우라면 이 방법을 통해 어플리케이션을 업데이트할 수 있다.

## 새 파드의 기동과 오래된 파드 삭제

새로운 것을 시작하고 일단 끝나면 오래된 것을 지운다.

어떤 다운타임도 발생하지 않지만, 짧은 시간 동안 두 배의 파드를 실행해야 하기 때문에 더 많은 하드웨어 리소스가 필요하다. 또한 두 가지 버전의 어플리케이션 실행을 동시에 처리해야 한다.

### 한 번에 기존 버전에서 새로운 버전으로 전환

새 파드를 추가하고 한 번에 기존 파드를 모두 삭제한다.

![https://velog.velcdn.com/images/niyu/post/c5f63803-e4fa-41ae-96a1-07102374516f/image.png](https://velog.velcdn.com/images/niyu/post/c5f63803-e4fa-41ae-96a1-07102374516f/image.png)

새 버전을 실행하는 파드를 가져오는 동안 초기 버전의 파드만 서비스와 연결된다. 그런 다음 **새 파드가 모두 올라오면 서비스의 라벨 셀렉터를 변경해 서비스를 새 파드로 전환한다.** 이를 `블루그린(blue-green) 배포` 라고 한다. 이전 버전을 blue 환경으로, 새 버전은 green 환경으로 부른다. 새 버전이 올바르게 동작하는지 확인하고 이전 레플리케이션컨트롤러를 삭제하면 기존 파드를 자유롭게 삭제할 수 있다.

### 롤링 업데이트 (Rolling Update)

새 파드를 순차적으로 추가하고 이전 파드를 점차 제거한다. 뒤에 나오는 디플로이먼트의 기본 전략이다.

![https://velog.velcdn.com/images/niyu/post/abaecff4-559b-4a1f-8ed1-6badd37918b4/image.png](https://velog.velcdn.com/images/niyu/post/abaecff4-559b-4a1f-8ed1-6badd37918b4/image.png)

**이전의 레플리케이션컨트롤러를 천천히 축소하고 새 레플리케이션컨트롤러를 확장한다.** 서비스의 파드 셀렉터에 이전 파드와 새 파드를 모두 포함시키고 이 두 요청의 집합으로 요청을 직접 전달한다.

# 디플로이먼트

디플로이먼트는 상태가 없는(stateless) 없는 앱을 배포할 때 사용하는 가장 기본적인 컨트롤러다.

**🔎 컨트롤러**: 기본 오브젝트를 생성하고 이를 관리하는 역할로, 대표적으로 ReplicaSet, DeamonSet, StatefulSet, Job, Deployment 등이 있다.

![https://velog.velcdn.com/images/niyu/post/47396bb2-14f7-4aa1-b037-551b26b7000d/image.png](https://velog.velcdn.com/images/niyu/post/47396bb2-14f7-4aa1-b037-551b26b7000d/image.png)

디플로이먼트를 만들면 레플리카셋 리소스가 아래에 만들어진다. 실제 파드는 디플로이먼트가 아니라 레플리카셋에 의해 생성되고 관리된다.

## 디플로이먼트 생성

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubia
spec:
  replicas: 3  # 레플리카 수 설정
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

```
# 디플로이먼트 생성
$ kubectl create -f kubia-deployment-v1.yaml --record

deployment.apps/kubia created
```

- `-record` 옵션은 디플로이먼트에 대한 변경 이력을 기록한다. 나중에 필요한 경우 리비전 번호를 사용해 롤백할 수 있다.

## 디플로이먼트 상태 확인

```
# 디플로이먼트의 롤링 업데이트 상태 감시
$ kubectl rollout status deployment kubia

deployment "kubia" successfully rolled out
```

```
# 디플로이먼트 변경 이력 출력
$ kubectl rollout history deployment kubia

deployment.apps/kubia
REVISION  CHANGE-CAUSE
1         kubectl create --filename=kubia-deployment-v1.yaml --record=true
```

```
# 파드 목록 보기
$ kubectl get po

NAME                    READY   STATUS    RESTARTS      AGE
kubia-6459db4dc-4rxg2   1/1     Running   0             2m27s
kubia-6459db4dc-8snvt   1/1     Running   0             2m27s
kubia-6459db4dc-j2f9w   1/1     Running   0             2m27s
```

디플로이먼트에서 만든 3개의 파드에는 이름 중간에 숫자 값이 추가로 포함됐다. 이 숫자는 디플로이먼트와 이 파드를 관리하는 레플리카셋의 `파드 템플릿의 해시 값`이다. 디플로이먼트는 파드를 직접 관리하지 않고 그 대신 레플리카셋을 생성하고 레플리카셋이 관리를 하도록 위임한다.

```
# 레플리카셋 보기
$ kubectl get replicasets

NAME              DESIRED   CURRENT   READY   AGE
kubia-6459db4dc   3         3         3       2m56s
...
```

레플리카셋의 이름도 해당 파드 템플릿의 해시 값을 포함한다. 디플로이먼트는 여러 버전의 레플리카셋을 만든다. 파드 템플릿의 각 버전마다 레플리카셋이 생성된다. **파드 템플릿의 해시 값을 사용하면 디플로이먼트에서 항상 파드 템플릿의 주어진 버전에 동일한 레플리카셋을 사용할 수 있게 된다.**

```
# 디플로이먼트 상태 확인
$ kubectl get deploy -o yaml
```

```yaml
apiVersion: v1
items:
- apiVersion: apps/v1
  kind: Deployment
  metadata:
    annotations:
      deployment.kubernetes.io/revision: "1"
      kubernetes.io/change-cause: kubectl create --filename=kubia-deployment-v1.yaml
        --record=true
    creationTimestamp: "2022-07-28T02:56:40Z"
    generation: 2
    name: kubia
    namespace: default
    resourceVersion: "194399"
    uid: 71ffad1d-32a5-4fae-a30e-4322007aeea5
  spec:
    minReadySeconds: 10
    progressDeadlineSeconds: 600
    replicas: 3
    revisionHistoryLimit: 10
    selector:
      matchLabels:
        app: kubia
    strategy:
      rollingUpdate:
        maxSurge: 25%
        maxUnavailable: 25%
      type: RollingUpdate
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: kubia
        name: kubia
      spec:
        containers:
        - image: luksa/kubia:v1
          name: nodejs
...
```

## 디플로이먼트 업데이트

업데이트를 위해 필요한 일은 디플로이먼트 리소스에 정의된 파드 템플릿을 수정하는 것 뿐이다.

```
# 이미지 변경
$ kubectl set image deployment kubia nodejs=luksa/kubia:v2

deployment.apps/kubia image updated
```

```
# 디플로이먼트 상태 확인
$ kubectl get deploy -o yaml
```

```yaml
apiVersion: v1
items:
- apiVersion: apps/v1
  kind: Deployment
  metadata:
    annotations:
      deployment.kubernetes.io/revision: "2"
      kubernetes.io/change-cause: kubectl set image deployment kubia nodejs=luksa/kubia:v2
    creationTimestamp: "2022-07-28T02:56:40Z"
    generation: 3
    name: kubia
    namespace: default
    resourceVersion: "194842"
    uid: 71ffad1d-32a5-4fae-a30e-4322007aeea5
  spec:
    minReadySeconds: 10
    progressDeadlineSeconds: 600
    replicas: 3
    revisionHistoryLimit: 10
    selector:
      matchLabels:
        app: kubia
    strategy:
      rollingUpdate:
        maxSurge: 25%
        maxUnavailable: 25%
      type: RollingUpdate
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: kubia
        name: kubia
      spec:
        containers:
        - image: luksa/kubia:v2
          name: nodejs
...
```

![https://velog.velcdn.com/images/niyu/post/0bcce257-e44b-4baf-a414-eb00011cf507/image.png](https://velog.velcdn.com/images/niyu/post/0bcce257-e44b-4baf-a414-eb00011cf507/image.png)

kubia 디플로이먼트의 파드 템플릿이 업데이트돼 nodejs 컨테이너에서 사용하고 있는 이미지가 v1에서 luksa/kubia:v2로 변경된다.

![https://velog.velcdn.com/images/niyu/post/9922ef4a-f16f-4e53-b318-143931003060/image.png](https://velog.velcdn.com/images/niyu/post/9922ef4a-f16f-4e53-b318-143931003060/image.png)

추가 레플리카셋이 생성되고 이전 레플리카셋이 0으로 축소되는 동안 느리게 확장된다.

```
# 레플리카셋 출력
$ kubectl get rs

NAME               DESIRED   CURRENT   READY   AGE
kubia-6459db4dc    0         0         0       75m
kubia-75c4ff7786   3         3         3       11m
```

레플리카셋의 리스트를 보면 이전 레플리카셋을 여전히 볼 수 있다.

## 디플로이먼트 롤백

버전 3에서는 어플리케이션 (참고: [app.js](https://github.com/luksa/kubernetes-in-action/blob/master/Chapter09/v3/app.js)) 이 처음의 네 개의 요청만 제대로 처리하고 다섯 번째 이후의 모든 요청은 내부 서버 오류를 반환한다.

```
# 이미지 변경
$ kubectl set image deployment kubia nodejs=luksa/kubia:v3

deployment.apps/kubia image updated
```

### 롤아웃 되돌리기

잘못된 롤아웃을 되돌리자. 디플로이먼트를 사용하면 k8s에게 디플로이먼트의 마지막 롤아웃을 취소하도록 해서 이전 버전으로 쉽게 롤백할 수 있다.

```
# 이전 버전으로 롤백
$ kubectl rollout undo deployment kubia

deployment.apps/kubia rolled back
```

**🧩 디플로이먼트의 롤아웃 히스토리 보여주기**

디플로이먼트는 리비전 히스토리를 유지하기 때문에 롤아웃 롤백이 가능하다. **히스토리는 레플리카셋의 내부에 저장된다.** 롤아웃이 완료되면 이전 레플리카셋은 삭제되지 않고 이전 레플리카셋은 이전 버전뿐만 아니라 모든 리비전으로 롤백할 수 있다.

```
$ kubectl rollout history deployment kubia

deployment.apps/kubia
REVISION  CHANGE-CAUSE
2         kubectl set image deployment kubia nodejs=luksa/kubia:v2
3         kubectl set image deployment kubia nodejs=luksa/kubia:v3
```

**🧩 특정 디플로이먼트 리비전으로 롤백하기**

```
# 첫 번째 버전으로 롤백
$ kubectl rollout undo deployment kubia --to-revision=1

deployment.apps/kubia rolled back
```

![https://velog.velcdn.com/images/niyu/post/2c99620f-5326-4c15-8b84-dfd62289077d/image.png](https://velog.velcdn.com/images/niyu/post/2c99620f-5326-4c15-8b84-dfd62289077d/image.png)

처음 디플로이먼트를 수정했을 때 비활성 레플리카셋이 디플로이먼트의 첫 번째 리비전을 나타낸다. **디플로이먼트에 의해 생성된 모든 레플리카셋은 전체 리비전 히스토리를 나타낸다.** 레플리카셋은 해당 특정 리비전의 디플로이먼트의 전체 정보를 저장하기 때문에 수동으로 삭제하면 안된다. 삭제하면 디플로이먼트 히스토리에서 특정 리비전을 잃어버리게 되고 롤백할 수 없다.

### 롤아웃 속도 통제

새 파드가 생성되고 이전 파드가 삭제되는 방식은 롤링 업데이트 전략의 `maxSurge`와 `maxUnavailable` 속성을 통해 구성할 수 있다. 두 속성은 **롤링 업데이트 중 한 번에 대체되는 파드의 수에 영향을 미친다.**

- `maxSurge` : **레플리카 수보다 얼마나 많은 파드 인스턴스 수를 허용할지를 나타낸다.** 기본값은 25%이다. 레플리카 수가 4로 설정되면 4의 25%인 1개만큼 더 허용될 수 있다. 즉, 5개의 파드까지 업데이트 중에 동시에 실행된다.
- `maxUnavailable` : **업데이트 중 레플리카 수를 기준으로 사용할 수 없는 파드 인스턴스 수를 나타낸다.** 기본값은 25%이다. 레플리카 수가 4인 경우 백분율이 25%이면 하나의 파드만 사용할 수 없다. 즉, 업데이트 과정에서 사용할 수 없는 파드는 최대 1개여야 하며 최소 3개의 파드는 항상 가용한 상태를 유지하면서 롤링 업데이트가 수행되어야 한다는 의미이다.
- * extensions/v1beta1 버전은 두 속성의 기본값을 25% 대신 1로 설정한다.

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: kubia
spec:
  replicas: 3
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
```

레플리카 수가 3이고 maxSurge가 1이고 maxUnavailable을 0으로 설정했다.

![https://velog.velcdn.com/images/niyu/post/18ca865e-ed63-499c-a35a-b82bf3e6b3dd/image.png](https://velog.velcdn.com/images/niyu/post/18ca865e-ed63-499c-a35a-b82bf3e6b3dd/image.png)

`maxSurge`가 1이기 때문에 모든 파드 수는 4개가 되는 것까지 허용했으며, `maxUnavailable`은 0이기 때문에 available한 파드는 레플리카 수와 동일한 3개여야 한다. 항상 3개의 파드를 사용할 수 있어야 한다.

위 예제에서 `maxUnavailable`만 1로 수정해보자.

![https://velog.velcdn.com/images/niyu/post/2488fd56-3177-4303-be9a-655c14ac516e/image.png](https://velog.velcdn.com/images/niyu/post/2488fd56-3177-4303-be9a-655c14ac516e/image.png)

`maxUnavailable`가 1로 설정되어 1개의 복제본을 사용할 수 없기 때문에 원하는 레플리카 수가 3개인 경우 2개만 사용할 수 있어야 한다. 롤아웃 프로세스는 즉시 하나의 파드를 삭제하고 2개의 파드를 새로 만든다. 2개의 파드를 사용할 수 있게 되면 나머지 2개의 오래된 파드는 삭제된다.

### 롤아웃 프로세스 일시 중지

롤아웃 프로세스 중에 일시 중지해서 나머지 롤아웃을 진행하기 전에 새 버전이 제대로 작동하는지 확인할 수 있다.

```
# 이미지 변경
$ kubectl set image deploy kubia nodejs=luksa/kubia:v4

deployment.apps/kubia image updated

# 롤아웃 일시 정지
$ kubectl rollout pause deploy kubia

deployment.apps/kubia paused

# 파드 확인
$ kubectl get po

NAME                    READY   STATUS    RESTARTS        AGE
kubia-6459db4dc-5x4tm   1/1     Running   0               28m
kubia-6459db4dc-9476w   1/1     Running   0               27m
kubia-6459db4dc-z297h   1/1     Running   0               27m
kubia-f797964fb-pqh6z   1/1     Running   0               26s
```

파드 목록을 보면 v4로 구동된 파드 (kubia-f797964fb-pqh6z) 1개가 추가로 구동 중인것을 확인할 수 있다. 새 파드가 가동되면 서비스에 대한 모든 요청의 일부가 새 파드로 리다이렉션된다.

**🔎 카나리 배포**
롤아웃 프로세스를 일시정지해서 일종의 카나리 배포를 실행할 수 있다.

![https://velog.velcdn.com/images/niyu/post/ca7629a3-d2dc-4b0c-b57a-21d2c0b3dac7/image.png](https://velog.velcdn.com/images/niyu/post/ca7629a3-d2dc-4b0c-b57a-21d2c0b3dac7/image.png)

카나리 배포는 잘못된 버전의 어플리케이션을 롤아웃하고 모든 사용자에게 영향을 미칠 위험을 최소화하기 위한 기술이다. 새 버전을 모든 사람에게 롤아웃하는 대신, **한 개 또는 소수의 이전 포드를 새 파드로 바꾼다.** 이렇게 하면 **소수의 사용자만 초기에 새 버전을 사용하게 된다.** 그런 다음 새 버전이 제대로 작동하는지 확인한 후 나머지 모든 파드를 통해 롤아웃을 계속하거나 이전 버전으로 롤백할 수 있다.

### 롤아웃 재개

롤아웃 프로세스를 일시 중지하면 클라이언트 요청 중 일부만 v4 파드로 전달되고 대부분은 여전히 v3 파드로 전달된다. 새 버전이 제대로 작동하면 디플로이먼트를 다시 시작해 모든 이전 파드를 새 파드로 교체할 수 있다.

```
# 롤아웃 재개
$ kubectl rollout resume deploy kubia

deployment.apps/kubia resumed

# 파드 확인
$ kubectl get po

NAME                    READY   STATUS        RESTARTS        AGE
kubia-f797964fb-hfd29   1/1     Running       0               24s
kubia-f797964fb-pqh6z   1/1     Running       0               7m11s
kubia-f797964fb-sw78t   1/1     Running       0               35s
```

### 잘못된 버전의 롤아웃 방지

`minReadySeconds` 속성으로 롤아웃 속도를 늦춰 롤링 업데이트 과정을 직접 볼 수 있는데, 이 속성의 주요 기능은 오작동하는 버전의 배포를 방지하는 것이다.

**`minReadySeconds`는 파드를 사용 가능한 것으로 취급하기 전에 새로 만든 파드를 준비할 시간을 지정하는 속성이다.** 파드를 사용할 수 있게 될 때까지는 롤아웃 프로세스가 진행되지 않는다.

이것과 `레디니스 프로브`를 함께 이용하여 오작동 버전의 롤아웃을 효과적으로 차단할 수 있다. 모든 파드의 레디니스 프로브가 성공하면 파드가 준비상태가 되는데, `minReadySeconds` 에 설정된 시간이 지나가기 전에 레디니스 프로브가 실패하기 시작하면 새 버전의 롤아웃은 차단이 된다.

**🧩 적절한 minReadySeconds와 레디니스 프로브 정의**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubia
spec:
  replicas: 3
  minReadySeconds: 10  # minReadySeconds를 10초로 설정
  selector:
    matchLabels:
      app: kubia
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0  # 파드를 하나씩만 교체하도록 설정
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
        readinessProbe:  # 레디네스 프로브 정의
          periodSeconds: 1  # 매초마다 레디네스 프로브 수행
          httpGet:  # 컨테이너에 HTTP GET 요청 수행
            path: /
            port: 8080
```

```
# 디플로이먼트 업데이트
$ kubectl apply -f kubia-deployment-v3-with-readinesscheck.yaml

deployment.apps/kubia configured
```

```
# 롤아웃 상태 확인
$ kubectl rollout status deploy kubia

Waiting for deployment "kubia" rollout to finish: 1 out of 3 new replicas have been updated...
error: deployment "kubia" exceeded its progress deadline
```

```
# 파드 확인
$ kubectl get po
NAME                     READY   STATUS    RESTARTS        AGE
kubia-695dd8898f-cxhjl   0/1     Running   0               118s
kubia-f797964fb-hfd29    1/1     Running   0               15m
kubia-f797964fb-pqh6z    1/1     Running   0               22m
kubia-f797964fb-sw78t    1/1     Running   0               15m
```

하나의 새 레플리카만 시작됐음을 보여주며, 파드는 준비되지 않은 것으로 표시된다.

![https://velog.velcdn.com/images/niyu/post/a2b86ecb-5a1c-41b1-b219-e424c4171502/image.png](https://velog.velcdn.com/images/niyu/post/a2b86ecb-5a1c-41b1-b219-e424c4171502/image.png)

새 파드가 시작되자마자 레디네스프로브가 매초 시작된다. 어플리케이션에서 5번째 요청 이후에 HTTP 상태 코드 500을 반환했기 시작했기 때문에 5번째 요청에서 어플리케이션 준비가 실패했다.  파드는 결과적으로 서비스로의 엔드포인트가 제거되어 파드는 준비되지 않은 것으로 표시된다.

**잘못된 버전은 레디니스 프로브 단계에서 차단되어 파드가 생성되지 않는다.** 사용 가능한 것으로 간주되려면 10초 이상 준비돼 있어야 하기 때문에 해당 파드가 사용 가능할 때까지 롤아웃 프로세스는 새 파드를 만들지 않는다. 또한 maxUnavailable 속성이 0으로 설정되었기 때문에 원래 파드도 제거되지 않는다.

**🧩 롤아웃 데드라인 설정**

기본적으로 롤아웃이 10분 동안 진행되지 않으면 실패한 것으로 간주된다. 이 값은 스펙의 `progressDeadlineSeconds` 속성을 통해 설정할 수 있다. progressDeadlineSeconds에 지정된 시간이 초과되면 롤아웃이 자동으로 중단된다.

```
# 디플로이먼트 정보 확인
$ kubectl describe deploy kubia

Name:                   kubia
...
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    False   ProgressDeadlineExceeded
```

Progressing의 Status 항목이 False로 실패한 것을 확인할 수 있다. Reason에도 배포해야 할 기준 시간이 지나서 실패했다는 것을 볼 수 있다.

잘못된 롤아웃은 중지하도록 한다.

```
# 롤아웃 중지
$ kubectl rollout undo deployment kubia

deployment.apps/kubia rolled back
```

> 디플로이먼트를 이용해 앱을 배포할 때 롤링 업데이트하거나, 앱 배포 도중 잠시 멈췄다가 다시 배포할 수 있다. 또한 앱 배포 후 이전 버전으로 롤백할 수 있다.
>

**References**

- [책 | Kubernetes in Action 9장](http://www.yes24.com/Product/Goods/89607047)
- [sungsu9022 | 9. 디플로이먼트](https://github.com/sungsu9022/study-kubernetes-in-action/issues/9)