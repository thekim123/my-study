## 🛠️ **4. Spring Boot 프로젝트 연동**
### ✅ **4.1 Spring Kafka 의존성 설정**
```groovy
implementation 'org.springframework.kafka:spring-kafka'  
implementation 'io.confluent:kafka-avro-serializer:7.3.0'  
implementation 'io.confluent:kafka-schema-registry-client:7.3.0'  
implementation 'org.apache.avro:avro:1.11.0'  
implementation 'org.springframework.kafka:spring-kafka-streams'
```

---
### ✅ **4.2 Spring Kafka 설정**
**`application.yml`:**
```yml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    consumer:
      group-id: video-processing-group
      auto-offset-reset: earliest
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    properties:
      schema.registry.url: http://localhost:8081\
```

---

## 🛠️ **2. 환경 상태 확인**

### ✅ **2.1 Docker 컨테이너 상태 확인**

```bash
docker ps
```

**확인해야 할 포트:**

- Kafka: `9092`, `29092`, `9999`
- Zookeeper: `2181`
- Schema Registry: `8081`
- Conduktor Console: `8080`
- PostgreSQL: `5432`
- Monitoring: `9009`, `9010`

---

### ✅ **2.2 Conduktor Console 접속**

1. 브라우저에서 `http://localhost:8080` 접속
    
2. **Admin 계정으로 로그인:**
    
    - **Email:** `admin@conduktor.io`
    - **Password:** `admin`
3. Kafka 클러스터가 정상적으로 연결되었는지 확인:
    
    - 클러스터 상태 (`My Local Kafka Cluster`)
    - 토픽 목록 확인

---

## 🛠️ **3. 토픽 및 메시지 테스트**

### ✅ **3.1 Kafka 토픽 생성**

**CLI에서 Kafka 토픽 생성:**

```bash
docker exec -it kafka kafka-topics.sh --create --topic video-processing-request --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
docker exec -it kafka kafka-topics.sh --create --topic video-processing-response --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```

**확인:**

```bash
docker exec -it kafka kafka-topics.sh --list --bootstrap-server localhost:9092
```

---

### ✅ **3.2 메시지 프로듀서 & 컨슈머 테스트**

**Producer 실행 (메시지 보내기):**

```bash
docker exec -it kafka kafka-console-producer.sh --broker-list localhost:9092 --topic video-processing-request
```

**메시지 입력:**

```
Hello from Kafka via Docker!
```

**Consumer 실행 (메시지 받기):**

```bash
docker exec -it kafka kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic video-processing-request --from-beginning
```

✅ **출력 확인:**

```
Hello from Kafka via Docker!
```

---

## 🛠️ **4. Spring Boot 프로젝트 연동**

### ✅ **4.1 Spring Boot Dependencies**

**`pom.xml`에 추가:**

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.avro</groupId>
    <artifactId>avro</artifactId>
</dependency>
<dependency>
    <groupId>io.confluent</groupId>
    <artifactId>kafka-schema-registry-client</artifactId>
    <version>7.3.0</version>
</dependency>
```

---

### ✅ **4.2 Spring Kafka 설정**

**`application.yml`:**

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    consumer:
      group-id: video-processing-group
      auto-offset-reset: earliest
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    properties:
      schema.registry.url: http://localhost:8081
```

---

### ✅ **4.3 Kafka Producer/Consumer 예제**

**Producer 예제:**

```java
@Autowired
private KafkaTemplate<String, String> kafkaTemplate;

public void sendMessage(String message) {
    kafkaTemplate.send("video-processing-request", message);
}
```

**Consumer 예제:**

```java
@KafkaListener(topics = "video-processing-request", groupId = "video-processing-group")
public void listen(String message) {
    System.out.println("Received message: " + message);
}
```

---

## 🛠️ **5. Python 엔진 연동 (OpenCV + Kafka 라이브러리)**

Python 환경에서 Kafka 메시지를 처리합니다.

### ✅ **5.1 Python Dependencies**

**`requirements.txt`:**

```
kafka-python==2.0.2
opencv-python==4.5.3.56
```

### ✅ **5.2 Kafka Producer (Python)**

```python
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers='localhost:9092')
producer.send('video-processing-request', b'Hello from Python Producer!')
producer.flush()
```

### ✅ **5.3 Kafka Consumer (Python)**

```python
from kafka import KafkaConsumer

consumer = KafkaConsumer('video-processing-request', bootstrap_servers='localhost:9092')
for message in consumer:
    print(f"Received: {message.value.decode('utf-8')}")
```

---

## 🚦 **6. 모니터링 및 알림 설정**

### ✅ **6.1 Grafana 설정**

1. Grafana 설치 및 Prometheus 연동
2. Kafka 대시보드 추가 (`7589`)

### ✅ **6.2 Alert 설정**

- Lag(지연) 임계값 초과 시 알림
- 브로커 다운 시 Slack/Email 알림

---

## 🎯 **7. 다음 단계**

1. **Spring Boot와 Python 엔진 간 메시지 연동 테스트**
2. **Conduktor Console에서 Kafka 클러스터 및 토픽 모니터링**
3. **Grafana를 통해 실시간 메트릭 시각화**

---

### 📊 **지금까지 진행한 단계 정리**

1. Kafka 클러스터 (Zookeeper 포함) Docker로 설정 ✅
2. Conduktor Console + Monitoring 설정 ✅
3. Kafka 토픽 및 메시지 테스트 ✅
4. Spring Boot 연동 준비 ✅
5. Python 엔진 환경 구축 ✅

---

**어떤 부분을 먼저 진행할까요?** 😊🚀