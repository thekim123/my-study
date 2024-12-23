## 🛠️ **1. 인가 절차 흐름**

### ✅ **1️⃣ JWT (Access Token) 검증 (Stateless)**

1. **사용자 요청:**
    
    - 사용자는 `Authorization: Bearer <Access Token>`을 포함해 요청을 보냅니다.
2. **서버 검증:**
    
    - JWT 서명(Signature) 검증.
    - 만료 시간(`exp`) 확인.
3. **검증 결과:**
    
    - **만료되지 않음:** 요청 통과 ✅
    - **만료됨:** `Session ID`를 사용해 리프레시 절차로 이동 🔄

---

### ✅ **2️⃣ Access Token 만료 시 (Stateful 검증)**

1. **클라이언트 요청:**
    
    - `Session ID`를 포함해 `/refresh` 엔드포인트로 요청합니다.
        - **쿠키 방식:** `SESSIONID=<session-id>`
        - **헤더 방식:** `X-Session-ID: <session-id>`
2. **Redis에서 Session ID 검증:**
    
    - `session_id`가 Redis에 존재하는지 확인.
    - 만료 여부 및 세션 유효성 검증.
3. **검증 결과:**
    
    - **유효:** 새로운 Access Token 발급 및 세션 만료 시간 갱신 ✅
    - **무효:** `401 Unauthorized` 반환 🚫
4. **갱신:**
    
    - 새로운 Access Token 발급 (`JWT`)
    - `Session ID`의 유효 시간 연장 (`expireAt` 갱신)
5. **응답:**
    
    - 새로운 Access Token을 사용자에게 반환.
    - 기존 Session ID는 그대로 유지.

---

## 📝 **2. 단계별 예시**

### **2️⃣ Access Token 만료 → Session ID 검증**

```java
@RequestMapping("/refresh")
public ResponseEntity<?> refreshToken(@CookieValue("SESSIONID") String sessionId) {
    String sessionData = redisTemplate.opsForValue().get("spring:session:sessions:" + sessionId);
    
    if (sessionData == null) {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body("Invalid session");
    }
    
    // 새로운 Access Token 발급
    String newAccessToken = jwtService.generateAccessToken();
    
    // Redis의 세션 만료 시간 갱신 (예: 1시간 연장)
    redisTemplate.expire("spring:session:sessions:" + sessionId, Duration.ofHours(1));
    
    return ResponseEntity.ok(Map.of("accessToken", newAccessToken));
}
```

---

## 🔄 **3. 인가 절차 다이어그램**

```plaintext
사용자 → 서버: 요청 + Access Token
  ├── JWT 검증 (Stateless)
  │     ├── 유효: 통과 ✅
  │     ├── 만료: Session ID 확인 🔄
  │
  ├── Session ID로 Redis 검증 (Stateful)
  │     ├── 유효: 새로운 Access Token 발급 + Redis 만료 시간 갱신 ✅
  │     ├── 무효: 401 Unauthorized 🚫
  │
  └── 사용자 → 새로운 Access Token 수신
```

---

## 📊 **4. 데이터 흐름 예시**

| 단계    | 요청/검증                     | 설명                                   |
| ----- | ------------------------- | ------------------------------------ |
| **1** | **JWT 검증**                | Access Token의 서명 및 만료 시간 검증          |
| **2** | **Access Token 만료**       | 만료 시 `/refresh` 요청                   |
| **3** | **Session ID 검증 (Redis)** | Redis에서 Session ID 검증                |
| **4** | **새 Access Token 발급**     | Session ID가 유효하면 새로운 Access Token 발급 |
| **5** | **세션 시간 갱신**              | Redis의 Session ID 만료 시간 갱신           |
| **6** | **사용자 전달**                | 새 Access Token 전달                    |

---

## ✅ **5. 중요 포인트 정리**

1. **Access Token 검증 (Stateless):**
    
    - JWT 서명 및 만료 시간 확인
    - 만료되지 않았다면 통과
2. **Access Token 만료 (Stateful):**
    
    - Redis에서 `Session ID` 검증
    - 유효하면 Access Token 재발급 및 Redis 세션 시간 갱신
3. **Session ID 시간 갱신:**
    
    - Access Token이 갱신될 때 Redis의 `Session ID` 만료 시간도 갱신
    - 세션 만료 시 사용자는 재로그인 필요
4. **세션 ID는 유지됨:**
    
    - Access Token은 만료 후 새로 발급되지만, Session ID는 그대로 유지

