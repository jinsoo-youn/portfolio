+++
author = "Hugo Authors"
title = "Markdown Syntax Guide"
date = "2019-03-11"
description = "Sample article showcasing basic Markdown syntax and formatting for HTML elements."
tags = [
"markdown",
"css",
"html",
"themes",
]
categories = [
"themes",
"syntax",
]
series = ["Themes Guide"]
aliases = ["migrate-from-jekyl"]
image = "pawel-czerwinski-8uZPynIu-rQ-unsplash.jpg"
+++

This article offers a sample of basic Markdown syntax that can be used in Hugo content files, also it shows whether basic HTML elements are decorated with CSS in a Hugo theme.
<!--more-->

# Pod

- 쿠버네티스의 가장 기본적인 배포 단위
- 하나 이상의 컨테이너를 포함한다.
- 여러 node에 걸쳐있지 않다.

- 컨테이너는 프로세스 자체가 하위 프로세스를 생성하지 않는 한 한 컨테이너당 하나의 프로세스만 실행하도록 설계됐다. (여러 프로세스를 한 컨테이너에서 실행하는 경우, 유지 및 로그 관리는 사용자의 책임)
- 포드는 밀접하게 연관된 프로세스를 함께 실행하고, 마치 하나의 컨테이너에서 실행되는 것처럼 동일한 환경을 제공하면서 격리된 상태로 유지한다.

- Pod의 모든 컨테이너는 모두 같은 호스트 이름 및 UTS 네트워크 인터페이스를 공유한다.
- 컨테이너가 동일한 IP 및 포트 공간을 공유하는 방법
  - Pod의 컨테이너가 동일한 네트워크에서 실행되므로 네임스페이스의 경우, 같은 IP 주소와 포트 공간을 공유한다.
  - 즉, 동일한 pod의 컨테이너에서 실행 중인 프로세스는 동일한 port 번호에 바인딩되지 않도록 주의해야 한다. 그렇지 않으면, 포트 충돌이 발생한다.
  - Pod의 컨테이너들은 localhost로 동일한 pod의 다른 컨테이너와 통신할 수 있다.

- 쿠버네티스 클러스터의 모든 pod는 공유된 단일 플랫, 네트워크 주소 공간에 위치한다.
- 이는 모든 pod가 다른 모든 pod에 접근할 수 있음을 의미한다.

  ![스크린샷 2022-07-22 오후 12.58.08.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a46639a4-a21c-4308-9cf7-124128c86096/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-07-22_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_12.58.08.png)


- 다수의 pod로 애플리케이션 분할

  개별의 pod는 특정 애플리케이션만 호스팅한다.

  - 각각의 애플리케이션에 따른 시스템 제공
  - 각각의 애플리케이션을 개별적으로 스케일링

- 하나의 pod에 다수의 컨테이너를 사용하는 경우
  - 주요 프로세스 한 개와 한 개 이상의 보조 프로세스로 구성
  - 서로 밀접하게 결합된 컨테이너

# Pod 만들기

kubectl run [pod_name] --image=[image_name]

```bash
kubectl run webserver --image=nginx --port 80

kubectl get pods
kubectl get pods -o wide
kubectl describe pod webserver
kubectl get pod webserver -o yaml
kubectl get pods -w
```

# Pod 만들기 (YAML 혹은 JSON 파일 디스크립터에서)

- 쿠버네티스 리소스는 일반적으로 JSON 또는 YAML 매니페스트를 쿠버네티스 REST API 엔드포인트에 게시해 생성한다.
- kubectl 을 사용하여 리소스를 생성할 수 있지만, 일반적으로 일부 속성만 구성할 수 있다.
- 모든 쿠버네티스 객체를 YAML 파일로 정의하면 버전 제어 시스템에 저장할 수 있으므로 버전 제어 시스템의 이점을 사용할 수 있다.

- 포드 정의 (yaml)의 주요 부분
  - metadata

    이름, 네임스페이스, 라벨, etc.

  - spec

    컨테이너, 볼륨, 그 밖의 포드 내용의 실제 설명에 해당하는 데이터.

  - status

    상태, 각 컨테이너의 설명 및 상태, 포드 내부의 IP, 그 밖의 기본 정보 등 현재 정보.

    새로운 pod를 정의할 때는 status를 제공할 필요가 없다.


