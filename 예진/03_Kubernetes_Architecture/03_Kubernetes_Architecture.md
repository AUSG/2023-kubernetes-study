# Kubernetes Architecture

## k8s 개념

---

- k8s 객체 모델 : k8s에서 관리하는 각각의 대상은 객체로 표시되며 사용자는 이러한 객체의 속성과 상태를 보고 변경가능
- 선언적 관리의 원칙 : 관리 대상이 될 객체의 원하는 상태를 k8s에 알려주어야 함, 그러면 k8s는 그 상태를 구현하고 유지하도록 작동하게 된다. 그 과정은 ‘감시 루프’를 통해 이루어짐

![image](https://github.com/AUSG/2023-kubernetes-study/assets/49095587/dc38edbb-1a8d-4e91-a23c-ed208d420fc6)

- `k8s Control Plane` : k8s 클러스터가 작업을 수행하도록 함께 작동하는 다양한 시스템 프로세스를 가리킴
    - 클러스터의 상태를 지속적으로 모니터링하고, 선언된 것과 현재 상태를 비교하면서 필요한 경우 상태를 수정한다.
- Pod : 표준 k8s 모델의 기본 구성 요소
    
    ![image](https://github.com/AUSG/2023-kubernetes-study/assets/49095587/64dbe15f-feaa-439a-a252-a878609dd148)
    
    - 배포 가능한 가장 작은 객체
    - k8s 시스템에서 실행 중인 모든 컨테이너는 포드에 있다.
    - 컨테이너가 있는 환경을 구성
    - k8s는 각 포드에 고유한 IP 주소를 할당
    - 같은 포드내의 컨테이너들은 [localhost](http://localhost) 127.0.0.1을 통해 통신 가능
    - 포드는 자가 복구되지 않는다.

![image](https://github.com/AUSG/2023-kubernetes-study/assets/49095587/633a980d-4030-45a9-ad12-63005fb47b64)

- 컨테이너 런타임 : 컨테이너 이미지에서 컨테이너를 실행하는 방법을 알고 있는 소프트웨어

<br>
<br>

## k8s Components

---

- 클러스터를 구성하는 컴퓨터는 보통 VM이다.
- 노드의 역할은 포드를 실행시키는 것이다.
- `Control Plane`의 역할은 전체 클러스터를 조정하는 것이다.
    - `kube-apiserver` : 포드 시작을 포함하여 클러스터의 상태를 보거나 변경하는 명령어를 수락
        - kubectl 명령어를 사용 : kube-apiserver와 연결하고, kubernetes api를 사용하여 통신
        - 들어오는 요청을 인증하고 이들이 승인되고 유효한지 확인하며 허용 제어를 관리
    - `etcd` : 클러스터 데이터베이스
        - 클러스터의 상태를 안정적으로 저장
        - kube-apiserver만이 etcd와 상호작용 가능
    - `kube-scheduler` : 노드에 포드를 예약
        - 각 포드의 요구사항을 평가, 가장 적합한 노드를 선택
    - kube-controller-manager : kube-apiserver를 통해 지속적으로 클러스터의 상태를 모니터링
        - 원하는 상태로 유지시킴
    - kube-cloud-manager : 기본 클라우드 제공업체와 상호작용하는 컨트롤러를 관리
- 각 노드는 kubelet을 실행한다.
    - `kubelet` : 각 노드에서 kubernetes의 에이전트
        - kube-apiserver가 노드에서 포드를 실행하려고 하면, 해당 노드의 kubelet과 연결됨.
- kube-proxy : 클러스터의 포드 간 네트워크 연결을 유지관리하는 것
    - Linux 커널에 내장된 iptables의 방화벽 기능을 사용해 이를 수행


<br>
<br>

## GKE 개념

---

- k8s 초기 설정을 대부분 자동화할 수 있는 kubeadm
    - 그치만, 노드가 실패하거나 노드에 유지보수가 필요하면 관리자가 수동으로 대응해야 함.
- gke를 사용하면 컨트롤 플레인이 딱히 필요하지 않다.
- 노드풀은 k8s가 아닌 gke의 기능이다.

![image](https://github.com/AUSG/2023-kubernetes-study/assets/49095587/dc9e9075-a6ad-478c-ad08-6dce794d4268)

![image](https://github.com/AUSG/2023-kubernetes-study/assets/49095587/31a96e7a-15f7-44b6-acd3-c25c1965c9e3)

<br>
<br>


## Object Management

---

- 포드의 원하는 상태는 yaml 파일에서 정의 된다.
    - 이름, 실행할 특정 컨테이너 이미지 정의
    - 필수로 작성되야하는 것
        - apiVersion
        - kind
        - metadata
- yaml 파일은 버전 컨트롤이 되어야 한다.
- name은 unique 해야함

### Deployment

- 웹 서버와 같이 수명이 긴 소프트웨어 구성요소를 그룹으로 관리하고자 하는 경우에 탁월
- 해당 객체 사양 내에서 원하는 복제본 포드 수를 지정하고 포드 실행방법, 포드 내에서 실행해야 하는 컨테이너, 마운트 해야하는 볼륨을 지정

### Namespace

- 포드, 배포, 컨트롤러와 같은 리소스에 이름을 지정할 수 있는 범위를 제공
- 네임스페이스를 사용하면 클러스터 전체에 리소스 할당량을 구현할 수 있다.
- 이러한 할당량은 네임스페이스 내 리소스 소비에 대한 제한을 정의한다.

- Deployment 및 ReplicaSet에 대한 참고사항
    
    이전 비디오에서 보셨던 nginx 배포 예제는 단순화되었습니다. 실제로는 비디오에서 설명한 대로 배포 오브젝트를 시작하여 원하는 세 개의 nginx 파드를 관리합니다.
    설명한 대로 실행합니다. 하지만 Deployment 오브젝트는 파드를 관리하기 위해 레플리카셋 오브젝트를 생성한다는 점에 유의하세요.
    파드를 관리합니다. 비디오의 다이어그램에는 이 세부 사항이 빠져 있습니다.
    여러분은 레플리카셋 오브젝트보다 훨씬 더 자주 직접 디플로이먼트 오브젝트로 작업하게 될 것이다. 하지만
    레플리카셋에 대해 알아두면 디플로이먼트의 작동 방식을 더 잘 이해할 수 있기 때문에
    작동 방식을 이해하는 데 도움이 됩니다. 예를 들어, 디플로이먼트의 한 가지 기능은 디플로이먼트가 관리하는 파드의 롤링 업그레이드를 허용하는 것이다.
    롤링 업그레이드를 허용하는 것이다. 업그레이드를 수행하기 위해 디플로이먼트 오브젝트는 두 번째 레플리카셋
    오브젝트를 생성한 다음, 두 번째 레플리카셋의 (업그레이드된) 파드 수를 늘리면서 첫 번째 레플리카셋의 수를 줄인다.
    첫 번째 레플리카셋의 수를 줄인다.
    
- Service에 대한 참고사항
    - 서비스는 지정된 파드에 대한 로드 밸런싱된 액세스를 제공하한다.
    - 서비스의 세가지 기본 유형
        1. ClusterIP : 이 클러스터 내에서만 접근할 수 있는 노출된 IP
        2. NodePort : 클러스터 내 각 노드의 IP 주소(특정 포트 번호)에 서비스를 노출
        3. LoadBalancer : 클라우드 공급자가 제공하는 로드밸런싱 서비스를 사용하여 서비스를 외부에 노출
- 알아야 할 컨트롤러 오브젝트
    
    **ReplicaSets**
    
    - 레플리카셋 컨트롤러는 서로 동일한 파드 모집단이 동시에 실행되도록 한다. 디플로이먼트를 사용하면 레플리카셋과 파드에 대한 선언적 업데이트를 수행할 수 있다.
    실제로 디플로이먼트는 사용자가 지정한 선언적 목표를 달성하기 위해 자체 레플리카셋을 관리한다. 따라서 가장 일반적으로 디플로이먼트 오브젝트로 작업하게 된다.
    
    **Deployments**
    
    - 디플로이먼트를 사용하면 필요에 따라 레플리카셋을 사용하여 파드를 생성, 업데이트, 롤백 및 확장할 수 있다.
    예를 들어, 디플로이먼트의 롤링 업그레이드를 수행할 때, 디플로이먼트 오브젝트는 두 번째 레플리카셋을 생성한 다음, 원래 레플리카셋의 파드 수를 줄이면서 새 레플리카셋의 파드 수를 증가시킨다.
    
    **Replication Controllers**
    
    - 복제 컨트롤러는 레플리카셋과 배포의 조합과 유사한 역할을 수행하지만 배포의 조합과 유사한 역할을 수행하지만 더 이상 사용하지 않는 것이 좋습니다. 디플로이먼트는 레플리카셋에 대한 유용한 유용한 "프런트엔드"를 제공하기 때문에, 이 교육 과정은 주로 배포에 중점을 둡니다.
    
    **StatefulSets**
    
    - 로컬 상태를 유지하는 애플리케이션을 배포해야 하는 경우 StatefulSet이 더 나은 옵션입니다. 스테이트풀셋은 파드가 동일한 컨테이너 사양을 사용한다는 점에서 디플로이먼트와 유사하다. 그러나 디플로이먼트를 통해 생성된 파드에는 퍼시스턴트 아이덴티티가 부여되지 않으며, 반대로 스테이트풀셋을 사용하여 생성된 파드는 는 안정적인 네트워크 아이덴티티와 영구 디스크 스토리지를 가진 고유한 퍼시스턴트 아이덴티티를 가지고 있다. 영구 디스크 스토리지를 가진 고유한 영구 아이덴티티를 갖는다.
    
    **DaemonSets**
    
    - 클러스터 내의 모든 노드 또는 선택한 노드에서 특정 파드를 실행해야 하는 경우, 데몬셋을 사용한다. 데몬셋은 특정 파드가 항상 노드의 전체 또는 일부 하위 집합에서 실행되도록 한다. 새로운 노드가 추가되면, 데몬셋은 해당 노드에 필요한 사양으로 파드를 자동으로 설정한다. "데몬"이라는 단어는 컴퓨터 과학 용어이다. 다른 프로세스에 유용한 서비스를 제공하는 비대화형 프로세스를 의미한다. 쿠버네티스 클러스터 은 데몬셋을 사용하여 플루언트드와 같은 로깅 에이전트가 클러스터의 모든 노드에서 실행되도록 할 수 있다.
    
    **Jobs**
    
    - 잡 컨트롤러는 작업을 실행하는 데 필요한 하나 이상의 파드를 생성한다. 작업이 완료되면, 잡은 해당 파드를 모두 종료한다. 관련 컨트롤러는 시간 기반 스케줄에 따라 파드를 실행하는 크론잡이다.
    

<br>
<br>

## GCP의 추가 k8s 서비스

---

### Migrate for Anthos

- 기존 애플리케이션을 Kubernetes 환경으로 마이그레이션
- 프로세스가 자동화됨
- 워크로드 위치가 온프레미스든, 다른 클라우드 환경이든 상관없음