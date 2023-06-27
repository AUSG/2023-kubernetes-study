# Kubernetes Concepts

## Two elements to K8S Objects

1. k8s object models
    1. 쿠버네티스 관리 항목은 객체 단위 (’Kind’로 정의되는 object)
2. a principle of declarative management
    1. 쿠버네티스가 관리 대상의 상태를 사용자가 원하는 대로 sync & 유지

<br>

## Containers in a Pod

; 배포 가능한 가장 작은 쿠버네티스 개체

Pod는 컨테이너가 있는 환경을 구현

- 하나 이상의 컨테이너를 수용
    
    → 둘 이상이면? 컨테이너 서로 밀접하게 결합하여 네트워킹 및 스토리지를 포함한 리소스 공유
    
- 각 파드에 고유 IP를 할당
    - 파드 내 컨테이너는 IP 주소 및 네트워크 포트를 포함한 네트워크 네임스페이스를 공유
    - 동일 파드 내 컨테이너는 localhost를 통해 통신 가능
    - 컨테이너 간에 공유할 스토리지 볼륨 집합 지정

<img src="https://github.com/AUSG/2023-kubernetes-study/assets/70079416/125c96ec-4304-43b2-a55e-13d43b611be9" width=60% height=60%>

<br>

쿠버네티스는 파드를 생성하고 시작하는 역할

- 파드는 자가 복구 불가능

쿠버네티스가 원하는 상태와 현재 상태를 체크

- 불일치하면 Control Plane에서 desired state와 일치하도록 동작
- 클러스터 상태를 모니터링하여 선언형으로 전달한 desired state와 싱크를 맞춤

<br>

# Kubernetes Components

## Control Plane

; 전체 클러스터를 조정하는 컴포넌트

<br>

### kube-apiserver

사용자와 직접 상호 작용하는 단일 구성요소

- 파드 시작과 클러스터의 상태를 확인하거나 변경하는 명령을 받아들이는 역할 (w/ `kubectl`)
- kubectl로 api server에 연결하고 k8s api로 통신
- 들어오는 요청을 인증, 요청이 유효한지 확인, 허용 제어 관리
- 클러스터 상태에 대한 변경 사항은 kube-apiserver로 전달

<br>

### etcd

- 클러스터 데이터베이스
    - 클러스터 구성 데이터, 어떤 노드가 클러스터의 일부인지, 어떤 파드가 어디에서 실행되어야 하는지에 대한 모든 정보가 저장되어 있는 공간
- 클러스터 상태를 안정적으로 관리
- 사용자는 kube-apiserver를 통해 etcd와 상호 작용 (직접 접근 X)

<br>

### kube-scheduler

; 파드를 어느 노드에 스케줄링할 것인지 관장하는 컴포넌트

- 노드에 할당되지 않은 파드를 발견할 때마다 노드를 선택하고, 해당 노드를 파드 객체에 스케줄링
    - 각 파드의 요구사항에 따라 가장 적합한 노드를 선택
- 모든 노드의 상태를 인지
- 사용자가 지정한 제약 조건을 준수
    - ex. 특정 파드가 특정 메모리의 노드에서만 실행되도록 or affinity를 설정하여 특정 파드 그룹이 같은 노드 또는 다른 노드에서 실행되도록 사양 정의

<br>

### kube-controller-manager

; 클러스터가 desired state에 도달하도록 변경 시도

- kube-apiserver를 통해 클러스터의 상태를 지속적으로 모니터링

<br>

### kube-cloud-manager

- 클라우드 공급자와 상호 작용하는 컨트롤러 관리
    - ex. GCE에 클러스터 환경을 구축하면, 필요 시에 LB, storage 등의 GCP 기능을 불러오는 역할

<br>

## Node

### kubelet

; 각 노드의 에이전트

특정 노드에 파드를 실행할 시, kube-apiserver는 kubelet에 연결

- 컨테이너 런타임으로 파드를 시작하고, liveness/readiness probe를 포함한 수명 주기 모니터링하여 apiserver에 보고

