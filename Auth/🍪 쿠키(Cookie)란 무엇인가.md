
## ✅ **1. 쿠키란?**

**쿠키(Cookie)**는 **웹 브라우저에 저장되는 작은 데이터 조각**이다.  
서버와 클라이언트(브라우저) 간에 **상태를 유지하거나 사용자 정보를 저장**하는 데 사용된다.

---

## 🧠 **2. 쿠키의 기본 개념**

### 📌 **2.1 쿠키의 역할**

- **세션 관리:** 로그인 정보, 사용자 설정 등을 저장.
- **개인화:** 사용자의 선호도나 테마 설정 저장.
- **추적 및 분석:** 사용자의 방문 기록, 클릭 패턴 등을 추적.

---

### 📌 **2.2 쿠키의 구조**

쿠키는 **이름(name), 값(value), 속성(attributes)**으로 구성된다.

```http
Set-Cookie: token=eyJhbGciOiJIUzI1NiIsIn...; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=3600
```

| **속성**       | **설명**                                  |
| ------------ | --------------------------------------- |
| **Name**     | 쿠키의 이름 (`token`)                        |
| **Value**    | 쿠키에 저장된 값 (`eyJhbGciOiJIUzI1NiIsIn...`) |
| **Domain**   | 쿠키가 전송될 도메인 (`example.com`)             |
| **Path**     | 쿠키가 전송될 경로 (`/`)                        |
| **Max-Age**  | 쿠키의 유효 시간 (초 단위, `3600` → 1시간)          |
| **Expires**  | 쿠키 만료 날짜 (`Wed, 21 Oct 2024`)           |
| **Secure**   | HTTPS를 통해서만 전송                          |
| **HttpOnly** | JavaScript로 접근 불가                       |
| **SameSite** | CSRF 공격 방지 (`Strict`, `Lax`, `None`)    |

---

## 🔑 **3. 쿠키의 종류**

### ✅ **3.1 세션 쿠키 (Session Cookie)**

- 브라우저를 닫으면 사라짐.
- **Expires**나 **Max-Age** 속성이 없음.
- 주로 **로그인 세션** 관리에 사용.

### ✅ **3.2 지속 쿠키 (Persistent Cookie)**

- 브라우저를 닫아도 유지됨.
- **Expires**나 **Max-Age** 속성이 설정됨.
- 주로 **자동 로그인**, **사용자 설정** 저장에 사용.

### ✅ **3.3 보안 쿠키 (Secure Cookie)**

- **Secure** 속성이 설정됨.
- **HTTPS** 통신에서만 전송됨.

### ✅ **3.4 HttpOnly 쿠키 (HttpOnly Cookie)**

- **JavaScript에서 접근할 수 없음.**
- **XSS (Cross-Site Scripting)** 공격 방지.

---

## 🛡️ **4. 쿠키의 주요 속성**

### ✅ **4.1 Secure**

- HTTPS 연결에서만 쿠키가 전송된다.
- **중간자 공격 (Man-in-the-Middle, MITM)** 방지.

### ✅ **4.2 HttpOnly**

- JavaScript를 통해 쿠키에 접근할 수 없다.
- **XSS (Cross-Site Scripting)** 공격 방지.

### ✅ **4.3 SameSite**

- **CSRF (Cross-Site Request Forgery)** 공격 방지.
- **옵션:**
    - `Strict`: 동일 사이트 내 요청만 쿠키 전송.
    - `Lax`: 동일 사이트 + 링크 클릭 시 쿠키 전송.
    - `None`: 모든 요청에 쿠키 전송 (반드시 `Secure` 필요).

---

## 🌐 **5. 쿠키의 동작 방식**

### ✅ **5.1 쿠키 설정 (Set-Cookie)**

서버가 쿠키를 설정할 때 브라우저에 `Set-Cookie` 헤더를 보낸다.

**예시 (서버 → 클라이언트):**

```http
HTTP/1.1 200 OK
Set-Cookie: sessionId=abc123; HttpOnly; Secure; Path=/; Max-Age=3600
```

### ✅ **5.2 쿠키 전송**

- 브라우저는 같은 도메인/경로로 요청을 보낼 때 자동으로 쿠키를 포함한다.

**예시 (클라이언트 → 서버):**

```http
GET /dashboard
Cookie: sessionId=abc123
```

---

## 🔄 **6. 쿠키와 JWT (JSON Web Token)**

### ✅ **6.1 JWT를 쿠키에 저장할 때의 장단점**

|**구분**|**로컬 스토리지 (LocalStorage)**|**HttpOnly Secure 쿠키**|
|---|---|---|
|**XSS 공격**|취약|안전|
|**CSRF 공격**|안전|`SameSite`로 방어|
|**브라우저 접근**|JavaScript 접근 가능|JavaScript 접근 불가|
|**보안 수준**|낮음|높음|
|**전송 방식**|수동으로 헤더에 추가 필요|자동으로 전송|

### ✅ **6.2 쿠키 vs 세션**

|**구분**|**쿠키 (Cookie)**|**세션 (Session)**|
|---|---|---|
|**저장 위치**|브라우저|서버|
|**보안**|보안 취약점 존재|상대적으로 안전|
|**속도**|빠름|느림|
|**유효성**|유효 시간 설정 가능|브라우저 종료 시 만료|

---

## 🧠 **7. 쿠키의 보안 취약점**

### ⚠️ **7.1 XSS (Cross-Site Scripting)**

- JavaScript를 통해 쿠키 접근 → 탈취.

**예방:**

- `HttpOnly` 속성 사용.

### ⚠️ **7.2 CSRF (Cross-Site Request Forgery)**

- 공격자가 사용자의 쿠키를 사용해 악성 요청 전송.

**예방:**

- `SameSite` 속성 사용.

### ⚠️ **7.3 중간자 공격 (MITM)**

- HTTP 통신에서 쿠키가 노출.

**예방:**

- `Secure` 속성 사용.
- HTTPS 사용 필수.

---

## 🚀 **8. 쿠키 사용 Best Practice**

1. **JWT는 HttpOnly, Secure 쿠키에 저장.**
2. **SameSite=Strict**로 CSRF 방지.
3. **HTTPS 강제 사용.**
4. **민감한 정보는 쿠키에 저장 금지.**
5. **유효 시간(`Max-Age`)을 최소화.**

---

## 🎯 **9. 결론**

- **HttpOnly + Secure + SameSite** 설정은 필수!
- 쿠키는 웹 보안을 위해 신중하게 사용해야 한다.
- 민감한 정보는 쿠키에 저장하지 말고, 토큰은 **HttpOnly Secure 쿠키**를 사용하자.

---

## 📚 **10. 추가 학습**

- [MDN Web Docs - 쿠키](https://developer.mozilla.org/ko/docs/Web/HTTP/Cookies)
- [OWASP - XSS 취약점](https://owasp.org/www-community/attacks/xss/)

---