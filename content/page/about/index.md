---
title: About
description: Cloud Platform Engineering
date: '2022-08-16'
aliases:
  - about-me
  - portfolio
  - contact
license: CC BY-NC-ND
lastmod: '2022-08-16'
menu:
    main: 
        weight: -90
        params:
            icon: user
---
# Abstract.


클라우드 세상에서 쓸모있는 인재가 되기 위해 노력하는 4년차 클라우드 개발자입니다. Kubernetes 기반 클라우드 플랫폼 구축, 운영 및 개발을 담당하고 있으며, 
**API Gateway 개발, 클라우드 기반 Oauth2 인증 개발, hypercloud 웹서버 백엔드 개발 및 CI&CD 파이프라인 구축**을 주된 역할을 맡고 있습니다.

* Golang기반 hypercloud backend 웹서버 개발과 kubernetes-operator 개발 경험을 보유하고 있으며, kubespray(ansible-playbook)으로 HA kubernetes cluster를
구축할 수 있습니다. Kubernetes를 명확히 이해하고 있으며, 클러스터 구축, trouble shooting 능력을 보유하고 있습니다. **CNCF 오픈소스 프로젝트(ArgoCD 
Helm, Cert-manager)와 keycloak, oauth2-proxy, dex** 등의 오픈소스 프로젝트를 사용해 문제 해결할 수 있으며, 소스코드를 수정하여 새로운 기능을 추가할 수 있습니다. 


* **kubernetes 인증과 인가**를 정확히 이해하고 있으며, 외부 인증서버(keycloak, google 등)를 통한 인증 환경을 구축할 수 있는 능력을 보유하고 있습니다. 
현재 kubernetes 클러스터의 보안 강화를 위해 OPA를 연구 중에 있습니다.


* **API Gateway** 개발 경험을 토대로 API 요청과 보안, tracing에 대한 개념을 잘 이해하고 있으며, 현재 backend 서버 간 **service mesh** (isio, linkderd)를 연구하여 
제품에 도입하고 있습니다. 


위 같은 직무 역량과 경험을 토대로 하이브리드 클라우드 환경에 kubernetes를 적용하고 배포부터 운영까지 자동화할 수 있으며, 여러 클러스터 보안을 구축하고 관리할 수 있는 기반을 
마련할 수 있습니다. 또 한 대학원 공부한 선형대수, 확률, 머신러닝 개요 등의 지식과 실무에서 쌓은 쿠버네티스 활용을 융합하여 MLOps 개발에서도 많은 기여를 할 수 있습니다. 


# Introduction. 

제 자신을 한 단어로 “왜?” 입니다.   
  의미와 진실을 추구하기에 탐구성향이 강하며, 이해가 끝나기 전엔 행동에 신중한 편입니다. 이를 통해 저의 특징을 소개 합니다.
- 선택과 집중을 통해 문제를 이해하고 해결하기위해 노력합니다. 핵심을 토대로 효율적인 일처리가 가능합니다.
- 의미와 진실이 중요하기에 맡은 일에 보다 신중하게 판단하려 노력합니다.
- "아는 것을 안다하고, 모르는 것을 모른다" 합니다. 모르는 것은 알기위해 타인에게 고개를 숙이는 일을 부끄러워하지 않습니다.
- 무엇을 해야하는지 알기때문에 주도적으로 일은 맡아 해결합니다.
- 문제를 해결하는데 있어 중요한 것이 협력임을 잘 이해하고 있습니다. 그러므로 상대방에게 항상 예의를 갖추고 겸손합니다.

위와 같은 특징을 통해 제가 현 회사에 업무 수행하는 방식은 다음과 같습니다.

 무엇이 중요한지 스스로 탐구하기에 회사에서 주어진 업무와 별개로 Kubernetes를 중심으로 구성된 CNCF 단체의 제품을 공부했으며, Cloud 환경의 devops의 필요성을 자각하고 제품에 helm, argocd등을 적극 도입했으며 회사에서 업무를 내려주기보다 스스로 업무를 만들어 일을 진행하였습니다.  현재 더욱 더 cncf 제품 중심으로 devops 환경에 대한 공부를 지속적으로 하며 성장에 힘쓰고 있습니다.
 부족함을 알고 끊임없이 성장하려 노력합니다. 스스로 찾아 공부하며, 공부한 내용을 실무에 적용하여 활용해 보는 것을 좋아합니다. 현재 DevOps 관련해 책을 찾아 읽고, 강의를 들으며 더 알고 이해하려고 노력합니다. 귀하의 회사에 입사하여 서로 발전을 도모하고 개발하여 더 나은 가치를 실현시키고자 합니다.

---

# Work Experience.

## 티맥스 클라우드 (️2019.02 ~ 현재)
 
### API Gateway 개발 (Ingress-Contoller)

📋 Description.

