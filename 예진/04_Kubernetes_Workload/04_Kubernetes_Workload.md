# Kubernetes Workload

<br>

## Kubectl 명령어 이해하기
---

- Kubectl은 관리자가 Kubernetes 클러스터를 제어하는데 사용하는 유틸리티
    - Kube-api 서버와 소통
- 기존의 클러스터를 조회하는 것 뿐이지, 새 클러스터를 만들거나 기존 클러스터의 형태를 바꿀수는 없음
    
    → 이를 하기 위해서는, gcloud와 클라우드 콘솔이 필요함
    

**`kubectl [command] [type] [name] [flags]`**
![image](https://github.com/AUSG/2023-kubernetes-study/assets/49095587/0ccbc669-a6a5-446f-9e09-a2ff55b5ab7d)



- kubectl 명령어로 할 수 있는 것들
    - kubernetes 객체들 생성
    - 객체 보기
    - 객체 삭제
    - 구성파일을 보거나 내보내기


<br><br>

## Kubernetes에서 Deployments를 어떻게 사용하는지 이해하기

---

Deployment는?

- 업데이트된 포드를 제어된 방식으로 롤아웃
- 업데이트된 포드가 안정적이지 않으면 이전 상태로 되돌릴 수 있음
- 포드를 스케일업/아웃 할 수 있음
- stateless application!

3가지 상태

- Progressing State
- Complete State
- Failed State

Deployment를 만드는 3가지 방법

1. kubectl apply 명령어를 사용하여 선언적으로 배포를 생성할 수 있다.
2. 매개변수를 인라인으로 지정하는 kubectl run 명령어를 사용하여 명령적으로 배포를 생성
3. GCP 콘솔에서 GKE 워크로드 메뉴를 사용하는 것

레플리카셋이 새로 업데이트 되는 방식 → 램프 방식

### 블루/그린 배포전략

- 새 버전의 애플리케이션을 배포한 후 배포가 업데이트되는 동안에도 애플리케이션 서비스를 사용 가능하도록 유지하려는 경우에 유용
- 최신 버전의 애플리케이션과 함께 완전히 새로운 배포가 생성

### 카나리아 배포

- 블루-그린 방식을 기반으로 하는 또 다른 업데이트 전략으로 트래픽은 점진적으로 새버전으로 이동
- 주요 장점 : 업데이트 동안 초과 리소스 사용량을 최소화할 수 있음
- 트래픽의 일부가 새버전으로 이동하고 트래픽의 전체 100%의 트래픽이 새버전으로 점진적으로 이동하는 형태
- 그치만 단점은 시간이 오래 걸릴 수 있고, 트래픽을 정확하게 이동시키기 위해 Istio와 같은 도구가 필요할 수도 있다.

**A/B 테스트?** 

→ 데이터에서 도출된 결과를 기반으로 예측을 수행할 때는 물론 비즈니스 결정을 내릴 때도 사용됨.


<br><br>

## Pods의 네트워킹 아키텍처 이해하기

---

- 각 파드는 하나의 IP를 갖는다.
- 동일한 네트워크 네임스페이스를 가짐 → 하나의 노드가 전반적으로 관리할 수 있음
- 노드가 포드의 IP를 가져올 수 있는 것은 GKE 클러스터 덕분!


<br><br>

## Kubernetes 스토리지 추상화 이해하기

---

- kubernetes에서는 스토리지 추상화를 Volume과 PersistentVolume으로 제공
- Volume : 포드에 있는 모든 컨테이너가 액세스할 수 있는 디렉터리

### **일시적인 Volume 타입 설명**

1. EmptyDir
2. ConfigMap
3. Secret
4. downwardAPI

### PersistentVolumes 추상화

- PV
    - kubernetes에서 관리함
- PVC
    - PV 사용을 위해 포드에서 보내는 요청 및 클래임