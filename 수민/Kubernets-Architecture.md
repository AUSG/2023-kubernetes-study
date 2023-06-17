# Kubernetes Architecture

## Kubernetes Concepts

kubernetes 작동 방식에 대한 이해

- kubernetes 객체 모델
- 선언적 관리의 원칙

### kubernets objects

- object spec(desired state)
- object status(current state)

Pod

- kubernetes 모듈의 기본 구성 요소로 배포 가능한 가장 작은 객체
- 컨테이너 환경 관리 및 하나 이상의 컨테이너로 이루어짐
- 고유의 ip를 가짐

k8s의 control plane이 클러스터의 상태를 계속해서 모니터링 하고 있다가 desired state와 current state를 비교하고 필요에 따라 current state를 수정한다.

## Kubernetes Control Plane

k8s 클러스터는 컨트롤 플레인과 node로 구성되어 있고 노드의 역할을 파트를 실행하는 것이고 컨트롤 플레인의 전체 클러스터를 조정하는 역할을 한다.

구성 요소

- kube-APIserver : 사용자와 직접 상호작용하는 요소(kubectl)
- etcd : 클러스터의 db로 클러스터의 상태를 저장하는 곳(클러스터 구성 데이터가 포함)
- kube-scheduler : 노드에 pod을 예약(실제로 pod을 실행하진 않음, 노드에 아직 할당되지 않는 pod 객체들에 node를 선택하고 이름을 naming하는 것만 수행)
    - 추가적으로 affinity 속성을 이용해서 pod이 특정 node에서 실행되거나 안되도록 할 수 있음
- kube-controller-manager : kube apiserver를 통해 지속적으로 클러스터의 상태를 모니터링

노드의 구성요소

- kubelet: 각각의 노드에서 kubernetes의 agent 역할을 하고, 컨테이너 런타임을 사용하여 pod을 시작하고 준비 및 활 프로브를 포함한 수명 주기를 모니터링하고 kube-APIserver에 보낸다. (kube-APIserver와 상호작용도 o)
- kube-proxy : 클러스터의 pod 간 네트워크 연결을 유지관리

## GKE Concepts

클러스터의 초기 설정 대부분을 자동화할 수 있는 kubeadm이라는 오픈소스 명령어가 존재, but 노드가 실패하거나 유지보수가 필요하면 관리자가 수동으로 해줘야하는 단점이 존재.

GKE는 사용자를 위해 모든 제어 영역 구성요소를 관리.  

GKE는 computer engine 가상 머신 인스턴스를 실행고 노드로 등록을 수행. 사용자는 cloud 콘솔을 이용해서 노드 설정을 직접 관리 가능.

또한, 여러 개의 노드 풀을 만들어서 여러 노드 머신 유형 선택도 가능하고 노드 풀 수준에서 자동 노드 업그레이드, 복구, 클러스터 자동 확장을 설정할 수 있음. 하지만 노드 풀에 cpu 용량을 15기가 설정한다 해도 노드를 위해서 전체 다 사용하는건 아님.

기본적으로 클러스터는 3개의 같은 노드가 있는 단일 google cloud 컴퓨팅 영역에서 시작되며 모두 하나의 노드 풀에 존재.

만약 전체 노드가 다운되면 ⇒ GKE 리전 클러스터를 사용하여 control plane과 노드를 리전 내 여러 compute engine 영역에 분산. 리전 클러스터는 애플리케이션의 가용성이 단일 리전의 여러 영역에서 유지 관리되도록 함.(클러스터에 대해서 단일 api 엔드포인트를 가짐.)

기본적으로 리전 클러스터는 3개의 영역으로 분산되어 있고 각 영역에는 1개의 control plane, 3개의 node로 구성.(하나의 영역에 노드를 5개로 늘리면 다른 영역도 마찬가지로 5개가 됨)

zonal cluster를 나중에 region cluster로 바꿀 순 없고 반대도 마찬가지!

regional or zonal GKE cluster를 비공개 cluster로도 설정이 가능!(control plane에 대한 접근은 google cloud products(내부 ip주소를 통한 클라우드 로깅이나 모니터링)를 이용하거나 authorized networks(외부 ip)를 이용)

## Object Management

object는 yaml파일로 정의되고 pod의 desired state와 name, container img를 정의한다.

- apiVersion : k8s api 버전
- kind : object의 종류
- metadata : 이름(unique), 고유id, 네임스페이스 등을 사용하여 사용자가 객체를 식별
    - name은 고유해야하고 string으로 작성
    - label은 객체를 생성하는 동안이나 생성한 후 객체에 태그를 지정하는 키값 쌍

### controller object types

- Deployment
- StatefulSet
- DaemonSEt
- Job

사용자는 객체 사양 내에서 원하는 복제본 pod 수와 pod 실행 방법, pod 내에서 실행해야 하는 컨테이너, mount할 볼륨을 지정(똑같이 yaml에 작성하는데 kind는 controller가 대상)

⇒ 사용자가 정의한 yaml파일로 controller는 클러스터 내에서 pod의 원하는 상태를 유지관리

