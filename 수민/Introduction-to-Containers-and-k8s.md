# Introduction-to-Containers-and-k8s

## 기존 VM의 문제

VM의 경우 다른 하이퍼바이저 제품으로 환경을 이전하기가 쉽지 않았고 운영체제 부팅 시간이 길었다. 또한, VM 안에서 애플리케이션이 종속성을 공유하고 격리되지 않으면 다른 애플리케이션에서 필요한 리소스를 사용하지 못하는 경우도 존재했다.

## 컨테이너

애플리케이션 코드 실행을 위해 격리된 사용자 공간으로 경량화된 패키지로 이식성이 뛰어난 프로세스 격리 기술이다.

###컨테이너 이미지

애플리케이션 코드와 종속된 라이브러리, 환경변수를 이미지화 한것을 컨테이너 이미지라고 한다.

이미지는 레이어로 구성되어 있고 이미지로 컨테이너를 만들면 읽기 전용 위에 쓰기 전용 레이어가 올라가게 되는데 해당 레이어를 container layer라고 부른다.

구글의 이미지 레지스트리는 gcr.io이고 aws의 이미지 레지스트리는 ECR이라고 부른다.

gcp에서는 이미지를 private하게 저장하기 위해서는 Cloud IAM 사용한다.

구글에서는 Cloud Build라는 managed 서비스를 제공하며 Cloud Build는 github, git repo나, cloud source repo에서 빌드하기 위한 소스 코드를 찾는다.

Cloud Build를 사용하기 위해선 steps를 정의해야한다(코드 컴파일, 통합테스트 등등). Cloud Build는 빌드된 이미지를 GKE나 App engine, cloud function에 전달하도록 연결할 수 있다.

## 컨테이너의 리눅스 기술들
- process
- linux namespaces
- cgroups
- union file systems

## 쿠버네티스

온프레미스, 클라우드의 컨테이너 관리 및 조정 솔루션으로 오픈 소스 플래폿이다.
- open source
- automation
- container management
- declarative configuration
- Imperative configuration

### 특징

- 다양한 워크로드 환경 지원(stateful, stateless app)
- autoscaling
- resource limits
- extensibility
- Portability

### GKE

### 특징
- 완전 관리형
- 컨테이너 최적화 os
- auto upgrade(안정적인 최신 k8s cluster 버전으로 자동 업그레이드)
- auto repair(비정상 노드 자동 복구)
- cluster scaling
- seamless integration(Cloud Build 및 Container Registry와 원활하게 통합)
- IAM을 통한 액세스 제어
- 통합된 로깅과 모니터링(Stackdriver)
- 통합된 네트워킹(VPC)
- 클라우드 콘솔

## Computing options detail


### Compute Engine

- custom 가능한 VM
- persistent disk 나 local ssd 사용.
- global LB와 autoscaling 활용 가능
- per-second billing

### App Engine

- 완전형 서비스. 인프라 관리 x, 서버 관리 x
- 다양한 프로그래밍 언어나 런타임 지원
- 모니터링 로깅으로 Stackdriver 지원

### Cloud Run

- stateless container를 실행(request 처리나 pub/sub 이벤트)
- 서버리스라서 인프라 관리 x
- 자동으로 스케일링
- stateless container를 GKE or 완전 관리형 환경에 띄울 수 있음.

### Cloud Function

- Event-driven 서버리스
- 코드 실행한만큼 과금
- 람다처럼 다른 서비스 트리거로 등록 가능

## 실습 코드

```
# 여기에 실습 코드를 작성해주세요.
gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/quickstart-image .

gcloud builds submit --config cloudbuild.yaml .
---
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: [ 'build', '-t', 'gcr.io/$PROJECT_ID/quickstart-image', '.' ]
images:
- 'gcr.io/$PROJECT_ID/quickstart-image'
---
```