---

## 🚀 **6. 결론**

1. **JWT 검증 (Stateless):** 빠르고 효율적이며 확장성 높음.
2. **Session ID 검증 (Stateful):** 만료 시 Redis를 통해 세션 검증 및 시간 갱신.
3. **매 요청마다 Session ID를 갱신할 필요 없음:**
    - Access Token만 주기적으로 갱신.
    - Session ID는 만료될 때까지 유지.

---

### 📌 **최종 절차:**

1. **Access Token 확인:** 만료 여부 및 유효성 확인.
2. **만료 시 Session ID 확인:** Redis에서 상태 검증.
3. **새 Access Token 발급:** Redis의 세션 만료 시간 연장.
4. **Session ID는 그대로 유지:** 새로운 Access Token만 반환.

이 방식은 **Stateless 검증의 효율성**과 **Stateful 검증의 보안성**을 동시에 충족하며, 복잡성도 최소화할 수 있습니다.

이제 인가 절차가 명확해졌나요? 추가적으로 궁금한 부분이 있다면 말씀해주세요! 😊

### 🚀 **Redis 기반 Session ID 설계**

**목표:**

1. 사용자는 여러 환경(브라우저, 모바일 등)에서 동시에 로그인할 수 있다.
2. 특정 정책에 따라 **1곳에서만 로그인**을 허용할 수도 있다.
3. 각 환경의 세션을 효율적으로 관리할 수 있어야 한다.
4. `Session ID`는 Redis에서 만료 시간과 함께 관리된다.

---

# 🛠️ **1. Redis Key-Value 설계**

### ✅ **1️⃣ Key 구조 설계**

|**Key 패턴**|**Value**|**설명**|**예시**|
|---|---|---|---|
|`session:<userId>:<sessionId>`|세션 메타데이터(JSON)|개별 세션 정보 저장|`session:123:abcd1234`|
|`user:<userId>:sessions`|Hash (sessionId → TTL)|사용자의 모든 세션 관리|`user:123:sessions`|

---

### ✅ **2️⃣ Value 구조**

#### **1. 세션 메타데이터 (Key: `session:<userId>:<sessionId>`)**

```json
{
  "userId": "123",
  "sessionId": "abcd1234",
  "device": "Chrome",
  "ip": "192.168.0.1",
  "loginTime": "2024-12-23T10:00:00",
  "expireAt": "2024-12-23T11:00:00"
}
```

- **userId:** 사용자 ID
- **sessionId:** 세션 ID (UUID로 생성)
- **device:** 로그인 환경 정보 (브라우저, 모바일 등)
- **ip:** 로그인 IP
- **loginTime:** 로그인 시간
- **expireAt:** 만료 시간

#### **2. 사용자 세션 목록 (Key: `user:<userId>:sessions`)**

```json
{
  "abcd1234": "2024-12-23T11:00:00",
  "efgh5678": "2024-12-23T12:00:00"
}
```

- **Key:** sessionId
- **Value:** 해당 세션의 만료 시간

---

# 📊 **2. Redis 세션 관리 정책**

### ✅ **1️⃣ 로그인 시 처리 흐름**

1. **사용자 로그인 요청**
    
    - 사용자 인증 성공 시 `sessionId` 생성 (UUID)
    - Redis에 `session:<userId>:<sessionId>` 메타데이터 저장
    - `user:<userId>:sessions` 해시에 세션 ID와 만료 시간 저장
2. **1곳에서만 로그인 정책 (선택적 적용)**
    
    - `user:<userId>:sessions`에 기존 세션이 있으면 강제 로그아웃 (기존 세션 삭제)
    - 새 세션만 Redis에 저장

**예시 (Java, Spring)**

```java
public void createSession(String userId, String sessionId, String device, String ip) {
    String sessionKey = "session:" + userId + ":" + sessionId;
    String userSessionsKey = "user:" + userId + ":sessions";

    // 세션 메타데이터 저장
    redisTemplate.opsForValue().set(sessionKey, Map.of(
        "userId", userId,
        "sessionId", sessionId,
        "device", device,
        "ip", ip,
        "loginTime", LocalDateTime.now().toString(),
        "expireAt", LocalDateTime.now().plusHours(1).toString()
    ), Duration.ofHours(1));

    // 사용자 세션 목록에 추가
    redisTemplate.opsForHash().put(userSessionsKey, sessionId, LocalDateTime.now().plusHours(1).toString());
    redisTemplate.expire(userSessionsKey, Duration.ofHours(1));
}
```