: 팀내에서 초기부터 운영까지 본인 스스로 구축하고 만든 제품입니다. 초기 API-Gateway 개념을 도입하고 해당 opensource 제품 분석과 troubleshooting위한 소스코드 분석, ingress-controller 중 하나인 traefik을 통한 hypercloud 제품을 위한 api-gateway 구축, 추가로 필요한 기능등을 golang으로 추가 개발하였으며, keycloak 연동을 통한 인증,인가 로직도 구축하였습니다. 더 나아가 secure통신을 위한 tls 인증서통합과 사내에 공인 도메인 환경을 위해 AWS route53을 통한 domain 구축도 진행했습니다. 해당 제품을 CI&CD를 위해 helm chart를 만들었으며, gitops 패턴을 적용하였습니다. 해당 프로젝트를 통해 devops의 거의 모든 경험을 했습니다.

🗓️ What did I do.

- traefik(Ingress-controller)를 사용한 api-gateway 구축
- 인증, 인가 관련 로직 구축
- helm chart 개발
- tls 인증서 생성 및 통합 관리
- hypercloud 제품들의 reverse-proxy 구축
- AWS router53을 통한 자사 도메인 환경 구축

🖥️ Tech Stack.

- Traefik, Cert-manager
- Golang, knowledge of L7 networking


### OAUTH2 인증 파이프라인 개발 및 구축 (Kubernetes Authentication)

📋 Description.

: hypercloud 제품의 사용자 인증 체계를 oidc(oauth2) 기반의 오픈 소스들을 활용하여 구축하였습니다. 기존 개발된 id provider(keycloak) 에서 oauth client 오픈 소스 중 하나인 oauth2-proxy에 필요한 REST API 기능을 구현하였습니다. front 개발자와 협력으로 로그인부터 로그아웃까지 기능 테스트를 거치고, 인증 파이프라인을 쿠버네티스 위에 구축하였습니다. 또한 손쉬운 배포와 관리를 위해 helm chart를 구성하였습니다.

🗓️ What did I do.

- oauth2(oidc), jwt, cookie 개념 정리 및 사내공유
- oauth2proxy를 통한 인증 연동
- helm chart 개발

🖥️ Tech Stack.

- oauth2의 개념이해, jwt 이해
- Golang, helmchart, kubernetes

### Hypercloud 인스톨러 개발 (Kubespray-Ansible, argoCD)

📋 Description.

: ansible 기반의 HA 쿠버네티스 원격설치 위한 kubespray를 사용하여 hypercloud 인스톨러 개발을 지원했으며 구축된 클러스터에서 argoCD를 통한 앱 배포를 담당하였습니다.

🗓️ What did I do.

- kubespray를 통한 인스톨러 개발
- argoCD를 통한 앱 배포 지원
- 여러 앱들의 설치를 위한 app of apps 패턴 지원 (helm chart)

🖥️ Tech Stack.

- ansible-playbook, kubespray
- argoCD
- helm chart

### HyperCloud (Web Cloud Console)

📋 Description.

: Kubernets 기반의 클라우드 리소스 관리가 익숙하지 않은 사용자에게 좀 더 쉽고 편리하게 관리할 수 있는 서비스를 제공해주는 웹 서비스 제품입니다. 해당 제품의 안정적 운영을 위해 golang으로 구성된 backend 서버 개발과 안정적 배포를 위한 CI/CD 파이프 라인 구축 및 운영을 담당하고 있습니다. Jenkins를 통한 이미지 빌드, argoCD를 통해 운영 환경에 맞는 배포 환경을 구축하였습니다. 또한 console을 kubernetes 리소스로 관리할 수 있는 console-operator도 개발하였습니다.

🗓️ What did I do.

- Backend 서버 개발 및 관리
- kubernetes 상에서의 CI&CD 파이프라인 구축
- 설치를 위한 installer 개발

🖥️ Tech Stack.

- Golang. web programming
- Jenkins, ArgoCD
- Docker, kubernetes

### Prozone (Event Management)

📋 Description.

: 클라우드 제품으로 쿠버네티스를 하기 전에 사내 클라우드 제품을 만들고 있었습니다. 해당 제품 중 중앙에서 제품의 Event을 관리하고 저장하는 어플리케이션에 대한 유지보수 및 기능개발을 담당했습니다.

🗓️ What did I do.

- 추가 기능 요건에 대한 개발과 제품 유지보수

🖥️ Tech Stack.

- Java
- ProObject (자사 개발툴, spring과 비슷)

---

# Problem-Solving Skill.

