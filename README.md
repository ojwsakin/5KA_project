# Cloud, K8S를 이용한 개발환경 자동구축 시스템
## 프로젝트 목표

- EKS로 구축한 인프라에서 **개발환경, 빌드, 배포를 자동화하는 시스템**을 구축합니다
- **개발자들이 자유롭게 시스템을 이용**할 수 있도록 웹 콘솔을 제공합니다
- **사용하고 있는 리소스의 정보**와 배포한 **어플리케이션의 로그 정보를 쉽게 확인** 할 수있습니다

---

## 사용한 기술 스택

**“EKS”**

- AWS에서 제공하는 K8S를 관리하는 서비스입니다

**“Jenkins”**

- 지속적인 통합과 지속적 배포를 지원하는 오픈소스 자동화 Tool입니다

**“ArgoCD”**

- 지속적 배포의 도구로서 자동으로 배포하고 업데이트 합니다

**“GitLab”**

- Git저장소를 제공하는 웹기반 플랫폼으로, CI/CD, 웹 IDE등 다양한 기능을 제공합니다

**“Nexus3”**

- 소프트웨어 레포지토리 관리자로 라이브러리, 도커 이미지등을 저장하고 관리하는 저장소입니다

**“EFK”**

- Elasticsearh, Fluent-bit, Kibana로 로그 수집, 저장, 분석을 위한 오픈소스 스택입니다

---

## Service Architecture

![image](https://user-images.githubusercontent.com/93701762/232328894-ad3be0f4-97c3-41e6-a095-67f424ee3c65.png)

- 개발자가 웹엡에 접속해서 회원가입을 합니다
    - 이 때 팀(Dev, Ops)에 맞게 회원가입을 합니다
- 프로젝트와 어플리케이션을 생성합니다
- 개발자가 변경된 code를 push합니다
- Central Cluster에 위치한 Jenkins가 변경된 코드를 기반으로 build합니다
    - 변경 사항을 반영하여 Dockerfile을 통해 image를 build합니다
- Docker image를 Nexus repository에 push합니다
- ArgoCD가 개발자가 소속된 팀의 클러스터에 어플리케이션을 배포합니다

---

## CI/CD pipeline 전략

![image](https://user-images.githubusercontent.com/93701762/232328943-7a2c340a-01de-4ecb-b59e-77633887b07f.png)

- 배포와 운영에 관련된 요소를 코드화하여 확장성, 안정성을 보장하는 GitOps기반 CI/CD pipeline을 구축했습니다
- 어플리케이션 소스 코드와 Deployment 코드의 저장소를 분리 하여 관리 했습니다
- Deployment 코드의 저장소 내의 manifest file은 Helm chart형식으로 작성하여 배포의 유연성과 수월한 버전 관리를 보장하도록 구성했습니다
- Job 실행 속도와 환경의 보안과 같은 이유로 Docker image build와 push가 일어나는 환경을 pod로 구축하여 jenkins Master와 Agent 실행 환경을 격리 했습니다

---

## 로그 모니터링 전략

![image](https://user-images.githubusercontent.com/93701762/232328982-776ce2f7-2320-41b2-9bc8-a57c830aec12.png)

- 로그 모니터링을 중앙에서 관리, 확인할 수 있도록 구축했습니다
- 분산되어 있는 로그 데이터를 한 곳에서 관리하고, 중요한 이벤트나 에러 등을 빠르게 감지하여 대응할 수 있습니다.
- 중앙 클러스터에서 로그 데이터를 집중 관리함으로써 시스템 성능도 향상될 수 있습니다

---

## EFK 선정 이유

- 높은 가용성과 검색 및 분석에 최적화된 분산형 오픈소스 검색 엔진으로 Elasticsearch를 선택했습니다
    - Elasticsearch을 Statefulset으로 배포하여 고가용성을 유지할 수 있도록 했습니다
- Fluentd의 경량화 버전인 Fluent-bit를 사용하여 프로젝트 규모에 맞는 자원 소모와 대규모 로그 데이터를 처리 할 수 있도록 했습니다
- 수집된 로그 데이터를 시각적으로 보여줄 수 있도록 Elasticsearch와 연동이 가능한 Kibana를 선택했습니다

---

## 트러블 슈팅 리스트

**“EKS를 사용하다 보니 elastic-cluster가 사용할 스토리지를 정의 하지 않으면 사용이 불가능”**

→ EBS CSI드라이버를 사용하고, storageclass를 선언하여 해결 했습니다

**“유저 별 계정을 만들려고 했는데 생성이 안됨”**

→ 공식 문서 확인 결과 유저 생성을 하기 위해서는 xpack기능을 true로 설정해주어야 합니다

**“오픈 소스 버전인 oss 이미지를 사용할 경우 xpack 기능을 지원하지 않음”**

→ 라이선스 비용을 들이지 않고 xpack 기능을 사용하는 방법은 basic 이미지를 사용하는 것이였습니다

**“elasticsearch cluster를 nodeport로 배포했을 시 다른 클러스터에 존재하는 flauent-bit에서 로그 데이터가 전송이 안됨”**

→ elasticsearch를 Loadbalancer로 배포하여 CLB를 통해 로그 데이터가 전송이 되도록 해결했습니다

---

## 프로젝트 결과, 느낀 점

- 서비스가 운영되는 환경의 로그 모니터링 시스템을 구축할 수 있었고, elasticsearch의 검색 기능을 통해 배포한 어플리케이션의 로그를 쉽게 확인 할 수 있었습니다
- 프로젝트 진행 중 에러가 발생했을 때 제가 올린 EFK stack에서 로그를 찾아보며 보다 쉽게 에러를 해결 하는 경험을 했습니다
- 협업 과정을 통해 문제를 같이 해결해 나가는 과정에서 혼자만이 아닌 모두가 성장 할 수 있었던 프로젝트 였습니다
- 많은 오픈소스들과 Tool사이에서 한번이라도 경험 해본 것과 새로운 것이라도 도전하는 것이 Devops로서 중요하다는 것을 알게 되었습니다
