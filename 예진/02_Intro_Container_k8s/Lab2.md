# Working with Cloud Build

<br>

## Task 1. Confirm that needed APIs are enabled

---

- API & Services에서 `Cloud Build` API와 `Container Registry` API를 활성화

<br>
<br>

## Task 2. Building container with Dockerfile and Cloud Build

---

- Cloud Shell 활성화
- nano 편집기로 `Dockerfile`과 간단한 테스트용 `쉘 스크립트`를 만든다.
    - 컨테이너의 기본 레이어는 `Alpine Linux`
    - 쉘 스크립트를 컨테이너의 루트 디렉터리에 배치하고 컨테이너를 실행할 때 함께 실행되도록 Dockerfile 작성
- `Cloud Build`를 호출하여 컨테이너 이미지를 빌드.
    - `Container Registry` 콘솔에서 확인가능

<br>
<br>

## Task 3. Building containers with a build configuration file and Cloud Build

---

- git clone 샘플레포
- `빌드를 설명하는 YALM 파일`이 있음
- 같은 이름의 이미지를 올려보자!
    - Dockerfile과 쉘스크립트는 이전 테스크와 동일하게 있다.
    - Cloud Build에 YAML 파일을 제출
- 빌드!
- `Congainer Registry` 콘솔을 새로고침하면 이전버전과 새버전을 확인할 수 있다.
- `Cloud Build` > `History`는 이러한 컨테이너 이미지 빌드 과정을 기록
    - Cloud Builder의 강력한 기능은 빌드에 로직을 포함한다는 것이다.

<br>
<br>

## Task 4. Building and testing containers with a build configuration file and Cloud Build

---

- 다른 yaml 파일을 살펴보자.
    - 이 yaml 파일은 방금 빌드된 이미지를 실행한다.
    - 컨테이너 이미지에 삽입된 테스트 모음을 실행하기 위한 것.
        - 이 샘플 레포에는 항상 실패하는 테스트가 있음
- 빌드하면 테스트모음이 실패하여 빌드도 실패됨