- Pod의 간단한 YAML 디스크립터 만들기

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: shkim-manual
    spec:
      containers:
      - image: nginx
        name: shkim
        ports:
        - containerPort: 8080
          protocol: TCP
    ```

  - apiVersion: 쿠버네티스 API version (v1)
  - kind: 리소스 유형 (pod)
  - metadata.name: 리소스 이름 (pod 이름)
  - spec.containers: 컨테이너 구성 (단일 컨테이너)
  - port를 지정하지 않아도 pod에 연결할 수 있지만, port를 빠르게 확인하기 위해 명시적으로 정의하는 것이 좋다.

    포트를 명시적으로 정의하면, 각 포트에 이름을 할당할 수도 있다.


- YAML 파일에서 Pod 만들기

    ```bash
    kubectl create -f shkim-manual.yaml
    ```

  Pod 뿐만 아니라, 다른 리소스를 생성하는 데에도 사용된다.

- 애플리케이션 로그 보기

    ```bash
    kubectl logs shkim-manual
    kubectl logs -f shkim-manual
    
    # 다중 container로 구성된 pod의 경우
    kubectl logs shkim-manual -c [CONTAINER_NAME]
    ```

- Pod의 port에 로컬 네트워크 port forwarding

  디버깅 혹은 다른 이유로 서비스를 거치지 않고 특정 포드와 통신하고 싶을 때 사용.

  개별 pod를 효과적으로 테스트할 수 있다.

    ```bash
    kubectl port-forward shkim-manual 8080:80
    
    # 다른 terminal에서
    curl localhost:8080
    ```

- 컨테이너 내부에 접속

    ```bash
    kubectl exec webserver -it /bin/bash
    
    cd /usr/share/nginx/html
    ```


# 라벨을 이용한 Pod 구성

- Pod 조직화
- Pod 및 그 밖의 쿠버네티스 객체는 라벨로 구성할 수 있다.

- 라벨
  - 리소스에 첨부하는 key-value pair
  - 라벨 셀렉터를 사용해 리소스를 선택할 때 활용된다.

- 라벨 지정하기

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: shkim-manual-v2
      labels:
        creation_method: manual
        env: prod
    spec:
      containers:
      - image: nginx
        name: shkim
        ports:
        - containerPort: 8080
          protocol: TCP
    ```


- 라벨 확인

    ```bash
    kubectl get pods --show-labels
    
    # 특정 라벨만 확인
    kubectl get pods -L creation_method,env
    ```


- 라벨 추가 및 수정

    ```bash
    kubectl label pod shkim-manual creation_method=manual
    
    kubectl label pod shkim-manual-v2 env=debug --overwrite
    ```


# Label Selector를 통한 하위 집합 나열하기

- Label selector를 사용한 Pod 나열

    ```bash
    # creation_method=manual로 필터링
    kubectl get pod -l creation_method=manual
    
    # env 라벨을 포함하는 pod 나열
    kubectl get pod -l env
    
    # env 라벨이 없는 pod 나열
    kubectl get pod -l '!env'
    
    # !=, env in (prod, devel), env notin (prod, devel) 등 사용 가능
    # , 를 통해 다중 조건 사용 가능
    ```


# Pod Scheduling 제약을 위한 라벨과 셀렉터의 사용

- 지금까지 작성한 모든 pod는 node 전체에 무작위로 스케쥴됐다.

- Label을 사용한 워커 노드 분류

    ```bash
    kubectl label node minikube gpu=true
    
    kubectl get nodes -l gpu=true
    ```