---

### ✅ **2️⃣ Access Token 리프레시 시 처리**

1. 사용자가 `/refresh` 요청 시 `SESSIONID`를 전달.
2. Redis에서 `session:<userId>:<sessionId>`로 세션 정보 조회.
3. 유효하면 Access Token 발급 및 `expireAt` 갱신.
4. `user:<userId>:sessions`의 세션 만료 시간도 갱신.

```java
public boolean validateSession(String userId, String sessionId) {
    String sessionKey = "session:" + userId + ":" + sessionId;
    return redisTemplate.hasKey(sessionKey);
}

public void refreshSession(String userId, String sessionId) {
    String sessionKey = "session:" + userId + ":" + sessionId;
    String userSessionsKey = "user:" + userId + ":sessions";

    // 만료 시간 연장
    redisTemplate.expire(sessionKey, Duration.ofHours(1));
    redisTemplate.opsForHash().put(userSessionsKey, sessionId, LocalDateTime.now().plusHours(1).toString());
    redisTemplate.expire(userSessionsKey, Duration.ofHours(1));
}
```

---

### ✅ **3️⃣ 로그아웃 처리**

1. 사용자가 로그아웃 요청 시 해당 `sessionId`를 제거.
2. `session:<userId>:<sessionId>` 키 삭제.
3. `user:<userId>:sessions`에서 해당 `sessionId` 제거.

```java
public void logoutSession(String userId, String sessionId) {
    String sessionKey = "session:" + userId + ":" + sessionId;
    String userSessionsKey = "user:" + userId + ":sessions";

    redisTemplate.delete(sessionKey);
    redisTemplate.opsForHash().delete(userSessionsKey, sessionId);
}
```

---

### ✅ **4️⃣ 사용자 모든 세션 강제 만료 (모든 환경 로그아웃)**

1. `user:<userId>:sessions`의 모든 `sessionId` 조회.
2. 각 `sessionId`의 `session:<userId>:<sessionId>` 키를 삭제.
3. `user:<userId>:sessions` 키 삭제.

```java
public void logoutAllSessions(String userId) {
    String userSessionsKey = "user:" + userId + ":sessions";
    Map<Object, Object> sessions = redisTemplate.opsForHash().entries(userSessionsKey);

    for (Object sessionId : sessions.keySet()) {
        String sessionKey = "session:" + userId + ":" + sessionId;
        redisTemplate.delete(sessionKey);
    }

    redisTemplate.delete(userSessionsKey);
}
```

---

# ✅ **3. TTL (Time-To-Live, 만료 시간) 설정**

- `session:<userId>:<sessionId>` → **1시간**
- `user:<userId>:sessions>` → **1시간 (가장 늦게 만료되는 세션 기준으로 갱신)**

---

# 🚦 **4. 예상 시나리오**

### 🔄 **1. 다중 로그인 허용**

- 사용자는 다양한 환경에서 여러 개의 `sessionId`를 가질 수 있음.
- `user:<userId>:sessions` 해시에 각 `sessionId`가 저장됨.

### 🔄 **2. 1곳만 로그인 허용 (옵션)**

- 기존 `user:<userId>:sessions`를 확인.
- 존재하면 강제 로그아웃 후 새 `sessionId` 발급.

### 🔄 **3. Access Token 갱신**

- `sessionId`가 Redis에서 검증되면 Access Token 갱신 및 세션 TTL 갱신.

### 🔄 **4. 로그아웃**

- `sessionId`만 로그아웃 가능.
- 모든 세션 강제 만료도 가능.

---

# 📝 **5. 최종 정리**

1. **Key 구조:** `session:<userId>:<sessionId>` + `user:<userId>:sessions`
2. **다중 로그인 지원:** 여러 환경에서 동시 로그인 가능
3. **1곳 로그인 정책:** 기존 세션 제거 후 새 세션 발급
4. **Access Token 갱신:** Session ID를 통해 유효성 검증 및 TTL 갱신
5. **로그아웃:** 단일 세션 로그아웃, 모든 세션 강제 로그아웃 지원

---

이 설계를 기반으로 리프레시와 세션 관리를 구현한다면 확장성, 보안성, 유연성을 모두 확보할 수 있습니다.

**이제 바로 적용해보세요! 🚀** 추가적으로 궁금하거나 막히는 부분이 있다면 언제든지 이야기해주세요. 😊✨