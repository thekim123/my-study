
Session 데이터를 레디스에 저장해두고 공유하면 안전하고 좋을지도 모르겠다. 어차피 레디스에서의 탐색은 O(1)의 성능을 가지고 있으니 성능상의 이슈도 없을 것 같다. 그러나 이것은 조금 더 검토해봐야 한다.

---

## 🚀 **1. Redis 접근이 O(1)이라도 비용이 발생하는 이유**

### ✅ **1️⃣ 네트워크 비용 (Network Latency)**

- Redis는 보통 **네트워크를 통해 접근**한다.
- 요청(Request)과 응답(Response) 사이에는 항상 네트워크 레이턴시(Latency)가 존재한다.
- 아무리 Redis가 O(1)이라도, 네트워크를 타고 데이터를 주고받는 시간은 무시할 수 없다.

**예시:**

- 서버와 Redis가 같은 데이터센터에 있어도 **네트워크 왕복 시간 (Round-Trip Time, RTT)**이 발생한다.
- 트래픽이 많아질수록 네트워크 I/O 오버헤드가 누적된다.

---

### ✅ **2️⃣ I/O 오버헤드 (I/O Overhead)**

- Redis와 통신은 주로 **TCP/IP** 프로토콜을 사용한다.
- 이로 인해 요청/응답의 **패킷 오버헤드**가 발생한다.
- 작은 요청이라도 여러 번 발생하면 Redis가 처리해야 할 I/O 오버헤드가 쌓인다.

---

### ✅ **3️⃣ Redis의 한계 (Single-threaded Model)**

- Redis는 **싱글 스레드(single-threaded) 아키텍처**로 동작한다.
- O(1) 접근이 가능하더라도, 모든 요청은 **단일 스레드**에서 순차적으로 처리된다.
- 트래픽이 많아지면 Redis 자체가 병목(Bottleneck)으로 작동할 가능성이 있다.

---

### ✅ **4️⃣ 병렬 처리 (Concurrency) 문제**

- 서버가 Redis에 접근할 때, 요청이 동시에 많이 발생하면 **큐잉(Queuing) 현상**이 발생할 수 있다.
- Redis는 단일 스레드이기 때문에 한 번에 하나의 요청만 처리한다.
- 다량의 요청이 동시에 들어오면 대기 시간(Queue Time)이 발생할 수 있다.

---

### ✅ **5️⃣ 캐시 미스 (Cache Miss)**

- Redis는 빠른 접근을 제공하지만, **데이터가 캐시에 없으면** 접근 시간이 더 길어질 수 있다.
- 이런 상황은 주로 TTL이 짧거나, 데이터가 빈번하게 만료될 때 발생한다.

---

## ⚖️ **3. Access Token vs Redis 검증 비교**

|**기준**|**Access Token (Stateless)**|**Session ID (Stateful with Redis)**|
|---|---|---|
|**확장성**|뛰어남 (서버 로컬 검증)|제한적 (중앙 Redis 의존)|
|**성능**|빠름 (네트워크 요청 없음)|네트워크 비용 및 Redis I/O 발생|
|**보안**|탈취 시 제한된 리스크|탈취 시 Redis 검증 가능|
|**복구**|Stateless 복구 쉬움|Redis 다운 시 검증 불가|
|**복잡성**|낮음|높음|

---

## 🔄 **4. 최적화 방안**

### ✅ **1️⃣ Access Token을 Stateless하게 검증**

- Access Token은 모든 요청에서 **Stateless 검증 (서명 및 만료 시간 확인)**을 수행합니다.
- 네트워크 및 Redis 부하 없음.

### ✅ **2️⃣ 민감한 작업에만 Redis 검증 추가**

- 모든 요청을 Redis로 보내지 않습니다.
- 중요한 요청(예: 비밀번호 변경, 결제 API)만 Session ID를 Redis에서 검증합니다.

### ✅ **3️⃣ Redis Connection Pool 사용**

- Redis 접근 시 커넥션 풀(Connection Pool)을 사용해 네트워크 오버헤드를 최소화합니다.

### ✅ **4️⃣ Session ID 검증 최적화**

- Redis에서 세션 검증을 단순화합니다.
- 필요 이상으로 복잡한 데이터 구조를 사용하지 않습니다.

### ✅ **5️⃣ 캐시 계층 추가**

- 세션 정보를 Redis 앞단에 로컬 캐시에 저장하여 Redis 부하를 줄입니다.
- **예시:** `Caffeine Cache`, `Guava Cache` 등.

---

## 🚦 **6. 최종 정리**

1. **Access Token (Stateless) 검증 우선:**
    
    - 서명(Signature)과 만료 시간(`exp`)을 검증.
    - Redis 접근 최소화.
2. **Session ID (Stateful) 검증은 민감한 API에만 적용:**
    
    - 비밀번호 변경, 결제 API 등 중요 API 요청에만 Redis 검증 수행.
3. **Access Token Rotation 사용:**
    
    - 요청마다 Access Token을 새로 발급해 탈취 리스크 최소화.
4. **Redis 최적화:**
    
    - Connection Pool 사용.
    - 캐시 계층 도입 (예: Caffeine Cache).

---
## ✅ **7. 결론**

- **Redis가 O(1)이라도 네트워크 비용과 I/O 오버헤드는 무시할 수 없다.**
- 모든 요청에서 Redis 검증을 수행하는 것은 비효율적이다.
- 따라서 **Stateless 검증 (Access Token)**을 우선 사용하고, **민감한 API에서만 Stateful 검증 (Session ID, Redis)**을 수행해야 한다.

### ✅ **3️⃣ 디바이스 ID를 서버가 직접 확인**

- 디바이스 바인딩은 **클라이언트가 전달하는 값만을 믿지 않습니다.**
- 서버는 클라이언트가 보낸 `deviceId` 외에도 **HTTP Header, IP 주소, User-Agent** 등을 확인하여 추가 검증을 수행합니다.

**예시 (검증 흐름):**

1. Access Token의 `deviceId` 확인
2. HTTP Header의 `deviceId` 확인
3. User-Agent 비교
4. IP 주소 비교