<br>

### Container Runtime

- GKE의 경우, Docker의 런타임 구성요소인 containerd를 사용하여 컨테이너 실행

<br>

### kube-proxy

- 클러스터의 파드 간 네트워크 연결을 유지
    - Linux 커널에 내장된 iptables의 방화벽 기능으로 수행

<br>

# GKE Concepts

### GKE의 Control Plane & Node

- Control Plane → 고객에게 노출되지 않는 GKE의 추상적인 부분
- 노드 → 클러스터 관리자가 외부에서 GCE 인스턴스로 생성

<br>

### GKE vs User

GKE → Compute Engine 가상머신 인스턴스를 실행하고 노드로 등록

사용자 → Cloud Console에서 노드 설정을 직접 관리, 컨트롤 플레인을 제외하고 노드의 수명 시간당 비용 지불

     → 노드의 코어 수와 메모리 용량은 사용자가 지정할 수 있음

- 노드 머신 유형(default) : vCPU 2, 4GB e2-medium

<br>

### Node Pool (노드 풀)

; 클러스터 내에서 모든 구성(ex. CPU, memory)이 모두 동일한 노드 집합 (GKE 기능)

- 워크로드가 클러스터 내 올바른 하드웨어에서 실행되도록 간편한 방법을 제공
- 사용자는 label 설정하여 apiserver에 노드 풀 정보 제공
- 자동 노드 업그레드, 자동 노드 복구, 클러스터 자동 확장 구현 가능

(default) 클러스터는 3개의 노드를 보유하고 있는 하나의 노드 풀을 가진 단일 GCE에서 시작

- 노드 수는 클러스터 생성 중 or 생성 후에 변경될 수 있음
- 그러나 클러스터 변경은 X

<br>

😶‍🌫️ **근데 전체 컴퓨팅 엔진이 다운되면? GKE 리전 클러스터 사용 !**    

<img src="https://github.com/AUSG/2023-kubernetes-study/assets/70079416/7dfe84ae-2049-4388-8b21-d6f47583be1d" width=60% height=60%>

- 3개의 AZ에 배포되어 가용성↑
- (default) 1개의 Control Plane과 3개의 Node 포함
- 각 AZ에 생성된 노드 수는 모두 동일

<br>

# Kubernetes Object Management

## YAML

`apiVerson` : 객체를 생성하는 k8s api 버전을 설명

`Kind` : 원하는 객체 정의 (ex. Deployment, Service, etc)

`metadata` : 이름, 고유 ID 및 namespace(optional)를 사용하여 객체 식별

- 하나의 파일에서 여러 개체 정의 가능

<br>

*🤓 **TIP** 🤓 YAML 파일을 VCS에서 관리할 것 !*

→ *변경 사항의 추적, 관리, 철회가 편리해짐*

*→ Cloud Source Repository를 함께 사용하면 다른 GCP 리소스와 동일한 방식으로 파일의 권한을 제어할 수 있음*

<br>

`.metadata.name` : 고유한 이름 부여

- 클러스터 수명 기간 동안 고유한 UID를 가짐

`label` : 객체 및 객체 하위 집합을 식별하고 구성하는 데 활용

- `kubectl get pods --selector=app=nginx`

<br>

## Deployment

; 웹 서버와 같이 수명이 긴 소프트웨어 구성 요소에 적합

- 특히 그룹으로 관리하는 경우에 용이
- 어느 시점에서든 정의된 파드 집합이 실행되도록 함
- 템플릿에 레플리카 수, 컨테이너 이미지, 볼륨 마운트 등을 정의 → 컨트롤러는 이를 기반으로 파드의 desired state를 유지

<br>

### Namespace

쿠버네티스는 단일 물리적 클러스터를 ‘namespace’라 칭하는 여러 가상 클러스터로 추상화

**namespace**? 파드, 배포, 컨트롤러와 같은 리소스에 이름을 지정할 수 있는 범위 제공