- Pod Scheduling

  pod를 만들면 scheduler는 node를 gpu=true 라벨이 있는 것으로 선택한다.

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: shkim-manual
    spec:
      nodeSelector:
        gpu: "true"
      containers:
      - image: nginx
        name: shkim
        ports:
        - containerPort: 8080
          protocol: TCP
    ```

  - 포드를 정확한 노드로 스케쥴링

    각 노드에는 [kubernetes.io/hostname](http://kubernetes.io/hostname) key의 고유한 label이 있다.

    이것을 이용해 pod를 정확한 노드로 scheduling 할 수 있다.

    하지만, 해당 노드가 offline인 경우, 문제가 발생할 수 있다.

    따라서 항상 label selector를 통해 지정된 노드의 논리적 그룹으로 설정해야한다.


# Pod에 주석 달기

- 주석은 key-value pair로 라벨과 유사하지만, 식별 정보를 보유하지 않아서 객체를 그룹화하는 데 사용할 수 없다.
- 각 포드나 API 객체에 대한 추가 설명
- 주석은 yaml 명세서에서 확인할 수 있다.

- 주석 추가와 수정

  label 과 유사하다.

    ```bash
    kubectl annotate pod shkim-manual anno-test="annotation test" [--overwrite]
    ```


# 그룹 리소스의 네임스페이스 사용하기

- 객체는 여러 개의 라벨을 가질 수 있으므로, 하나의 객체가 여러 그룹을 구성하게 된다.
- 네임스페이스는 객체를 서로 겹치지 않는 별개의 그룹으로 분리한다.
- 네임스페이스는 객체 이름의 범위를 제공하여 동일한 리소스 이름을 여러 네임스페이스에서 사용할 수 있다.
- 대부분의 리소스 유형은 네임스페이스이지만 일부는 그렇지 않다. → 클러스트 수준의 리소스 (ex. 노드) 4장

- 네임스페이스 확인

    ```bash
    kubectl get ns
    
    # namespace를 지정하지 않으면 default 사용
    kubectl get pods --namespace kube-system
    ```

- 네임스페이스를 통해 다른 사용자의 리소스를 수정하거나 삭제하는 실수를 방지할 수 있고, 이름의 충돌에 신경쓸 필요가 없다.
- 그 외에도 특정 사용자에게만 특정 리소스 접근을 허용하거나 컴퓨팅 리소스의 제한을 줄 수도 있다. → 12~14장

- 네임스페이스 만들기
  - YAML 파일로 만들기

      ```yaml
      apiVersion: v1
      kind: Namespace
      metadata:
        name: custom-namespace
      ```

      ```bash
      kubectl create -f custom-namespace.yaml
      ```

  - kubectl create namespace로 만들기

      ```bash
      kubectl create namespace custom-namespace
      ```


    네임스페이스 객체 이름에는 점을 포함할 수 없다.


- 네임스페이스의 객체 관리
  - YAML 명세서에 `namespace: custom-namespace` 항목을 metadata 섹션에 추가.
  - 혹은 create 명령 때에 namespace 지정

      ```bash
      kubectl create -f shkim-manual.yaml -n custom-namespace
      ```

  - namespace를 지정하지 않으면, 현재 context의 기본 namespace에서 작업을 수행.

  - 현재 context의 namespace와 context 자체는 `kubectl config` 명령을 통해 변경. → 부록 A

- 네임스페이스가 제공하는 격리
  - 서로 다른 사용자가 서로 다른 네임스페이스에 pod를 배포하면 pod는 서로 격리돼 통신할 수 없지만 반드시 그렇지는 않다.
  - 이는 쿠버네티스와 함께 배포되는 네트워킹 솔루션에 따라 다르다.

# Pod의 중지와 삭제

- 이름으로 pod 삭제

    ```bash
    kubectl delete pod shkim-manual
    ```

  pod를 삭제하면 해당 pod에 포함된 모든 컨테이너를 종료하도록 SIGTERM 신호를 보내고 정상적으로 종료되도록 일정 시간 (기본적으로 30초) 동안 대기한다.

  시간 내에 종료되지 않으면 프로세스가 SIGKILL을 통해 종료된다.

  프로세스가 항상 정상적으로 종료되도록 하려면 SIGTERM 신호를 적절하게 처리해야 한다.


- Label selector로 pod 삭제

    ```bash
    kubectl delete pod -l creation_method=manual
    ```

- 전체 네임스페이스 삭제 (포함된 pod도 함께 삭제)

    ```bash
    kubectl delete ns custom-namespace
    ```

- 네임스페이스 유지하면서 해당하는 pod 모두 삭제

    ```bash
    kubectl delete pods --all
    ```

- Replication controller로 생성된 pod의 경우, 해당 pod 삭제시 새 포드가 생성된다. 해당 pod를 삭제하기 위해선 replicaiton controller도 삭제해야 한다.

- 네임스페이스의 (거의) 모든 리소스 삭제

    ```bash
    kubectl delete all --all
    ```

  특정 리소스 (7장의 시크릿 등)은 보존돼 있으므로 명시적으로 삭제해야 한다.

  위 명령은 또한 쿠버네티스 서비스를 삭제하지만 잠시 후에 자동으로 다시 생성돼야 한다.