DevOps는 단순한 도구 모음이 아니라 **문화 + 프로세스 + 기술**이 어우러진 개념이다.  

아래의 10가지 구성요소 중에 내가 현실적으로 달성할 수 있는 것들을 정리해두고 이것들을 지속적으로 만들어가려고 한다.

| 번호  | 구성 요소                    | 달성여부 |
| --- | ------------------------ | ---- |
| 1   | CI/CD (지속적 통합 / 배포)      |      |
| 2   | 버전 관리 시스템                | ✅    |
| 3   | 빌드 및 자동화 도구              | ✅    |
| 4   | 인프라 자동화 (IaC)            |      |
| 5   | 컨테이너 & 오케스트레이션           |      |
| 6   | 모니터링 & 로깅                |      |
| 7   | 알림 & Incident Management |      |
| 8   | 보안 & 컴플라이언스 (DevSecOps)  |      |
| 9   | 테스트 자동화                  |      |
| 10  | 협업 도구                    |      |

이 중에서 단기적으로 달성하고자 하는 것은 CI/CD, 협업도구이다. CI/CD는 gitlab cicd를 사용하고자 한다. "외장 톰캣 기반 WAS에 대해 GitLab Runner를 통해 자동 빌드 및 배포 작업을 수행할 예정이다." 또한, 

***협업도구***는 wiki.js 와 mattermost를 사용할 것이다. 이것을 gitlab과 연동하여 이슈 관리를 해보려고 한다. 이것으로 ***알림 항목***을 달성할 수 있다.

또한 인프라 자동화로는 서버 6개가 vm으로 떠 있고, 이것에 대해 내가 제어할 수 있는 권한은 없으므로....  GitLab 기반 CI/CD, Docker 컨테이너화, Ansible을 활용한 자동화 구성을 해보려고한다.
또한 k8s를 이용하여 인프라 자체를 구성하기보다는 

- 서버 8대를 K8s 클러스터로 묶고 
- AI 엔진 Pod는 NodeAffinity로 1대당 1~2개만 올라가게 배치
- NAS 락 문제 없다면 `HPA` 적용해서 부하 높을 때만 Pod 증가
- `.dat` 쓰기 충돌 생기면 → 쓰기 Pod 1개 + 읽기 Pod N개 구조 설계 

의 방식으로 해보려고한다. 즉 소프트 오토스케일링을 도전해볼 것이다.

이렇게 인프라 자동화를 부분적으로 달성하고 컨테이너 & 오케스트레이션 역시 달성할 수 있다.

모니터링과 로깅, 보안 컴플라이언스 항목은 grafana나 다른 툴들을 도입하는 것인데, 올해 하기에는 힘들것 같다. 올해는 저 6가지 항목을 확실하게 달성하는게 목표이다.


아래는 주로 쓰이는 **핵심 구성요소** 10 가지를 구체적으로 정리한 것이다.

---

## ✅ DevOps의 주요 구성 요소

### 1. **CI/CD (지속적 통합 / 지속적 배포)**

- **CI (Continuous Integration):**  
    코드 변경사항을 자주 통합하고, 자동으로 테스트  
    → Jenkins, GitHub Actions, GitLab CI, CircleCI 등
    
- **CD (Continuous Delivery / Deployment):**  
    변경사항을 자동으로 배포까지 이어지게 하는 파이프라인 구성  
    → ArgoCD, Spinnaker, Flux 등 (특히 쿠버네티스 환경에서)
    

---

### 2. **버전 관리 시스템**

- **Git** (GitHub, GitLab, Bitbucket 등)
    
- 협업의 중심이자 모든 DevOps의 시작점
    

---

### 3. **빌드 및 자동화 도구**

- **Maven**, **Gradle** (Java 계열)
    
- **Docker** (이미지 빌드 및 배포 환경 통일)
    
- **Make**, **Bazel** 등
    

---

### 4. **인프라 자동화 (IaC: Infrastructure as Code)**

- **Terraform**, **Pulumi**, **AWS CloudFormation**
    
- 인프라 리소스를 코드로 관리하고 배포
    

---

### 5. **컨테이너 & 오케스트레이션**

- **Docker** (컨테이너 기반 앱 패키징)
    
- **Kubernetes** (컨테이너 오케스트레이션)
    
- Docker Compose (소규모 환경 구성)
    

---

### 6. **모니터링 & 로깅**

- **Prometheus**, **Grafana** – 시스템/앱 상태 시각화
    
- **ELK Stack (Elasticsearch, Logstash, Kibana)** – 로그 분석
    
- **Loki**, **Fluentd**, **Datadog**, **New Relic**, **Sentry** 등도 있음
    

---

### 7. **알림 & Incident Management**

- **Slack**, **PagerDuty**, **Opsgenie**
    
- 실시간 장애 대응 및 알림 연동
    

---

### 8. **보안 & 컴플라이언스 (DevSecOps)**

- CI/CD 파이프라인에 보안 검사를 포함
    
- **Trivy**, **SonarQube**, **Snyk** – 취약점 분석 도구
    

---

### 9. **테스트 자동화**

- 단위 테스트, 통합 테스트, E2E 테스트
    
- JUnit, Selenium, Cypress 등
    

---

### 10. **협업 도구**

- **JIRA**, **Confluence**, **Slack**, **Notion**
    
- 이슈 관리 및 문서화
    

---

## 🔁 DevOps 라이프사이클 흐름 (요약)

> **Plan → Develop → Build → Test → Release → Deploy → Operate → Monitor**  
> 이 모든 단계가 DevOps 툴들과 자동화로 연결됨

---

필요하면 각 도구별 비교나, 어떤 순서로 공부하면 좋을지도 알려줄게!  
혹시 지금 어떤 DevOps 구성에 손대보고 있는 거 있어? (Docker만 쓸 예정인지, 쿠버네티스까지 갈 건지 등)