좋지, 그럼 **“내부망 로그 통합용 ELK 도입 계획서 1차 초안”** 느낌으로 쭉 짜줄게.  
바로 문서에 옮겨 써도 되게 단계별로 정리할게.

---

## 0. 전제 / 목표 정리

### 🔧 전제

- 이미 **Kafka 3노드(KRaft 모드)** 운영 중
    
- 여러 서버/프로그램에서 **파일 로그** 발생
    
- 내부망이라 외부 SaaS(X) → 온프레미스 자급자족 구조 필요
    
- Elasticsearch는 **1노드로 시작** (향후 필요 시 3노드로 확장)
    

### 🎯 목표

1. 여러 서버에 흩어진 로그를 **Kafka → Elasticsearch로 중앙집중**
    
2. Kibana에서 **서비스별/서버별/레벨별로 검색 & 대시보드 제공**
    
3. 초기에는 **PoC 수준으로 간단히 시작**, 운영 가능성 확인 후 단계적 확대
    
4. 나중에 필요하면 **Elasticsearch 다중 노드 + 고가용성**까지 확장 가능하게 설계
    

---

## 1. 도입 범위 및 우선순위 정의

### 1-1. 1차 대상 시스템 선정

- 1단계에서는 “전사 로그”가 아니라 **테스트용 또는 핵심 1~2개 서비스**부터 시작
    
    - 예:
        
        - 서비스 A (백엔드 API 서버)
            
        - 서비스 B (영상 처리/AI 연계 서버)
            

### 1-2. 수집 대상 로그 종류 결정

- 애플리케이션 로그
    
    - `/var/log/myapp/app.log`
        
    - `/opt/service/logs/service.log`
        
- 시스템 로그 (선택)
    
    - `/var/log/messages`, `/var/log/syslog`
        

➡ **1차 PoC**에선:  
👉 각 서비스의 **애플리케이션 로그만** 먼저 수집

---

## 2. 전체 아키텍처 설계

### 2-1. 목표 구조 (1노드 ES 기준)

```text
[각 서버 로그 파일]
        │
        ▼
    Fluent Bit
        │ (Kafka Producer)
        ▼
      Kafka 3노드
        │ (Kafka Consumer)
        ▼
      Logstash
        │
        ▼
Elasticsearch 1노드 ── Kibana
```

### 2-2. 서버 배치 계획 (예시)

|구성 요소|배치 방식|
|---|---|
|Kafka|기존 3노드 유지|
|Elasticsearch|신규 VM/서버 1대 (메모리 8~16GB 권장)|
|Kibana|ES와 같은 서버 또는 별도 1대|
|Logstash|ES와 같은 서버 또는 별도 1대|
|Fluent Bit|로그가 있는 각 서버(Agent 방식)|

---

## 3. 인덱스/보관 정책 설계

### 3-1. 인덱스 네이밍 전략

- 인덱스 패턴:
    
    - `app-logs-YYYY.MM.DD`
        
- 예:
    
    - `app-logs-2025.11.14`
        

### 3-2. 샤드/레플리카 설정 (1노드 기준)

- `number_of_shards: 1`
    
- `number_of_replicas: 0` (1노드라 replica가 의미 없음)
    

템플릿 예시:

```json
PUT _template/app-logs-template
{
  "index_patterns": ["app-logs-*"],
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}
```

### 3-3. 로그 보관 기간 정책

- PoC 단계:
    
    - **7일~30일 정도** 보관 후 삭제 (디스크 상황 보고 조정)
        
- 나중에:
    
    - 서비스별, 로그레벨별 다른 정책 가능
        
        - 예: ERROR 로그 90일, INFO 로그 30일
            

---

## 4. 단계별 도입 플랜

### ▶ 단계 1: PoC 환경 구성 (테스트용)

**목표:**

- 작은 범위에서 ELK + Kafka 파이프라인이 **정상 동작하는지 검증**
    

**작업 목록:**

1. Elasticsearch 단일 노드 설치
    
    - `discovery.type: single-node` 설정
        
2. Kibana 설치 및 ES 연결 확인
    
3. Logstash 설치
    
4. Kafka에 **로그용 토픽** 생성
    
    ```bash
    kafka-topics.sh --create \
      --topic app-logs \
      --partitions 3 \
      --replication-factor 3 \
      --bootstrap-server node1:9092,node2:9092,node3:9092
    ```
    
5. 특정 테스트 서버 1대에 Fluent Bit 설치
    
    - `/var/log/myapp/test.log`만 수집
        
6. Logstash 파이프라인 구성 (Kafka → ES)
    
7. Kibana에서 인덱스 패턴 `app-logs-*` 등록 후 로그 검색 테스트
    

**완료 기준:**

- 테스트 서버에서 로그를 남기면 **Kibana에서 바로 조회** 가능
    