- 하나의 namespace 내에 같은 이름의 파드를 생성할 수 없다
- 클러스터 전체에 리소스 할당량 구현
    - 여기서의 리소스 할당량? ‘ns 내 리소스 사용에 대한 제한’을 의미하며, 정의된 쿠버네티스 클러스터에 특별하게 적용되는 것
- types
    - default (기본 네임스페이스)
    - kube-system : k8s 시스템 자체에서 생성된 객체에 대한 네임스페이스
    - kube-public : 모든 사용자가 읽을 수 있는 객체에 대한 네임스페이스

<br>

🔥 파드를 생성, 업데이트, 롤백, 스케일링 → **Deployment**
🔥 기존 Deployment에 persistent storage로 지속성을 더하고 싶은 경우 → **StatefulSet**
🔥 특정 파드를 모든 노드에 배포하고 싶은 경우 → **DaemonSet**

<br>

# Migrate for Anthos Architecture

Migrate for Anthos : 워크로드를 Google Cloud의 컨테이너화된 배포로 가져오기 위한 도구

기존 애플리케이션을 쿠버네티스 환경으로 마이그레이션

- 프로세스 자동화
- 자동화 속도가 빠름 → 대부분 10분 이내로 마이그레이션 완료
- 한 번에 데이터를 마이그레이션하거나 애플리케이션이 활성화될 때까지 클라우드로 스트리밍

<br>

## Architecture

<img src="https://github.com/AUSG/2023-kubernetes-study/assets/70079416/baeddcbf-b303-4496-bd0e-e22aac95004d" width=60% height=60%>

<br>

1. Migrate for Compute Engine이 Google Cloud로 데이터를 스트리밍하거나 마이그레이션하기 위한 파이프라인 생성을 허용
2. Migrate for Anthos가 GKE 클러스터에 설치됨
    1. 배포 artifact를 생성하는 데에 사용됨
    2. 쿠버네티스 구성이나 도커파일 같은 일부 아티팩트는 VM wrapping container 생성에 활용
3. 컨테이너는 Cloud Storage로 이동, 이미지는 GCR에 저장
4. 배포 자산이 생성되면 대상 클러스터에 애플리케이션 배포 가능 !

<br>

## Migration Path

1. 처리 클러스터 생성하고 Migrate for Anthos 구성요소 설치 (w/ `migctl`)
2. 마이그레이션 소스를 추가
    1. VMWare, AWS, Azure, GCP에서 클러스터로 이관 가능
3. 수행 중인 마이그레이션 세부 정보를 사용해서 객체를 생성 ⇒ YAML 파일에 계획 템플릿이 생성됨
    1. 사용자 의도대로 구성 파일 커스터마이징
4. 계획이 준비되면 마이그레이션을 위한 아티팩트 생성
    1. 배포에 필요한 애플리케이션 컨테이너 이미지와 YAML 파일을 생성하는 단계
5. 컨테이너 이미지와 배포 테스트 수행
6. 테스트에 성공하면, 생성된 아티팩트를 사용해서 애플리케이션을 프로덕션 클러스터에 배포 가능 !

<br>

```bash
# migctl cmd로 설치 진행
migctl setup install

# 마이그레이션할 애플리케이션의 위치 지정 (ex. compute engine에서 mg할 경우)
migctl source create ce my-ce-src --project my-project --zone zone

# 소스 VM과 마이그레이션에서 제외할 데이터 식별
migctl migration create test-mg --source my-ce-src --vm-id my-id --intent Image

# 마이그레이션에 대한 아티팩트 생성
migctl migration generate-artifacts my-mg

# 생성한 아티팩트를 불러와서 테스트 진행
migctl migration get-artifacts test-mg

# kubectl로 애플리케이션 배포
kubectl apply -f deployment.yaml
```

<br>

✨ 대기 시간을 최소화하기 위한 설계 방법 : 컨테이너를 동일 파드 내에 배치 → 같은 노드 내에 스케줄링되므로 !