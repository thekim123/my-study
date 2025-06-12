---
Date: 2025-06-12
Category: Kafka
---
## ✅ 서버 구성

| 서버 이름    | IP 주소         | node.id |
| -------- | ------------- | ------- |
| broker-1 | 192.168.1.101 | 1       |
| broker-2 | 192.168.1.102 | 2       |
| broker-3 | 192.168.1.103 | 3       |

## ✅ 공통 설정 항목

```properties
controller.quorum.voters=1@192.168.1.101:9093,2@192.168.1.102:9093,3@192.168.1.103:9093
```

## ✅ 각 서버용 server.properties 템플릿

### 🔹 공통 설정 예시 (broker-1)

```properties
node.id=1
process.roles=broker,controller

controller.quorum.voters=1@192.168.1.101:9093,2@192.168.1.102:9093,3@192.168.1.103:9093

listeners=PLAINTEXT://0.0.0.0:9092,CONTROLLER://0.0.0.0:9093
advertised.listeners=PLAINTEXT://192.168.1.101:9092

controller.listener.names=CONTROLLER
listener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT

log.dirs=/data/kafka-logs
num.network.threads=3
num.io.threads=8
num.replica.fetchers=2

offsets.topic.replication.factor=3
transaction.state.log.replication.factor=3
transaction.state.log.min.isr=2

share.coordinator.state.topic.replication.factor=3
share.coordinator.state.topic.min.isr=2

min.insync.replicas=2

log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000

# 대용량 메시지 허용 시
socket.request.max.bytes=209715200
```

※ broker-2, broker-3는 `node.id` 및 `advertised.listeners`만 IP에 맞게 바꾸면 동일함

---

## ✅ 클러스터 초기화 절차 요약

1. Kafka 압축 해제 후 설정 작성 (`server.properties`)
    
2. 클러스터 UUID 생성 (한 번만 실행)
    

```bash
CLUSTER_ID=$(bin/kafka-storage.sh random-uuid)
```

3. 각 브로커에서 storage 포맷 (CLUSTER_ID는 동일하게 유지)
    

```bash
bin/kafka-storage.sh format -t $CLUSTER_ID -c config/kraft/server.properties
```

4. Kafka 서버 실행:
    

```bash
bin/kafka-server-start.sh config/kraft/server.properties
```

---

## ✅ 필수 설정 정리

```properties
node.id=브로커 고유 ID
process.roles=broker,controller
controller.quorum.voters=...
controller.listener.names=CONTROLLER
listeners=...
advertised.listeners=...
listener.security.protocol.map=...
log.dirs=/data/kafka-logs
```

---

## ✅ 안정성 관련 설정 (권장)

```properties
min.insync.replicas=2
share.coordinator.state.topic.replication.factor=3
share.coordinator.state.topic.min.isr=2
offsets.topic.replication.factor=3
transaction.state.log.replication.factor=3
transaction.state.log.min.isr=2
```

---

## ✅ 생략 가능한 설정 요약

| 설정                                  | 생략 가능 | 기본값              | 비고                  |
| ----------------------------------- | ----- | ---------------- | ------------------- |
| controller.quorum.bootstrap.servers | ✅     | 없음               | 포맷 후 제거 권장          |
| inter.broker.listener.name          | ✅     | 추론               | PLAINTEXT만 쓰면 생략 가능 |
| num.recovery.threads.per.data.dir   | ✅     | 1                | 디스크 1개면 생략 OK       |
| socket.send/receive.buffer.bytes    | ✅     | 102400           | 네트워크 최적화 시 조정 가능    |
| socket.request.max.bytes            | ✅     | 104857600(100MB) | 메시지가 클 경우만 설정       |

---

필요 시 systemd 등록, 모니터링, ACL, 인증 등도 이어서 구성 가능.