- API Gateway 도입과정과 배포까지(devops)
    - 문제 정의: 초기 쿠버네티스를 기반으로 클라우드 제품 서비스를 개발할 때 L4 기반으로 서비스를 노출하고 있었습니다. 초기 오픈시프트 콘솔 기반으로 webserver를 구축하고 있었고, 해당 websever에서 backend 서비스의 reverse proxy 제공하고 있는 상태였습니다.
    - 대안 도출: 서비스 노출을 L4 기반인 쿠버네티스 서비스 기반이 아닌 더 상위 레이어인 L7 기반의 ingress 객체를 사용하여 노출 시키는 방법이 더 유용함을 확인하였고, ingress-controller 후보 중 traefik이라는 opensource를 사용하여 사내 제품에 대한 API Gateway를 도입하였습니다.
    - 비교 검증: ingress-controller 후보 중 kong, istio, nginx, traefik, haproxy 등등 많은 제품들이 있었으나 traefik을 API-Gateway로 선정하였으며, 이유는 다음과 같습니다.
        - golang으로 개발되어 소스코드 단까지 분석이 가능하다. 이는 어떤 원인이 생겼을 때 내부 로직까지 들어가 원인을 파악할 수 있고, 내부 구조를 이해했기에 추후 다른 ingress-controller를 사용할 시에도 해당 경험을 기반으로 이해를 손쉽게 할 수 있다.
        - API Gateway로써의 다양한 요구사항을 만족시켰으며, CRD를 통해 해당 요구 기능등을 손쉽게 추가, 삭제, 변경할 수 있다.
    - 점진 적용: API Gateway를 통해 사내 클라우드 제품인 hypercloud를 ingress 기반으로 통합하였으며, backend통합 인증서 관리를 위해 cert-manager를 도입하였고, 공인된 TLS인증서 발급을 위해 aws route53을 통한 도메인 발급, let’s encrypt를 통한 인증서 발급을 진행하였습니다.
    - 유지 보수: 개발된 API Gateway를 쿠버네티스 상에서 원할한 설치를 위해 helm chart로 만들었으며, 배포를 위해 argoCD를 적극 도입하였으며, app of apps 패턴을 이용해 여러개의 app을 하나의 app으로 묶어서 설치 순서가 보장되게끔 작업하였습니다.


---
# Skill.

## Kubernete

- kubernetes 리소스 사용법을 충분히 숙지하고 있으며, 동작 원리에 대한 이해를 기반으로 App을 배포할 수 있습니다.
- CNCF의 생태계를 이해하고 있으며 kubernetes 기반 위의 다양한 opensource 프로젝트를 활용한 경험이 있습니다.
    - Helm을 통한 App 배포 및 helm chart를 만들 수 있습니다.
    - Jenkins에서 kubernetes pod를 통한 CI 환경을 구축할 수 있습니다.
    - ArgoCD를 통한 안정적인 CD 환경을 구축할 수 있습니다.
    - Cert-Manager를 통해 안정적인 TLS 인증서 생성 및 관리할 수 있습니다.
    - API-GATEWAY 개념을 이해하고있으며, traefik을 통한 reverse-porxy 환경을 구축할 수 있습니다.
    - keycloak을 통한 유저별 인증, 인가 관리에 방안 및 사용법을 알고 있습니다.
    - helm 을 사용하여 app을 배포하고 helm chart를 production 레벨까지 만들었습니다. [https://github.com/tmax-cloud/gateway-helm-chart](https://github.com/tmax-cloud/gateway-helm-chart)
    - ArgoCD를 사내에 적극 도입하였으며 argocd를 통한 helm chart 배포를 만들었습니다. 또 한, GitOps 패턴을 이용해 git을 통한 안정적인 형상관리 개념을 사내에 도입했습니다.
    - Cert-Manager를 통한 TLS 인증서 관리를 할 수 있습니다.

## Golang

- http 서버를 개발하고 image로 패키징 할 수 있습니다.
    - net/http 패키지를 이해하고 있으며, web framework 중 gorilla/mux 와 go-chi를 통한 http 핸들러를 개발할 수 있습니다.
    - golang 으로 개발된 제품을 이미지 생성과 배포까지 할 수 있습니다.
        - [https://github.com/tmax-cloud/console](https://github.com/tmax-cloud/console)
    - kubebuilder를 사용하여 kubernetes operator를 개발할 수 있습니다.
        - [https://github.com/tmax-cloud/console-operator](https://github.com/tmax-cloud/console-operator)

## Linux 기본 지식 및 Shell Script 작성

- 리눅스 기본 명령어를 통한 trouble shooting을 할 수 있습니다.
    - 예) systemctl, journalctl를 통한 프로세스 동작
- 기본 shell script와 makefile을 통해 제품 설치 shell script를 작성할 수 있습니다.
    - [https://github.com/tmax-cloud/install-console](https://github.com/tmax-cloud/install-console)

---

# Education.

2009.03-2014.02 숭실대학교 정보통신전자공학부 (학점: 4.11/4.5, 졸업)

2014.03-2018.08 연세대학교 전기전자공학과 무선통신전공 (학점 4.12/4.5, 석박사 수료)

# Certification.

Certificate Kubernates Administrator, CKA 자격증 (2021.03.03 취득)

# Language.

TEPS 741점 취득일자 2017.11.11
