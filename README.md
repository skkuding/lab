# Lab

스꾸딩에서 진행 중인 각종 실험적 프로젝트(ProSeed, 주당, 인프라팀 온보딩 등)를 진행하는 쿠버네티스 클러스터 및 인프라를 관리 합니다.
이는 개발 및 테스트 환경으로 사용되며 다른 서비스들(특히 코드당)과는 별도로 운영됩니다.

## 클러스터 정보

- 클러스터 이름: Lab
- 클러스터 유형: K3s
- 노드: [10번 서버](https://www.notion.so/skkuding/2d7ef9cff54580209eeafb3bb567fd81?v=2d7ef9cff54580e1a8c2000cff3d7e2e&source=copy_link)(컨트롤 플레인 + 워커 노드)

## 접속 방법

- 자세한 설명은 [내부 Notion 문서](https://www.notion.so/skkuding/cc8b3463567449158336c9c4ea0436f6?source=copy_link#2d7ef9cff545807fae02dd19d7476899)를 참고해주세요.

## 관리 중인 앱

- [ProSeed](https://github.com/skkuding/proseed)
- to be continued...

## 규칙과 컨벤션

### 네임스페이스 관리

각 프로젝트는 `namespace` 단위로 격리하여 배포합니다. `namespace` 이름은 프로젝트명과 동일하게 설정합니다.
예: ProSeed 프로젝트의 경우 `proseed` 네임스페이스 생성

### IaC 도구 사용

클러스터 내 리소스는 반드시 IaC(Infrastructure as Code) 도구를 사용하여 관리합니다. 이는 팀원들이 리소스 구성을 쉽게 이해하고 재현할 수 있도록 돕기 위함입니다.
만약 IaC 도구를 사용하지 않고 리소스를 생성해야 하는 경우, 반드시 해당 리소스의 생성 및 관리 방법을 문서화하여 공유해야 합니다. 그렇지 않다면 언제든 예고 없이 삭제될 수 있습니다.

- 사용 도구: YAML manifests, Helm charts, Kustomize, Terraform 등

### 리소스 쿼터 및 제한

각 네임스페이스에는 적절한 리소스 쿼터(Resource Quota)와 리소스 제한(Resource Limits)을 설정하여 클러스터 자원의 과도한 사용을 방지합니다. 이는 다른 프로젝트에 영향을 미치지 않도록 하기 위함입니다.

- 예: CPU, 메모리, 스토리지 등
현재는 리소스 쿼터 및 제한이 설정되어 있지 않습니다. 필요 시 설정을 추가할 예정입니다.

### 환경 정책

코드당과는 다르게 production, staging 환경을 구분하지 않습니다. 모든 배포는 동일한 환경에서 이루어지며, 각 프로젝트 내에서 자체적으로 환경을 구분하여 관리합니다.