resource management for Pods and Containers

- 컨테이너가 실행할 충분한 리소스 필요
- 애플리케이션이 더 많은 리소스를 사용하는 경우(복제본이 많거나 잘못된 구성으로 프로그램이 제어할 수 없는 상태가 되어 cpu 사용이 높아지는 경우)
- pod을 지정할때 컨테이너에 필요한 각 리소스의 양(CPU, RAM)을 필요에 따라 지정 가능

namespace

- 단일의 물리적 클러스터를 여러 가상 클러스터로 추상화하는 기법
- pod, 배포, 컨트롤러와 같은 리소스에 이름 지정
- 같은 네임스페이스에서 중복된 객체 이름을 가질 수 없음(pod의 이름이 달라야함)
- 클러스터 전체에 리소스 할당량을 구현 가능(namespace 내 리소스 사용에 대한 제한 정의)
- 기본적으로 default namespace가 존재하고 workload 리소스는 기본적으로 이걸 사용
- kube-system namespace는 k8s 시스템 자체에서 생성된 객체에 대해서 사용(ConfigMap, Secrets, Deployments, Controllers)
- kube-public namespace는 모든 사용자가 읽을 수 있는 객체에 대한 namespace

namespace를 사용하는 방법은 두개

- kubectl로 yaml을 배포할 때 옵션을 거는 방법(mos flexible)

```bash
kubectl -n demo apply -f mypod.yaml
```

- yaml에 metadata에 정의하는 방법

```yaml
metadata:
	name: mypod
	namespaces: demo
```

### ReplicaSet

ReplicaSet은 pod들을 관리해서 spec에 정의한 desired state를 유지하는 객체이고 Deployment는 하나 이상의 ReplicaSet으로 구성되어 있고 롤링 업데이트를 할 때 새로운 ReplicaSet을 만들고 이전 ReplicaSet의 pod들을 하나씩 지우면서 새로운 버전으로 update를 진행

### Deployments

ReplicaSet을 이용해서 pod을 생성하고 update하고 roll back하고 scale하는 객체.

### StatefulSet

Deployment를 사용해서 만든 pod은 영구적인 id가 부여되지 않지만 StatefulSet을 사용하여 만든 pod에는 안정적인 네트워크 ID와 영구 Disk 스토리지가 포함된 고유 영구 ID가 부여된다.

### DaemonSet

DaemonSet은 특정 pod이 전체 또는 일부분의 노드에 실행되도록 보장해준다. 그리고 새로운 노드가 추가되게 되면 자동으로 pod을 띄우게 된다. DaemonSet은 주로 fluentd 같은 logging agent를 위해 사용된다.(전체 노드에서 항상 실행되게 되니까)

### Jobs

task를 수행하기 위해서 pod들을 띄우게 되고 task가 완료되면 종료된다. CronJob이 대표적이고 이건 스케쥴에 맞춰 실행되고 종료된다.(데이터 엔지니어링 쪽에서 쓰이는 유형)

### Services

특정 pod들에 access 할 수 있도록 하는 객체

- ClusterIP: 클러스터 내에서 사용할 수 있는 ip를 노출. 기본 타입
- NodePort: 클러스터의 각각의 노드에 특정 port를 뚫어서 ip를 노출
- LoadBalancer: cloud provider의 load balancing 서비스를 이용해서 서비스를 외부로 노출

GKE에서, LoadBalancer를 사용하면 기본적으로 regional network load balancing 설정 접근 가능. HTTP를 로드밸런싱을 사용하려면 ingress object를 사용해야함.

## Migration Anthos

워크로드를 google cloud의 컨테이너화된 배포로 가져올 수 있는 도구

기존 애플리케이션을 k8s로 마이그레이션. 프로세스가 자동화되고 워크로드의 위치가 상관없음(온프레미스인지 다른 클라우드인지)

### Migrate for Anthos 아키텍쳐

먼저 Migrate for compute engine이 온프레미스나 클라우드 provider로부터 google cloud로 데이터를 스트리밍해서 마이그레이션 할 수 있도록 파이프라인을 생성하도록 허용해야함. Migrate for compute engine 도구는 기존의 애플리케이션을 google cloud의 vm으로 가져오는 도구이다.

Migrate for Anthos은 GKE 클러스터에 설치되며 많은 k8s 리소스들로 구성되어 있고 이것은 k8s 구성들이나 dockerfile 같은 deployment artifacts 생성에 사용된다. 컨테이너는 cloud storage로 옮겨지고 이미지는 registry에 저장된다. deployment assets이 생성된 후에는 대상 클러스터로 배포할 수 있다.

### step

1. processing cluster를 만들고 Migrate for Anthos 설치
2. migration source 추가(vmware, aws ,azure or google cloud)
3. 수행 중인 migration의 세부정보로 migration 객체를 생성(yaml파일에 plan template 생성)
4. migration을 위한 artifacts 생성(배포에 필요한 컨테이너 이미지와 yaml파일)
5. test (컨테이너 이미지와 배포 테스트)
6. 생성된 아티팩트로 deploy