# Introduction to Containers

## Virtualization

### Base Structure

하나의 실물 서버(dedicated server)에 애플리케이션과 종속 항목 등 다 때려박음

처리 성능, 보안, 확장성 향상 & 중복성 개선 ? → 물리적인 컴퓨터를 추가

— 보통 하나의 컴퓨터는 하나의 목적으로만 사용해옴 (웹 서버, db 서버 등)

→ 근데 이건 리소스 낭비, 배포 및 유지 시간 ↑, 이식성 ↓ (특정 하드웨어에 최적화되어 있기 때문)

<br />

### 가상화 개념 등장 !

하나의 컴퓨터에 여러 가상 서버와 운영체제의 실행이 가능해짐

[Hypervisor](https://aws.amazon.com/ko/what-is/hypervisor/)(KVM) → 운영체제 종속성 타파, 여러 가상머신이 하드웨어를 공유할 수 있게 해줌

→ 배포 시간↓, 이식성 ↑

<img src="https://github.com/AUSG/2023-kubernetes-study/assets/70079416/092b10d3-a7a6-43f7-a348-ba5a0deac567" width=60% height=60%>

<br />

그러나 여전히 ..

- 애플리케이션과 종속성, 운영체제가 하나의 번들로 묶여 있음 → 매번 운영체제 부팅 시간이 요구
- 종속성을 공유하는 애플리케이션이 분리되어 있지 않음
    
    ⇒ 어느 한쪽의 종속성 업그레이드 시, 다른 애플리케이션의 작동이 멈추는 상황 → 또 다른 장애를 야기
    
- 통합 테스트로 애플리케이션 작동 모니터링 ⇒ 그러나 기본 무결성 확인을 이에 의존하면 개발 속도 둔화 가능

<br />

각 애플리케이션에 대해 전용 가상 머신을 실행하자 !

커널 격리 → 애플리케이션이 서로의 성능에 영향을 주지 않음

⇒ 그러나 이는 각 애플리케이션에 대해 각각의 커널이 돌아간다는 의미로, 확장 시에 금방 한계를 보일 수 있음

커널 업데이트

→ 대규모 시스템의 경우, 전용 VM을 갖는 것은 중복이자 낭비

→ 또한 운영체제 전부 부팅되어야 하므로 시작 속도↓

<br />

### 사용자 공간만을 가상화하는 추상화 구현 !

> 가상 머신이나 운체 전부를 가상화할 필요 없이 사용자 공간만 가상화

**이게 바로 컨테이너** → 컨테이너 : 애플리케이션 코드 실행을 위해 격리된 사용자 공간

- OS까지 포함하지 않아 가벼움
- 빠른 생성 및 종료 (vm 전체를 부팅하거나 운체를 초기화하지 않기 떄문)
- 시스템의 나머지 부분 고려할 필요 X

🔥 코드와 종속성을 패키징 → 컨테이너 실행 엔진은 런타임에서 종속 항목을 사용할 수 있도록 한다
⇒ 성능, 확장성↑

<br />

## Containers and the Images

### composition of techs

*what containers uses to isolate workloads*   
<br />

1. **Linux process**
    1. each has its own virtual memory address space (독립)
    2. rapidly created and destroyed
2. **Linux namespaces**
    1. control what applications can **see**
        1. process ID numbers
        2. directory trees
        3. IP addresses
    2. ≠ k8s namespaces
3. **Linux cgroups**
    1. control what applications can **use**
        1. maximum consumption of CPU time, memory, I/O bandwidth, etc
4. **Union file systems**
    1. encapsulate apps and dependencies (minimal layer)
    
<br />

### Container Images, structured in layers

> image read instructions from ‘container manifest’ (a.k.a Dockerfile)

- 각 레이어는 **read-only**
    - 실행 중인 컨테이너는 최상위 레이어만 수정 가능
    
    `FROM` : base image
    
    `COPY` : add a new layer
    
    `RUN` : build application using ‘make’ command
    
    `CMD` : specify a command to run within the container when launched
    
- **writable layer**도 존재
    - 실행 중인 컨테이너에 대한 변경 사항은 writable layer에 기록
- ephemeral (일시적)
    - 컨테이너 삭제 시, writable layer 내용은 영원히 손실
    - underlying image → unchanged
    - permanent data → 실행 중인 컨테이너가 아닌 다른 위치에 저장

⇒ 각 컨테이너에는 writable layer가 자체적으로 존재하고 변경 사항이 저장되므로 동일한 기본 이미지에 대한 액세스는 공유 가능, 그러나 고유한 데이터 state를 가질 수 있다

변경 사항이 있으면, 차이가 있는 레이어만 복사하여 pulldown → 새 가상머신 실행하는 것보다 훨씬 빠름

<br />

### Container Registry

nginx 웹 서버 → 컨테이너 패키징에 사용

gcr.io (Google Container Registry) : 구글에서 지원하는 컨테이너 이미지 레지스트리

- Cloud IAM과 통합하여 gcr에 프라이빗 이미지 저장

Cloud Build : 컨테이너 빌드를 위한 서비스

- 여러 코드 저장소에서 빌드용 소스코드 검색 가능
- 빌드 단계를 구성하여 종속성 가져오고 코드 컴파일
- 통합 테스트 실행 및 Docker, Gradle, Maven 도구 사용

Cloud Build가 새로 빌드한 이미지를 GKE, App Engine, Cloud Functions 등의 실행 환경에 전달

<img src="https://github.com/AUSG/2023-kubernetes-study/assets/70079416/fd23b975-1914-478c-bdd1-7d9968df929a" width=60% height=60%>

<br />
<br />
<<<<<<< HEAD

=======
>>>>>>> 2814dc5 (docs: update 2w study)

## LAB1

### Dockerfile & Cloud Build로 컨테이너 이미지 빌드

```bash
#!/bin/sh
echo "Hello, world! The time is $(date)."
```

```docker
FROM alpine
COPY quickstart.sh /
CMD ["/quickstart.sh"]
```

```bash
chmod +x quickstart.sh

# build
gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/quickstart-image .
```
<img src="https://github.com/AUSG/2023-kubernetes-study/assets/70079416/d01bae58-fc34-4d77-9d01-8b11a7358851" width=60% height=60%>

<br />
<br />
<<<<<<< HEAD

=======
>>>>>>> 2814dc5 (docs: update 2w study)

### yaml file & Cloud Build로 컨테이너 이미지 빌드

```yaml
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: [ 'build', '-t', 'gcr.io/$PROJECT_ID/quickstart-image', '.' ]
images:
- 'gcr.io/$PROJECT_ID/quickstart-image'
```

```bash
gcloud builds submit --config cloudbuild.yaml .
```

<img src="https://github.com/AUSG/2023-kubernetes-study/assets/70079416/f68c33b6-15d7-45e1-b7f5-be0a9b9484e4" width=60% height=60%>

<br />
<br />

### yaml file & Cloud Build로 이미지 빌드 및 테스트

```yaml
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: [ 'build', '-t', 'gcr.io/$PROJECT_ID/quickstart-image', '.' ]
- name: 'gcr.io/$PROJECT_ID/quickstart-image'
  args: ['fail']
images:
- 'gcr.io/$PROJECT_ID/quickstart-image'
```

```bash
gcloud builds submit --config cloudbuild.yaml .
```

test result of failure

```bash
ERROR: build step 1 "gcr.io/qwiklabs-gcp-01-a58a6495ce8e/quickstart-image" failed: starting step container failed: Error response from daemon: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "fail": executable file not found in $PATH: unknown
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

BUILD FAILURE: Build step failure: build step 1 "gcr.io/qwiklabs-gcp-01-a58a6495ce8e/quickstart-image" failed: starting step container failed: Error response from daemon: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "fail": executable file not found in $PATH: unknown
ERROR: (gcloud.builds.submit) build 9b3794be-2f1b-4a13-bafd-da01854c7183 completed with status "FAILURE"
```

<br />
<br />

# Introduction to Kubernetes

컨테이너에서 실행하는 애플리케이션은 네트워크를 통해 통신해야 하는데 컨테이너가 서로의 존재를 알 수 있게 하는 장치가 없음 

***→ 컨테이너 인프라를 잘 관리하는 방법 ? Kubernetes !***

<br />

## Kubernetes ?

**_온프렘과 클라우드 컨테이너 인프라를 조정 및 관리할 수 있도록 하는 오픈 소스 플랫폼_**

- 컨테이너화된 앱 배포, 확장, 모니터링, 부하 분산, 로깅 등의 작업을 자동화 (PaaS 솔루션의 특징)
- 광범위한 사용자 설정과 유연한 구성이 가능 (IaaS 특징)
- 선언적 구성 → 달성하고자 하는 상태를 설명하여 desired state에 도달
    - current state → desired state 일치하도록, 장애 발새 시에도 그대로 유지하도록 기능
- 명령적 구성 → 시스템 상태를 변경하기 위한 명령

<br />

## Aspects of Kubernetes

1. 다양한 워크로드 유형 지원
    1. nginx, apache 웹 서버와 같은 stateless 애플리케이션 지원
    2. 사용자 및 세션 데이터를 저장할 수 있는 stateful 애플리케이션 지원
    3. 배치 작업과 데몬 작업 지원
2. 리소스 사용률에 따라 auto scaling
3. 워크로드에 대한 리소스 요청 수준과 리소스 제한 지정
    1. 쿠버네티스는 클러스터 내에서 전반적인 워크로드 성능 개선
    2. 풍부한 플러그인과 부가기능 생태계로 쿠버네티스 확장
4. 확장성
5. 이식성 (portability)
    1. 어디서든 배포 가능
    2. 공급업체에 국한되지 않고 자유롭게 이전

<br />
<br />

# Introduction to GKE

**_쿠버네티스를 사용하는데 인프라 관리가 부담이 된다면? GKE를 활용 !_**

- 컨테이너화된 애플리케이션을 위한 쿠버네티스 환경을 배포, 관리, 확장
- 쿠버네티스 워크로드를 클라우드로 쉽게 가져올 수 있음
- 완전 관리형으로, 기본 리소스 프로비저닝할 필요 없음
- 컨테이너에 최적화된 운영체제
    - 최소한의 리소스 공간으로 빠르게 확장

<br />

## Cluster

- 자동 업그레이드 기능 → 항상 안정적인 버전으로 클러스터가 자동 업그레이드

<br />

## Node

- 컨테이너를 호스팅하는 가상 머신
- 자동 복구 기능으로 비정상 노드 자동 복구
- 주기적으로 상태 확인, 비정상이면 GKE가 노드를 드레이닝 ⇒ 워크로드가 원활하게 종료되도록 + 재생성
- 클러스터 자체의 확장을 지원
- Cloud Build, Container Registry와 원활히 통합
- IAM 통합 → 계정 및 역할 권한으로 액세스 제어
- Stackdriver(서비스, 컨테이너, 인프라 모니터링하고 관리하는 시스템)과 통합 → 앱 성능 이해↑
- VPC와 통합 → GCP의 네트워킹 기능 활용

<br />
<br />

# Computing Options

## Compute Engine

- GCP에서 실행되는 가상 머신 제공
- 성능 및 비용 요구 사항에 일치하는 맞춤형 구성 생성
- 영구 디스크와 로컬 SSD 옵션 제공
    - 최대 64TB까지 확장
- 관리형 인스턴스 그룹 기능 제공
- 비용 세밀하게 제어

### Use Cases

- 인프라 완벽 제어 !
- 애플리케이션 변경하지 않고도 워크로드를 GCP로 쉽게 lift-and-shift 가능
- 다른 컴퓨팅 옵션이 애플리케이션이나 요구사항을 지원하지 않을 때 가장 적합한 솔루션

<br />

## App Engine

- 완전 관리형 애플리케이션 플랫폼
- 서버 관리 및 구성 배포의 불필요성 → 코드만으로 인프라 배포
- 컨테이너 워크로드 실행
    - stackdriver 모니터링, 오류 보고 등의 로깅 및 진단도 통합 가능
- 버전 제어 및 트래픽 분할 지원

### Use Cases

- 웹사이트, 모바일 앱, 게임 백엔드 및 Restful API 제공

<br />

## GKE

- 배포, 확장, 로드 밸런싱, 로깅 및 모니터링 등의 기능 추가
- 클러스터 확장, 영구 디스크, 최신 업데이트에 대한 자동 업데이트 지원
    - 쿠버네티스 버전 및 비정상 노드에 대한 자동 복구
- 다양한 GCP 서비스와의 통합
    - Cloud Build, Container Registry, Stackdriver Monitoring/Logging, etc
- 여러 환경에 대한 이식성 (하이브리드, 멀티 클라우드)

### Use Cases

- 컨테이너화된 애플리케이션
- 클라우드 기반 분산 시스템
- 하이브리드 애플리케이션

<br />

## Cloud Run

- 웹 요청이나 Cloud Pub/Sub 이벤트를 통한 상태 비저장 컨테이너 실행
    - GKE 클러스터에서 컨테이너 실행 가능
- 서버 프로비저닝, 구성, 관리 등의 모든 인프라 관리를 추상화하여 코드 작성에 집중할 수 있게 함
- Auto Scaling
    - 100밀리초까지 계산하여 비용을 청구하므로 과도한 청구 방지

### Use Cases

- 요청이나 이벤트를 수신 대기하는 상태 비저장 컨테이너 배포
- 인프라 관리 필요없이 원하는 프레임워크와 언어로 애플리케이션 빌드하고 몇 초 내에 배포

<br />

## Cloud Functions

- 이벤트 기반 서버리스 컴퓨팅 서비스
    - JS, Python, Go로 작성된 코드 업로드하면 자동으로 적절한 컴퓨팅 용량 배포
- 서버 자동 확장, 내결함성 설계의 고가용성
- 코드 실행 시간에 대하여 비용 청구
- 이벤트 기반으로 몇 밀리초 내에 코드 트리거
    - HTTP 엔드포인트와 Firebase 모바일 애플리케이션 백엔드의 이벤트 기반으로 트리거 가능

### Use Cases

- MSA 일부로 사용
- 서버리스 백엔드
    - 모바일, IoT 백엔드 구축하거나 타사 서비스 및 API와 통합
    - GCS 버킷에 업로드된 파일을 실시간으로 처리
- 쿼리 및 분석을 위한 데이터 ETL 가능
    - 동영상 및 이미지 분석, 감정 분석 등의 지능형 애플리케이션 일부로도 사용
