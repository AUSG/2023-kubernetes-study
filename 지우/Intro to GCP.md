# Brief Overview about Cloud Computing

## Five Attributes of Cloud Computing

<img src="https://github.com/AUSG/2023-kubernetes-study/assets/70079416/fb60493f-2058-4e06-af3f-bdc680878af8" width=80% height=80%>

<br>

## computing resources

`Compute Engine`: IaaS computing resource

`GKE` : Compute Engine을 기반으로 컨테이너화된 애플리케이션을 실행하는 컨테이너 서비스

- 용어 정리
    
    컨테이너화 : 이식성이 뛰어나고 리소스를 효율적으로 사용하도록 설계된 코드 패키징 방법
    
    Kubernetes: 컨테이너에서 코드를 조정하는 방법
    

`App Engine`: 완전 관리형 PaaS 서비스

- 애플리케이션 배포 서버리스 서비스

`Cloud Functions`: 서버리스 환경, 이벤트에 대한 응답으로 코드 실행

<br>

# Resource Management

- 미국, 유럽, 아시아 태평양. 3가지 리전의 3개의 가용 영역에서 서비스 제공
- 영역과 리전 내에서 네트워크 연결 속도가 빠름 (less than 1 milisecond)
- 높은 처리량과 짧은 지연 시간 제공
    
    ⇒ 에지 캐싱 네트워크로 콘텐츠와 최종 사용자를 지리적으로 가깝게 배치 → 지연 시간 최소화
    
↓↓리소스가 영역, 리전, 그리고 전역에서 관리되는 구조↓↓
<img src="https://github.com/AUSG/2023-kubernetes-study/assets/70079416/00c2fde0-99e6-41a3-bd61-4432de439daf" width=60% heigh=60%>

## H**ierarchical structure of Resources**

GCP는 프로젝트로 리소스를 관리한다.

- 프로젝트 : 리소스와 서비스 생성 및 사용, 결제, API, 권한 관리
- 영역과 리전은 GCP 리소스를 물리적으로, 프로젝트는 논리적으로 구성

**전체 구조 : 리소스 < 프로젝트 < 폴더 < 조직**

⇒ 상속 관계

- 고유 ID와 번호로 프로젝트 식별  
- 프로젝트는 폴더에 속함. 중첩도 가능  
- 단일 조직이 이 모든 폴더를 관할 및 소유 (필수는 아니지만 권장)  
- 조직을 통해 엔터프라이즈 전체에 정책을 부여 using Cloud IAM  

<img width="247" alt="Untitled 2" src="https://github.com/AUSG/2023-kubernetes-study/assets/70079416/6fa5b2fa-aab2-45db-9782-99d8f2912969" width=60% height=60%>

<br>

# Billing

> **프로젝트 수준**에서 결제 설정

- 하나 이상의 프로젝트에 결제 계정 연결 가능
    - 결제 하위 계정 분리 가능

<br>

**🔥 과도한 청구 방지 방법 3가지**

1. Budgets and alerts
    - 웹훅으로 결제 알림을 기반으로 자동화 제어 가능 (ex. 리소스 종료 스크립트 트리거, 장애 티켓 제출)
2. billing export
    - 사용량, 예상비용, 가격 책정 데이터를 빅쿼리로 보내고, 빅쿼리는 이 데이터에 액세스해 시각화
    - 파일로 내보내는 기능은 지원 중단
3. reports
    - 지출 모니터링 시각적 도구

<br>

🔥 결제 비용을 제한하는 할당량 구현 가능

1. 비율할당량
    1. GKE는 100초마다 각 프로젝트의 API 할당량을 호출 1,000개로 구현 → 100초 뒤에 한도 재설정
    2. 클러스터 자체의 관리 구성에 대한 호출 제한
2. 배정할당량
    1. 프로젝트 내 리소스 수 제어
    2. 일정 간격마다 재설정되지 않음 (비율할당량과의 차이점)

+) 고객에게도 할당량 제어 가능

<br>

# Interacting with Google Cloud

1. GCP Console
2. Cloud SDK
    1. gcloud, kubectl, gsutil, bq
3. Cloud Shell
    1. 도구 설치 필요 없이 프로젝트와 리소스 관리 가능
    2. Compute Engine 가상 머신을 기반으로 빌드, 요금 청구 X
    3. Cloud Console 액세스 기본 제공
4. Cloud Console mobile app
    1. 가상머신 관리 및 최신 청구 정보, 예산 초과 프로젝트에 대한 경보 알림 등 제공
    2. 별도 비용 X