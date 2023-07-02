## Introduction to Kubernetes Workloads

### kubectl

- 관리자가 kubernetes 클러스터를 제어하는 데 사용하는 유틸리티(kube-APIserver와 통신)
- 제어 영역 서버에서 https(송수신 전부 해당)를 통해 kube-APIserver로 전송하면 kube-APIserver는 etcd를 쿼리하여 요청 처리

### kubectl config

- $home/.kube/config.에 위치
- config 파일은 cluster name과 각 cluster에 연결하는 데 사용할 사용자 인증 정보 포함
- `kubectl config view`를 통해 현재 config를 볼 수 있음
- GKE cluster에 kubectl로 접속하기 위해선 gcloud를 이용해서 인증 정보를 디렉토리에 저장해야함(`gcloud container clusters \ get-credentials [cluster_name] \ —zone [zone_name]` ⇒ 해당 명령어는 cloud shell에서 클러스터별로 한 번만 실행하면 됨

### kubectl command syntax

```bash
kubectl [command] [type] [name] [flags]
```

- command : get, describe, logs, exec, etc..
- type : pods, deployments, nodes 같은 k8s 객체
- name : 객체의 이름
- flags : 특정한 방식으로 포맷팅 하는 특별한 요청 (ex: -o=yaml or -o=wide)

### Deployments

- pod의 상태를 선언(desired state)
- roll out update
- roll back
- scale or autoscale
- for stateless applications
- yaml file → deployment object → deployment controller → create pod

### deployment lifecycle

progressing state : task가 수행됐음을 나타냄. 새로운 replicaset을 만들거나 수직 확장하거나 축소하는 task

complete state : 새로운 replicas가 최신 버전으로 업데이트됐고 사용 가능하며 실행 중인 기존 복제본이 없음을 의미

failed state : 새로운 replicaset을 만들지 못하는 경우(리소스 문제나, 권한 문제, 이미지 문제일 수도)

### create deployment ways

- yaml파일 생성 후 `kubectl apply -f test.yaml`으로 선언적으로 배포
- `kubectl run `으로 배포
    
    kubectl run [DEPLOYMENT_NAME] \
    --image [IMAGE]:[TAG] \
    --replicas 3 \
    --labels [KEY]=[VALUE] \
    --port 8080 \
    --generator deployment/apps.v1 \
    --save-config
    
- GCP console을 이용하는 방법

`kubectl get deployment [name]` or `kubectl describe deployment [name]`을 통해 상세 보기 가능!

또한, `kubectl get deployment [DEPLOYMENT_NAME] -o yaml > this.yaml`을 통해서 yaml로도 볼 수 있음

### services and scaling

`kubectl scale deployment [DEPLOYMENT_NAME] –replicas=5` 명령어를 통해 deployment를 scale 할 수 있음. 또는 cloud 콘솔에서 가능. 아니면 yaml을 변경해서 apply할 수도 있음.

또한, cpu 사용률 기준점과 함께 원하는 pod의 최소 및 최대치를 정할 수 있음. 이때는 autoscale(`kubectl autoscale deployment [DEPLOYMENT_NAME] --min=1 --max=3 --cpu-percent=80`) 명령어를 이용하거나 콘솔에서 autoscale 설정을 할 수 있는데 이렇게 하면 HorizontalPodAutoscaler라고 하는 object가 만들어짐.

### updating Deployments

- yaml을 수정하고 `kubectl apply -f [yaml file]` 사용
- `kubectl set image deployment [DEPLOYMENT_NAME] [IMAGE] [IMAGE]:[TAG]` 을 사용
- `kubectl edit deployment/[DEPLOYMENT_NAME]` 을 사용(vim editor로 수정)
- gcp console 이용해서 업데이트

### blue-green deployment

- 애플리케이션의 새 버전과 함께 배포가 하나 새로 생김. 그리고 트래픽을 blue 버전에서 green 버전으로 이동하고 이때 service를 이용하여 label selector로 설정

### canary deployments

- 블루그린 방식을 기반으로 트래픽을 점진적으로 이동시킴.
- 업데이트 동안 초과 리소스 사용량을 최소화할 수 있다는 장점이 있음
- 카나리 배포에서 selector는 버전 label로 지정하지 않고 application label로 지정함.
- 하지만 traffic을 정확하게 이동하려면 Istio 같은 도구가 필요
- 버전이 달라서 문제가 생길 수도 있음으로 sessionAffinity 필드를 clientIP로 설정하여 클라이언트의 첫 번째 요청이 후속 연결할 pod을 결정하도록 설정(같은 pod으로 접근하도록)


A/B 테스트는 애플리케이션의 기능의 효과 측정에 가장 적합! ⇒ 버전을 측정하고 비교한 후에 더 나은 결과를 얻은 버전으로 프로덕션 환경을 업데이트

A/B 테스트에서는 새 기능의 대상을 제어하고 사용자 행동에서 통계적으로 유의미한 차이점이 있는지 모니터링함.


Shadow test에서는 새 버전을 사용자로부터 숨기는 방식을 이용하여 새 버전을 현재 버전과 함께 배포하고 실행할 수 있음.

해당 테스트는 traffic이 복제가 되기 때문에 shadow data를 처리하는 버그와 서비스가 프로덕션에 영향을 미치지 않음.

Diffy 등의 도구를 함께 사용하면 트래픽 shadowing을 통해 실제 프로덕션 트래픽과 비교하여 서비스의 동작을 측정할 수 있음.


### Rolling back Deployment

```bash
kubectl rollout undo deployment [DEPLOYMENT_NAME]
kubectl rollout undo deployment [DEPLOYMENT_NAME] --to-revision=2
kubectl rollout history deployment [DEPLOYMENT_NAME] --revision=2
```

rollout undo를 통해 이전 버전으로 돌릴 수 있고 특정 버전으로도 돌릴 수 있음. history는 롤아웃 내역을 확인하는데 사용

### deployment에서 사용되는 action

```bash
# 정지
kubectl rollout pause deployment [DEPLOYMENT_NAME]
# 재개
kubectl rollout resume deployment [DEPLOYMENT_NAME]
# 모니터링
kubectl rollout status deployment [DEPLOYMENT_NAME]
```

## Pod networking

services, pods, containers, nodes 모두 다 ip 주소와 port를 사용하여 통신함.

pod은 각자의 고유한 ip를 가지고 node에서는 pod가 node의 root network namespace를 통해 상호 간에 연결됨. 그래서 pod은 vm 내에서 서로를 찾고 도달할 수 있음.

node의 root network namespace는 node의 VM NIC를 사용하면 트래픽을 node 밖으로 전달할 수 있음.

## Volumes

k8s에서는 스토리지 추상화를 volume과 PV로 제공

volumes은 pod에 있는 모든 컨테이너가 access할 수 있는 디렉토리를 의미. 임시적이라 pod이 사라지면 사라짐.

PV는 클러스터의 내구성 있는 스토리지를 관리. GKE에서는 일반적으로 영구 디스크로 지원하고 pod와 독립적으로 존재하는 클러스터 리소스임.

PV는 PVC를 통해 동적으로 프로비저닝하거나 클러스터 관리자가 명시적으로 생성할 수 있음.

volumes type

- emptyDir: pod 내의 컨테이너가 볼륨에서 읽고 쓰게 해주는 빈 디렉토리(pod이 삭제되면 디렉토리도 삭제되고 k8s가 emptyDir를 만들 때 node의 로컬 디스크를 사용하거나 메모리 지원 파일 시스템을 사용)
- ConfigMap : k8s에서 pod으로 애플리케이션 구성 데이터를 삽입하는 방법을 제공
- Secret : ConfigMap과 유사하며 보안 비밀을 사용하여 민감한 정보를 저장(비번 같은), 인메모리에 백업되기 때문에 비휘발성 스토리지에 저장되지 않음. base64 encoding 방식을 사용
- downwardAPI : downwardAPI(컨테이너가 pod 환경에 대해 가져갈 수 있는 방식) 데이터를 애플리케이션에 제공하는 데 사용

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: web
spec:
containers:
- name: web
 image: nginx
volumeMounts:
 - mountPath: /cache
 name: cache-volume
volumes:
- name: cache-volume
 emptyDir: {}
```

### PersistentVolumes 추상화

PV는 클러스터 내의 스토리지를 관리. Pod이 생명주기 동안 사용될 수 있으며 pod이 사라져도 PV의 데이터는 사라지지 않음. k8s에 의해서 관리되며 수동으로 또는 동적으로 프로비저닝됨.

GKE는 compute engine 영구 디스크를 PV로 사용할 수 있음

PVC는 PV를 사용하기 위해 pod에서 보내는 요청 및 claim임. PVC 내에서 볼륨 크기, access 모드, 스토리지 클래스를 정의.

PV는 애플리케이션 구성에서 스토리지를 분리할 수 있게 해주는 추상화 수준을 제공. PV의 스토리지는 PVC에 결합되어야만 pod에서 접근 가능.

### PV 생성 manifest

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
 name: pd-volume
spec:
storageClassName: "standard"
capacity:
storage: 100G
 accessModes:
- ReadWriteOnce:
gcePersistentDisk:
pdName: demo-disk
fsType: ext4

---
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
 name: standard
provisioner: kubernetes.io/gce-pd
parameters:
 type: pd-standard
 replication-type: none
```

여기서 storageClass는 PV 구현에 사용되는 리소스이고 PVC와 PV의 StorageClassName이 일치해야함.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
 name: pd-volume
spec:
storageClassName: "ssd"
capacity:
storage: 100G
 accessModes:
- ReadWriteOnce:
gcePersistentDisk:
pdName: demo-disk
fsType: ext4
---
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
 name: ssd
provisioner: kubernetes.io/gce-pd
parameters:
 type: pd-ssd
```

만약 ssd를 사용하려고 할 경우엔 위 예시처럼 ssd라는 새로운 storageClass를 만들면 됨.

### PVC를 pod에 추가하는 방법