- 레벨/메시지/타임스탬프 기준 검색 가능
    

---

### ▶ 단계 2: 구조 확정 & 표준 로그 포맷 정의

**목표:**

- “로그 구조를 서비스마다 제각각 쓰면 나중에 지옥” → **JSON 포맷 표준화**
    

**작업 목록:**

1. 애플리케이션 로그 포맷 확정 (가능하면 JSON)
    
    - 예시:
        
    
    ```json
    {
      "timestamp": "2025-11-14T13:20:31+09:00",
      "level": "INFO",
      "service": "video-api",
      "host": "svr-01",
      "traceId": "abcd-1234",
      "message": "search request",
      "userId": "u-1001"
    }
    ```
    
2. Fluent Bit에서 `Parser json` 사용 → Kafka로 JSON 그대로 보내기
    
3. Logstash에서 공통 필드 기준으로 처리:
    
    - `timestamp → @timestamp`
        
    - `service, host, level`를 필터링 기준 필드로 유지
        

**완료 기준:**

- 서비스가 달라도 Kibana에서 **동일한 필드 구조**로 필터링 가능
    
    - 예: `service:video-api AND level:ERROR`
        

---

### ▶ 단계 3: 1차 대상 서비스들로 확대

**목표:**

- PoC 성공 후, **실제 운용 중인 1~2개 서비스**에 적용
    

**작업 목록:**

1. 대상 서비스 서버 목록 정리
    
2. 각 서버에 Fluent Bit 설치 & 공통 설정 배포
    
3. 애플리케이션 로깅 설정 변경 (가능하면 JSON 로그로)
    
4. Kafka/Logstash/ES에 부하 모니터링:
    
    - Kafka 토픽 메시지 처리량
        
    - Logstash CPU/메모리
        
    - ES JVM heap 사용량
        
5. Kibana에서 서비스별 대시보드 생성:
    
    - 에러 로그 추이 그래프
        
    - 서비스별 로그량, 서버별 로그량
        
    - 특정 에러 발생 시 타임라인 분석용 시각화
        

**완료 기준:**

- 운영 중인 서비스 로그가 실시간에 가깝게 Kibana에 반영
    
- 장애/오류 발생 시 기존보다 **확실히 디버깅이 편해졌다고 체감**
    

---

### ▶ 단계 4: 운영 정책 수립

**목표:**

- PoC → “이제 이걸 정식으로 돌려도 되겠다” 수준으로 운영 정리
    

**작업 목록:**

1. 모니터링/알람
    
    - ES 프로세스 다운 감지
        
    - 디스크 사용량 80% 이상 알람
        
    - Logstash 장애 시 알림
        
2. 인덱스 라이프사이클
    
    - 일정 기간 지난 인덱스 삭제(또는 스냅샷 후 삭제)
        
3. 백업/복구 전략 (원하면)
    
    - 중요한 시스템이면 ES 스냅샷 → NFS or 백업 스토리지
        
4. 운영 책임자/담당 영역 정리
    
    - Kafka 팀 (너)
        
    - 로그 수집/ELK 운영 담당 (사실상 너일 확률 99%…)
        

---

### ▶ 단계 5: 향후 고도화/확장 옵션

**나중에 할 수 있는 것들:**

1. Elasticsearch 3노드 클러스터로 확장
    
    - 장애 허용, 성능 향상
        
2. 로그 레벨별 인덱스 분리
    
    - `app-logs-info-*`, `app-logs-error-*` 등
        
3. 서비스별 인덱스 분리
    
    - `video-api-logs-*`, `auth-service-logs-*`
        
4. APM, 메트릭 연계 (여유 생기면)
    
    - Jaeger/Tempo/Zipkin, Prometheus + Grafana와 결합
        

---

## 6. 문서용 한 줄 요약 버전

> **도입 목표:**  
> 내부망의 여러 서버/서비스에서 발생하는 애플리케이션 로그를 Fluent Bit–Kafka–Logstash–Elasticsearch–Kibana 구조로 중앙 수집·검색·시각화하여, 장애 분석 및 운영 효율성을 향상한다. 초기에는 Elasticsearch 단일 노드 및 일부 서비스 로그만 대상으로 PoC를 수행하고, 이후 안정성을 검증한 뒤 대상 시스템 확장 및 클러스터링(3노드)로 고도화한다.

---

원하면 다음으로는:

- **실제 도입계획서를 “목차 + 문단” 형식으로 정리한 워드용 버전**
    
- 또는 **docker-compose 기반 PoC용 환경 구성 파일**  
    (`elasticsearch + kibana + logstash` 한 번에 올라가는 셋)
    

이 중에 하나 골라달라 안 하고, 그냥 다음 답변에서 아예 **PoC용 docker-compose**까지 같이 뽑아줄까? 😄