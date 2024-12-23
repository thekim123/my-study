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