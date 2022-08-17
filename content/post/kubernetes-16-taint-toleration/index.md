+++
author = "Jinsoo Youn"
title = "16장 - Taint & Toleration and Affinity"
date = "2022-08-10"
description = "pod scheduling의 고급 기술"
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

pod를 node에 스케쥴링 하는 방법인 Taint & Toleration 그리고 Affinity에 대해 알아보자
<!--more-->

<aside>
📂 [https://velog.io/@niyu/k8s-taint-toleration](https://velog.io/@niyu/k8s-taint-toleration)
[https://velog.io/@niyu/k8s-node-affinity-pod-affinity](https://velog.io/@niyu/k8s-node-affinity-pod-affinity)

</aside>

파드 스펙의 노드 셀렉터를 통해 특정 노드로 스케줄되는 것을 수행할 수 있지만 k8s는 이를 확장하는 메커니즘을 제공한다.

> 테인트 와 톨러레이션 은 특정 노드에서 파드의 실행 여부를 통제하는 데 사용된다. 파드는 노드의 테인트가 허용(tolerate)된 경우에만 노드에 스케줄될 수 있다.
>

# 테인트

테인트를 설정한 노드에는 파드들을 스케줄링하지 않는다.

테인트에는 `key`, `value`, `effect` 가 있고, `<key>=<value>:<effect>` 으로 표시한다.

**🔎 테인트 effect**
어떤 식으로 스케줄링이 동작할지를 정의하는 설정이다.

- `NoSchedule` : 노드가 테인트를 허용(tolerate)하지 않는 경우, 파드가 노드에 스케줄되지 않는다.
- `PreferNoSchedule` : 노드 스케줄을 피하려고 하지만 다른 곳에서 스케줄할 수 없는 경우, 해당 노드에 스케줄한다.
- `NoExecute` : 스케줄링에만 영향을 미치는 NoSchedule과 PreferNoSchedule과 달리, 노드에서 실행 중인 파드에도 영향을 미친다. 해당 노드에서 이미 실행 중인 파드도 NoExecute 테인트를 톨러레이션하지 않는 파드면 노드에서 제거한다.

> 테인트는 새로운 파드(NoSchedule effect)의 스케줄링을 방지하고, 선호하지 않는 노드(PreferNoSchedule effect)를 정의하며 노드에서 기존 파드를 제거(NoExecute effect) 할 수 있다.
>

## 테인트 확인

![https://velog.velcdn.com/images/niyu/post/71875306-7197-4d52-bd79-45fdfba979dd/image.png](https://velog.velcdn.com/images/niyu/post/71875306-7197-4d52-bd79-45fdfba979dd/image.png)

마스터 노드에는 하나의 테인트가 있고, `node-role.kubernetes.io/master` 키, `null` 값, `NoSchedule`의 effect를 가진다.

이 파드가 테인트를 허용(tolerate)하지 않으면 파드는 마스터 노드에 스케줄링되지 못한다. 이 테인트를 허용(tolerate)하는 파드는 보통 시스템 파드이다.

![https://velog.velcdn.com/images/niyu/post/524dc187-2516-4e9d-8d14-82bb89b8b73d/image.png](https://velog.velcdn.com/images/niyu/post/524dc187-2516-4e9d-8d14-82bb89b8b73d/image.png)

**시스템 파드의 톨러레이션**과 **노드의 테인트**가 일치하기 때문에 마스터 노드에 스케줄된다. 톨러레이션이 없는 파드는 테인트가 없는 일반 노드에 스케줄된다.

## 테인트 추가

```
# 노드에 테인트 추가
kubectl taint node node1.k8s node-type=production:NoSchedule

node "node1.k8s" tainted
```

키는 node-type, 값은 production, effect는 NoSchedule가 있는 테인트가 추가된다.

```
# 디플로이먼트 배포
kubectl run test --image busybox --replicas 5 -- sleep 9999

deployment "test" create
```

![https://velog.velcdn.com/images/niyu/post/569dc908-a11a-4fe1-a7ab-bf8f2fff5123/image.png](https://velog.velcdn.com/images/niyu/post/569dc908-a11a-4fe1-a7ab-bf8f2fff5123/image.png)

테인트된 노드에 스케줄된 파드가 없음을 확인할 수 있다.

# 톨러레이션

톨러레이션은 `파드`에 적용되며, 테인트된 노드에 파드들을 스케줄링되게 한다.

> 노드는 하나 이상의 테인트를 가질 수 있고 파드는 하나 이상의 톨러레이션을 가질 수 있다.
>

## 톨러레이션 추가

톨러레이션에서 key, value, effect는 원하는 taint의 값을 지정해 주면 된다.

```yaml
# 톨러레이션 추가 (1)
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: prod
spec:
  replicas: 5
  template:
    spec:
      ...
      tolerations:  # 프로덕션 노드에 파드가 스케줄링 될 수 있도록 톨러레이션 추가
      - key: node-type
        operator: Equal
        value: production
        effect: NoSchedule

```

![https://velog.velcdn.com/images/niyu/post/fc10cf27-b239-4f8f-b1a1-bb68ee382c15/image.png](https://velog.velcdn.com/images/niyu/post/fc10cf27-b239-4f8f-b1a1-bb68ee382c15/image.png)

디플로이먼트를 배포하면 해당 파드가 프로덕션 노드에 배포되는 것을 확인할 수 있다.

```yaml
# 톨러레이션 추가 (2)
...
tolerations:
- key: node-type
  operator: Exists
  effect: NoSchedule
```

`Equal` 연산자를 지정해 특정 값을 허용하거나 `Exists` 연산자를 사용하는 경우에는 특정 테인트 키의 값을 허용할 수 있다.

- `Equal` : key, value가 일치하는 경우에만 파드 스케줄링을 진행한다.
- `Exist` : value는 필요없고, key만 일치할 경우 파드 스케줄링을 진행한다.

또한 key, effect 값을 주지 않고 Exists 옵션만 줄 수도 있는데, 이러면 모든 key와 effect에 적용되서 어떤 taint가 걸려있던 상관없이 스케줄링되서 파드가 실행된다.

```yaml
tolerations:
- operator: "Exists"
```

## 톨러레이션의 지속 시간 설정

**🔎 빌트인 테인트**
노드 컨트롤러는 특정 컨디션이 참일 때 자동으로 노드를 테인트시킨다.

- `node.kubernetes.io/not-ready` : 노드가 준비되지 않았다. 이는 NodeCondition에서 Ready 가 "False"로 됨에 해당한다.
- `node.kubernetes.io/unreachable` : 노드가 노드 컨트롤러에서 도달할 수 없다. 이는 NodeCondition에서 Ready 가 "Unknown"로 됨에 해당한다.
- `node.kubernetes.io/network-unavailable` : 노드의 네트워크를 사용할 수 없다.
- `node.kubernetes.io/unschedulable` : 노드를 스케줄할 수 없다.

[이외의 빌트인 테인트 보기](https://kubernetes.io/ko/docs/concepts/scheduling-eviction/taint-and-toleration/#%ED%85%8C%EC%9D%B8%ED%8A%B8-%EA%B8%B0%EB%B0%98-%EC%B6%95%EC%B6%9C)

장애 상태가 정상으로 돌아오면 kubelet 또는 노드 컨트롤러가 관련 테인트를 제거할 수 있다.

**이 기능을 `tolerationSeconds` 와 함께 사용하면, 위와 같은 문제가 있는 노드에서 파드를 얼마나 오래 실행할 수 있는지에 대한 기간을 지정할 수 있다.** 예를 들어, 네트워크 장애에서 네트워크가 복구된 후에 파드가 제거되는 것을 피하기 위해 오랫동안 노드에 바인딩된 상태를 유지하려고 할 수 있다.

```yaml
# 톨러레이션에 tolerationSeconds 지정
tolerations:
- key: "node.kubernetes.io/notReady"
  operator: "Exists"
  effect: "NoExecute"
  tolerationSeconds: 300
- key: "node.kubernetes.io/unreachable"
  operator: "Exists"
  effect: "NoExecute"
  tolerationSeconds: 300
```

```
# 파드 확인
$ kubectl describe pod kube-proxy-tf7w5 -n kube-system

Name:                 kube-proxy-tf7w5
...
Tolerations:          node.kubernetes.io/notReady:NoExecute op=Exists
                      node.kubernetes.io/unreachable:NoExecute op=Exists
                      ...
```

위 두가지 톨러레이션은 파드가 300초 동안 준비되지 않았거나 접근할 수 없는 노드도 허용하고 대기할 수 있음을 의미한다.

**k8s 컨트롤 플레인은 노드가 준비되지 않았거나 더 이상 접근할 수 없다고 감지하면 300초 동안 기다렸다가 파드를 삭제하고 다른 노드로 다시 스케줄한다.**

톨러레이션을 별도로 정의하지 않은 파드는 이 두 톨러레이션이 자동으로 추가된다. 만약 조정하고 싶다면 파드 스펙에 이 두 톨러레이션을 추가해 지연 시간을 짧게 설정할 수 있다.

> 테인트와 톨러레이션은 주로 노드를 특정 역할만 하도록 만들 때 사용한다. 예를 들어 데이터베이스용 파드를 실행한 후 노드 전체의 CPU나 RAM 자원을 독점해서 사용할 수 있도록 설정할 수 있다. GPU가 있는 노드에는 실제로 GPU를 사용하는 파드들만 실행되도록 설정할 수도 있다.
>

---

k8s는 파드를 어떤 노드에 실행할 것인지에 관한 다양한 옵션이 있다. 특정 파드들을 노드 하나에 모아두거나, 같은 기능이 있는 파드들이 노드 하나에 몰려있지 않게 분산해서 실행할 수도 있다.

# 노드 어피니티

테인트는 특정 노드에서 파드를 멀리 떨어뜨리는 데 사용되지만, 노드 어피니티(=친화성)를 이용하면 파드를 특정 노드로 유인할 수 있다. 즉, k8s에게 **노드의 특정 하위 집합에만 파드를 스케줄하도록** 지시할 수 있다.

`노드 어피니티`는 `노드 셀렉터`와 같은 방식으로 라벨을 기반으로 노드를 선택한다.

## 어피니티 규칙 설정

### Hard affinity

**🧩 기존의 노드 셀렉터를 이용하는 방식**

```
apiVersion: v1
kind: Pod
metadata:
  name: kubia-gpu
spec:
  nodeSelector:
    gpu: "true"  # 이 파드는 gpu=true 라벨을 갖고 있는 노드에만 스케줄된다
...

```

노드 셀렉터를 사용해 GPU가 있는 노드에만 GPU가 필요한 파드를 배포한다.

**🧩 노드 어피니티 규칙을 이용하는 방식**

```
apiVersion: v1
kind: Pod
metadata:
  name: kubia-gpu
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:  # 이 파드는 gpu=true 라벨이 있는 노드로만 스케줄
          - key: gpu
            operator: In
            values:
            - "true"
...

```

노드 셀렉터보다 훨씬 복잡해졌지만 훨씬 더 세부적으로 설정할 수 있다.

**🔎 requiredDuringSchedulingIgnoredDuringExecution**

- `requiredDuringScheduling...` : 노드가 파드를 노드에 스케줄하기 위해 갖고 있어야 하는 라벨을 지정함을 의미한다.
- `...IgnoredDuringExecution` : 노드에서 이미 실행 중인 파드에는 영향을 미치지 않는다는 의미이다.

즉, 스케줄링하는 동안 꼭 필요한 조건을 의미한다.

**🔎 nodeSelctorTerms**

- `nodeSelectorTerms` 필드와 `matchExpressions` 필드를 이용해서 파드를 의도한 노드에 스케줄링되도록 정의할 수 있다.

![https://velog.velcdn.com/images/niyu/post/f791c47b-d61e-4ddd-8024-6871421b19d7/image.png](https://velog.velcdn.com/images/niyu/post/f791c47b-d61e-4ddd-8024-6871421b19d7/image.png)

### Soft affinity

노드 어피니티의 `preferredDuringSchedulingIgnoredDuringExecution` 필드는 스케줄링하는 동안 **만족하면 좋은** 조건을 의미한다. prefer 글자 그대로 만족하면 좋은 것이지 꼭 이 조건을 만족해야 하는 것은 아니다.

이를 이용해 스케줄링 중에 **노드의 우선순위를 지정**할 수 있다. 즉, 특정 파드를 스케줄할 때 스케줄러가 선호하는 노드를 지정할 수 있다.

먼저 동작 확인을 위해 노드에 라벨링을 한다.

![https://velog.velcdn.com/images/niyu/post/5cb9e002-340b-492b-8ac1-95298f32e387/image.png](https://velog.velcdn.com/images/niyu/post/5cb9e002-340b-492b-8ac1-95298f32e387/image.png)

예제에서는 각 노드에 노드가 속한 `가용 영역`을 지정하는 라벨과, 이를 `전용` 또는 `공유` 노드로 표시하는 라벨을 설정한다.

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: pref
spec:
  replicas: 5
  template:
    ...
    spec:
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:  # 필수가 아닌 선호 어피니티 속성 지정
          - weight: 80  # 가중치 80의 availability-zone 선호도 지정
            preference:  # 파드가 zone1에 스케줄되는 것을 더 선호한다
              matchExpressions:
              - key: availability-zone
                operator: In
                values:
                - zone1
          - weight: 20  # 가중치 20의 share-type 선호도 지정
            preference:  # 파드를 전용 노드로 스케줄하는 것을 선호하지만 가용 영역 선호도보다 4배는 덜 중요하다
              matchExpressions:
              - key: share-type
                operator: In
                values:
                - dedicated
...

```

`weight` 필드는 1부터 100까지의 값을 설정할 수 있다.

**zone1을 전용 노드로 선호**하는 디플로이먼트를 만들 수 있다. `availability-zone=zone1` 과
`share-type=dedicated` 라벨이 있는 노드에 스케줄되길 원하는 것을 확인할 수 있다. 첫 번째 어피니티 규칙은 매우 중요하기 때문에 가중치를 80으로 설정했고, 두 번째 규칙은 덜 중요하므로 20으로 설정했다.

![https://velog.velcdn.com/images/niyu/post/04eb3318-1e0a-4333-9e7a-97d0c8780988/image.png](https://velog.velcdn.com/images/niyu/post/04eb3318-1e0a-4333-9e7a-97d0c8780988/image.png)

노드는 네 개의 그룹으로 분할된다. `가용성 영역`과 `공유 유형` 라벨이 파드의 노드 어피니티와 일치하는 노드에는 **가장 높은 순위**가 매겨진다. 어피니티 규칙의 가중치에 따라 `zone1의 공유 노드`가 그 다음 우선권을 갖고 그 다음으로 `다른 영역의 전용 노드`가 우선권을 가진다. 가장 낮은 우선순위는 `그 외의 다른 모든 노드`다.

**🔎 SelectorSpreadPriority**

스케줄러가 노드를 결정하는 데 있어서 **어피니티 우선순위 지정 기능 외에도 다른 우선순위 지정 기능을 사용한다.** 그 중 하나는 `Selector-SpreadPriority` 이다.

`Selector-SpreadPriority` 기능은 동일한 레플리카셋 또는 서비스에 속하는 파드를 여러 노드에 분산시켜 노드 장애로 인해 전체 서비스가 중단되지 않도록 하는 기능이다.

만약 노드가 2개인 클러스터에 20개의 파드를 배포한다고 하면 노드 선호도 우선순위에 따라 한쪽 노드에만 20개가 스케줄 될 것 같지만, 20개 중 2개만 다른 한쪽의 노드로 스케줄된다. 노드 어피니티를 지정하지 않은 경우라면 파드는 두 노드 주위에 고르게 확산된다.

# 파드 어피니티

노드 어피니티를 이용하면 오직 **파드와 노드 사이의 친화성**에만 영향을 미친다. k8s에서는 `파드 어피니티` 를 통해 **파드들 간의 친화성** 또한 지정할 수 있다.

## 파드 어피니티

예를 들어 프론트엔드 파드와 백엔드 파드가 있다고 가정할 때, 이 두 파드를 가까이 배치할 수 있다면 대기 시간을 줄이고 어플리케이션의 성능을 향상시킬 수 있다. 이를 파드 어피니티를 이용해 달성할 수 있다.

```
# app=backend 레이블을 붙인 백앤드 파드 생성
$ kubectl run backend -l app=backend --image busybox -- sleep 99999

deployment "backend" created
```

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 5
  template:
  ...
    spec:
      affinity:
        podAffinity:  # podAffinity 규칙 정의
          requiredDuringSchedulingIgnoredDuringExecution:  # 필수 파드 어피니티 속성 정의
          - topologyKey: kubernetes.io/hostname  # 셀렉터와 부합하는 파드와 동일한 노드에 배포
            labelSelector:
              matchLabels:
                app: backend
...
```

노드 어피니티와 다르게 파드 어피니티에는 `topologyKey` 필드를 사용한다. `topologyKey` 필드는 노드 레이블의 키로, 다른 파드가 **파드와 얼마나 가까운 위치에 배포되어야 하는지 지정**한다. 파드를 스케줄링할 때 먼저 파드의 라벨 기준으로 대상 노드를 찾고 topologyKey에 매칭되는 노드들을 배포 대상으로 선택한다.

예제에서는 값을 kubernetes.io/hostname으로 지정했기 때문에 hostname을 기준으로 같은 노드에 파드를 실행한다.

![https://velog.velcdn.com/images/niyu/post/9bbafb16-0639-4e72-861e-3faba32e90c8/image.png](https://velog.velcdn.com/images/niyu/post/9bbafb16-0639-4e72-861e-3faba32e90c8/image.png)

`app=backend` 라벨이 있는 `파드`와 `디플로이먼트`가 동일한 노드에 배포되도록 하는 필수 요구 사항을 갖는 파드를 생성한다.

![https://velog.velcdn.com/images/niyu/post/b5165290-511e-4496-8737-94d8ef703f81/image.png](https://velog.velcdn.com/images/niyu/post/b5165290-511e-4496-8737-94d8ef703f81/image.png)

![https://velog.velcdn.com/images/niyu/post/1fab1324-10f6-406f-b7ac-77faa4dfa956/image.png](https://velog.velcdn.com/images/niyu/post/1fab1324-10f6-406f-b7ac-77faa4dfa956/image.png)

프론트엔드 파드가 생성되면 백엔드 파드와 동일한 node2 노드로 스케줄된 것을 확인할 수 있다.

**🔎 필수 요구 사항 대신 파드 어피니티 선호도 표시하기**

스케줄러에게 프론트엔드 파드를 백엔드 파드와 동일한 노드에 스케줄하는 것을 `선호`하도록 할 수 있다. `podAffinity`의 `preferredDuringSchedulingIgnoredDuringExecution` 속성을 이용하여 노드 어피니티 설정한 것과 동일한 처리를 할 수 있다.

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 5
  template:
    ...
    spec:
      affinity:
        podAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:  # 선호 파드 어피니티
          - weight: 80  # 가중치 80의 availability-zone 선호도 지정
            podAffinityTerm:
              topologyKey: kubernetes.io/hostname
              labelSelector:
                matchLabels:
                  app: backend
...
```

노드 어피니티의 선호도 설정 규칙과 마찬가지로 각 규칙의 가중값을 정의해야 한다.

![https://velog.velcdn.com/images/niyu/post/6c7a83e6-5624-4075-8bea-5c7ade6f2301/image.png](https://velog.velcdn.com/images/niyu/post/6c7a83e6-5624-4075-8bea-5c7ade6f2301/image.png)

![https://velog.velcdn.com/images/niyu/post/f72e1277-6801-4d9a-9bc8-f46d5a486774/image.png](https://velog.velcdn.com/images/niyu/post/f72e1277-6801-4d9a-9bc8-f46d5a486774/image.png)

백엔드 파드와 동일한 노드에 4개의 파드가 배포되고 다른 노드에는 파드가 하나가 배포되는 것을 확인할 수 있다.

> 파드 어피니티를 통해 스케줄러가 파드를 동일한 위치에 배포할 수 있다.
>

## 파드 안티-어피니티 (Pod Anti-Affinity)

파드를 서로 멀리 떨어뜨려 놓고 싶은 경우 파드 안티-어피니티 속성을 사용해서 처리할 수 있다.

`podAffinity` 대신 `podAntiAffinity` 속성을 사용하는 것 외에는 podAffinity 작성 방식과 동일하다. **podAntiAffinity의 라벨 셀렉터와 일치하는 파드가 실행 중인 노드를 선택하지 않는다.**

![https://velog.velcdn.com/images/niyu/post/2de5c947-5b9c-4d78-9063-ca856ff22f8d/image.png](https://velog.velcdn.com/images/niyu/post/2de5c947-5b9c-4d78-9063-ca856ff22f8d/image.png)

위 파드는 `app=foo` 레이블이 있는 파드가 실행중인 노드로 스케줄링 되지 않는다.

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 5
  template:
    metadata:
      labels:  # 프론트엔드 파드는 app=frontend 라벨을 갖는다
        app: frontend
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:  # 필수의 파드 안티 어피니티 사용
          - topologyKey: kubernetes.io/hostname
            labelSelector: # 프론트엔드 파드가 app=frontend 라벨을 가진 파드와 같은 노드에서 실행되지 않도록 설정
              matchLabels:
                app: frontend
      containers: ...

```

파드 안티-어피니티를 활용해서 같은 디플로이먼트의 파드를 분산시켰다.

![https://velog.velcdn.com/images/niyu/post/e31e98a4-43aa-40fc-b09a-4711f1007ed6/image.png](https://velog.velcdn.com/images/niyu/post/e31e98a4-43aa-40fc-b09a-4711f1007ed6/image.png)

2개의 파드 중 1개는 `노드 1`에, 나머지 1개는 `노드 2`에 스케줄됐다. 스케줄러가 동일한 노드에 스케줄할 수 없기 때문에 나머지 3개의 파드는 `대기(pending) 상태`가 된다.

**🔎 파드 안티-어피니티 활용 방안**

두 개의 파드 세트가 동일한 노드에서 실행되는 경우 **서로의 성능을 방해할 수도 있는 경우**에 이 기능을 사용할 수 있다. 또한 스케줄러가 동일한 그룹의 파드를 다른 가용 영역 또는 리전에 분산시켜 전체 가용 영역에 장애가 발생해도 서비스가 완전히 중단되지 않도록 하는 경우 사용할 수 있다.

> 노드 어피니티를 이용해 파드를 스케줄할 노드를 지정할 수 있다. 파드 어피니티는 동일한 노드에 파드를 배포하는 데 사용된다. 파드 안티-어피니티를 이용해 특정 파드 간에 서로 멀리 떨어지게 할 수 있다.
>

**References**

- [책 | Kubernetes in Action 16장](http://www.yes24.com/Product/Goods/89607047)
- [sungsu9022 | 고급 스케줄링](https://github.com/sungsu9022/study-kubernetes-in-action/issues/16)