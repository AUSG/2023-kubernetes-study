### 클라우드 컴퓨팅의 5가지 기본 속성

---

1. 주문형 셀프 서비스
    1. 클라우드 컴퓨팅 고객은 자동화된 인터페이스를 사용해 작업자의 개입없이 필요한 처리 능력, 스토리지, 네트워크를 확보
2. 네트워크를 통해 어디에서나 리소스에 액세스
    1. 대규모 풀
    2. 고객은 물리적 위치를 알필요가 없음
3. 탄력적인 리소스
    1. 확장/축소 
4. 사용하거나 예약한 만큼만 비용을 지불
    1. 리소스 사용을 중단하면 지불도 중단

<br>

### Compute

---

1. Compute Engine : IaaS 솔루션으로, 서버 인스턴스를 스스로 관리하려는 사용자에게 최대한의 유연성을 제공
2. Google Kubernetes Engine(GKE) : 구글 클라우드의 클라우드 환경에서 주어진 관리 권한에 따라 컨테이너화된 애플리케이션을 실행
3. App Engine : GCP의 완전관리형 PaaS 프레임워크로, 인프라에 대해 걱정할 필요없이 클라우드에서 코드 실행
4. Cloud Functions : 완전한 서버리스 실행 환경, 서비스로서의 기능. 이벤트에 대한 응답 

| Kubernetes Engine | A managed environment for deploying containerized applications |
| --- | --- |
| Compute Engine | A managed environment for deploying virtual machines |
| App Engine | A managed serverless platform for deploying applications |
| Cloud Functions | A managed serverless platform for deploying event-driven functions |

<br>

### Resource Management

---

- 여러 물리적, 가상화된 리소스들은 Google의 글로벌 데이터 센터 내에서 관리된다.
- Google Cloud는 멀티 리전, 리전, 영역에서 리소스를 제공
- 미주, 유럽, 아시아 태평양 이 3개의 멀티 리전으로 크게 나뉨
    - 리전 내에서는 네트워크 지연시간이 1밀리초 미만이다. (왕복)
- 리전은 또 영역으로 나뉨
    - 영역? 집중된 지리적 지역 내의 GCP 리소스 배포 위치를 말함, 리전 내의 데이터 센터
- 구글의 네트워크는 전세계에서 가장 크다.
- 작업을 멀티리전, 리전, 영역 중 어떠한 범위로 수행할지 여부를 지정할 수 있다.
- GKE를 사용하면 한 리전의 여러 영역에 리소스를 분산시키는 방법을 알 수 있다. (나중에 배울 부분)
- 프로젝트 : 기본 수준의 구성 항목으로 리소스와 서비스를 생성 및 사용하고 결제, API, 권한을 관리
    - 각 프로젝트는 고유한 프로젝트 ID와 번호를 가짐 (변경 X)
- 폴더 : 프로젝트(들)를 감쌈
    - 엔터프라이즈의 계층 구조를 반영하고 올바른 수준에서 정책을 적용하려면 폴더를 사용해야 함.
    - 폴더에 폴더 중첩 가능
- 단일 조직(Organization) : 루트
    - 폴더를 감쌈
    - 엔터프라이즈 전체에 정책 적용 가능

![image](https://github.com/AUSG/2023-kubernetes-study/assets/49095587/da8b65e9-13c1-445b-bfc2-85bbc6d37a80)

- Cloud IAM(Identity and Access Management)를 통해 사용하는 모든 GCP 리소스에 대한 액세스 제어를 미세 조정할 수 있다.
    - 리소스에 대한 사용자 액세스를 제어하는 IAM 정책을 정의하는 것
    - 해당 정책은 하위 수준으로 상속됨
    
    ![image](https://github.com/AUSG/2023-kubernetes-study/assets/49095587/589f6622-7601-4d6c-af3a-b712b7699e7e)
    
- 결제는 프로젝트 수준에서 누적됨

<br>

### Billing

---

- 프로젝트 수준에서 결제가 진행
- 결제 계정은 하나이상의 프로젝트와 연결
- 자동 청구, 인보이스 발행 가능
- 결제 하위계정 설정으로 프로젝트 별로 관리 가능
- 빌링 컨트롤 도구 3가지
    1. 예산 및 알림
        1. 결제 계정 혹은 프로젝트 수준에서 설정 가능
        2. 한도 초과시 알림
    2. 결제 내보내기
        1. BigQuery로 사용량, 예상 비용, 가격 책정 데이터를 지정한 빅쿼리 데이터 세트로 내보내고 BigQuery에서 Cloud Billing 데이터에 액세스해 세부 분석을 수행하거나 Google 데이터 스튜디오와 같은 시각화 도구로 시각화 가능
        2. 파일 내보내기 기능은 중단됨
    3. 보고서
        1. 프로젝트 또는 서비스를 기반으로 지출을 모니터링하는 콘솔의 시각적 도구
- Quotas are helpful limits
    - 비율 할당량 : 특정 시간이 지나면 재설정
        - 짧은 시간안에 굉장히 많은 수의 호출이 가능
    - 배정 할당량 : 프로젝트에 있는 리소스 수를 제어

<br>

### Interacting with Google Cloud

---

1. Google Cloud Console
    1. GCP 리소스를 관리하는 웹 기반 그래픽 사용자 인터페이스
2. Cloud SDK and Cloud Shell
    1. Cloud SDK 
        1. GCP용 명령줄 도구 모음이 포함 : gcloud, kubectl, gsutil, bq
    2. Cloud Shell : 브라우저내에서 명령줄을 통해 클라우드 리소스에 액세스 가능
        1. 완전한 최신 상태
        2. GCP 사용자에게 하나씩 제공
3. Cloud Console moblie app
4. REST-based API