---
Date: 2024-12-16
Category: Auth
---
JWT 기반 인증 시스템에서 **Access Token**과 **Refresh Token**을 사용하는 로직은 보통 다음과 같은 단계로 설계된다. 이때, Access Token이 만료되었을 때의 처리는 조금 더 세밀한 설계가 필요하다. 아래에서 상황별로 자세히 알아보자.

---

### 1. **일반적인 요청 흐름**

- 클라이언트는 **Access Token**을 `Authorization` 헤더의 `Bearer` 토큰으로 포함해서 서버로 요청을 보낸다.
- 서버의 **Spring Security 필터**는 이 Access Token을 검증하고, 유효하면 요청을 처리합니다.
- Refresh Token은 일반적인 요청에서 사용되지 않고, 보통 클라이언트가 저장하고 있다가 **Access Token이 만료된 경우에만 사용**된다.

---

### 2. **Access Token이 만료된 경우**

- **클라이언트 요청 시 Access Token이 만료**되었다면, 서버는 보통 `401 Unauthorized` 응답을 반환한다.
- 클라이언트는 이 응답을 보고 **Refresh Token을 사용해서 새로운 Access Token을 요청**한다.  
    예: `/auth/refresh` 같은 별도의 엔드포인트를 사용.

---

### 3. **Refresh Token 요청 처리 흐름**

#### Refresh Token 요청 자체는 별도의 로직으로 처리된다:

- Refresh Token 요청은 보통 `Authorization` 헤더나 `Cookie`로 Refresh Token을 서버로 보낸다.
- 서버는 **Refresh Token을 검증**한다:
    - 유효하다면 새로운 Access Token과 Refresh Token을 발급한다.
    - 만료되었다면 클라이언트에게 `401 Unauthorized` 응답을 보내고, 사용자는 재인증을 해야 한다.
- 발급된 새로운 Access Token과 Refresh Token을 클라이언트에 반환한다.
    - 클라이언트는 새 Access Token으로 기존 요청을 다시 시도한다.

---

### 4. **Spring Security에서 Refresh Token을 처리하려면?**

Spring Security의 필터는 기본적으로 **Access Token** 검증에만 사용된다. 하지만, Refresh Token은 별도의 엔드포인트와 로직으로 처리해야 한다.

1. **별도의 엔드포인트를 만든다**
    
    - 예: `/auth/refresh`  
        이 엔드포인트는 **Spring Security 필터 체인 바깥에 위치**해야 한다.
    - 이 요청은 Access Token 없이도 동작해야 하기 때문이다.
2. **Refresh Token을 검증하고 새로운 Access Token 발급**
    
    - Refresh Token의 유효성을 확인하고 새로운 Access Token과 Refresh Token을 발급한다.
    - 예를 들어, 아래처럼 컨트롤러에서 처리:

```java
@RestController
@RequestMapping("/auth")
public class AuthController {

    @PostMapping("/refresh")
    public ResponseEntity<?> refreshToken(@RequestHeader("Authorization") String refreshTokenHeader) {
        String refreshToken = refreshTokenHeader.replace("Bearer ", "");

        // Refresh Token 검증 로직
        if (isValidRefreshToken(refreshToken)) {
            // 새 Access Token 및 Refresh Token 발급
            String newAccessToken = generateAccessToken();
            String newRefreshToken = generateRefreshToken();

            // 반환
            return ResponseEntity.ok(new Tokens(newAccessToken, newRefreshToken));
        } else {
            // 만료된 경우 재인증 요청
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body("Refresh token expired");
        }
    }
}
```

---

### 5. **Access Token 만료 후 요청 처리**

#### 질문: "재발급된 경우 요청도 다 처리해주는건가?"

- 서버는 기본적으로 Access Token이 만료된 요청은 실패로 처리하고, 클라이언트가 새 Access Token을 받은 후 **다시 요청**해야 한다.
- 하지만 클라이언트 요청을 **자동 재시도**하도록 설계할 수도 있다:
    1. 클라이언트가 Access Token 만료 시 Refresh Token 요청을 보내고,
    2. 새 Access Token을 받은 뒤 이전 요청을 다시 전송.

---

### 6. **Spring Security 필터와 Refresh Token**

#### Refresh Token은 Access Token과 다르게 일반적인 인증 흐름에서 처리되지 않는다.

- Spring Security 필터는 Access Token의 유효성을 검사하고, 유효하지 않은 경우 `401` 응답을 반환한다.
- Refresh Token은 일반적으로 Access Token과 다른 엔드포인트에서 처리되므로, Spring Security 필터 체인과는 별도로 구현해야 한다.

#### 만약 Access Token 만료 상태를 필터에서 처리하고 싶다면?

- Custom Filter를 추가해서 Access Token 만료 시 `Refresh Token`을 자동으로 검증하거나 처리할 수도 있다.
- 하지만 **보안상 좋은 설계는 아닙니다**. Refresh Token은 인증 로직과 별도로 처리하는 것이 권장된다.

---

### 7. **정리**

- **Access Token 만료** 시 요청은 실패(`401`)로 끝나고, Refresh Token을 사용해 새로운 Access Token을 받아야 한다.
- Refresh Token 처리는 보통 Spring Security 필터와 별개로 구현된 **인증 엔드포인트**를 통해 이뤄진다.
- **자동 재시도**는 클라이언트의 책임으로 두는 것이 일반적이며, 서버는 Access Token과 Refresh Token의 역할을 명확히 분리해야 한다.

이 설계는 보안과 확장성 면에서 유리하며, Spring Security의 기존 구조와도 잘 맞아떨어진다.



즉, **자동 재시도와 토큰 관리**는 일반적으로 클라이언트가 담당하고, 서버는 Access Token과 Refresh Token을 다루는 명확한 API만 제공하면 된다. 하지만 클라이언트 입장에서는 매번 토큰 검증과 재발급 처리를 수동으로 하면 번거롭기 때문에, 개발이 편리하도록 **자동화된 토큰 관리**가 필요하다. 아래에서 어떻게 설계하면 좋을지 단계별로 알아보자.

---

### 1. **클라이언트의 기본 구조**

- 클라이언트는 Access Token과 Refresh Token을 적절히 저장하고, 요청을 보낼 때 Access Token을 사용한다.
- Access Token이 만료되었을 때는 Refresh Token을 사용해 새로운 Access Token을 받아와야 한다.

---

### 2. **자동 토큰 갱신 로직 설계**

#### 2.1 **요청 인터셉터 사용**

- 클라이언트(예: React, Axios, Retrofit 등)는 요청을 가로채는 **인터셉터(Interceptor)**를 사용해서 모든 HTTP 요청 전에 Access Token을 검증하거나 추가 로직을 수행할 수 있다.
- 만약 Access Token이 만료된 경우:
    1. Refresh Token으로 새로운 Access Token을 요청.
    2. 새로운 Access Token으로 원래 요청을 다시 전송.

#### Axios 예제:

```javascript
import axios from 'axios';

const apiClient = axios.create({
  baseURL: 'https://example.com/api',
  timeout: 5000,
});

// Refresh Token 요청 함수
async function refreshAccessToken() {
  const refreshToken = localStorage.getItem('refreshToken'); // Refresh Token 가져오기
  try {
    const response = await axios.post('/auth/refresh', {}, {
      headers: {
        Authorization: `Bearer ${refreshToken}`,
      },
    });
    const { accessToken, refreshToken: newRefreshToken } = response.data;
    // 새 토큰 저장
    localStorage.setItem('accessToken', accessToken);
    localStorage.setItem('refreshToken', newRefreshToken);
    return accessToken;
  } catch (error) {
    console.error('Refresh token expired. Please re-authenticate.');
    throw error;
  }
}

// 요청 인터셉터 설정
apiClient.interceptors.request.use(async (config) => {
  let accessToken = localStorage.getItem('accessToken');
  if (isTokenExpired(accessToken)) { // 토큰 만료 여부 확인
    try {
      accessToken = await refreshAccessToken();
    } catch (error) {
      // 재인증 필요 시 에러 처리
      return Promise.reject(error);
    }
  }
  // Access Token을 Authorization 헤더에 추가
  config.headers.Authorization = `Bearer ${accessToken}`;
  return config;
}, (error) => {
  return Promise.reject(error);
});

// 토큰 만료 여부 확인 함수 (JWT의 exp 확인)
function isTokenExpired(token) {
  if (!token) return true;
  const payload = JSON.parse(atob(token.split('.')[1]));
  const currentTime = Math.floor(Date.now() / 1000);
  return payload.exp < currentTime;
}

export default apiClient;
```

---

### 3. **서버와의 협력**

#### 서버가 클라이언트를 편리하게 개발할 수 있도록 제공해야 하는 요소:

1. **Refresh Token API**
    
    - Refresh Token 검증 및 Access Token 재발급.
    - 클라이언트가 만료된 Access Token을 대체할 수 있는 명확한 방식 제공.
2. **명확한 HTTP 상태 코드**
    
    - Access Token이 만료되었음을 명확히 알리기 위해 **401 Unauthorized** 응답을 반환.
    - 응답 본문에 만료 사유를 추가하면 디버깅이 쉬워짐.
3. **만료 시간(exp) 정보**
    
    - JWT 토큰에 `exp` 필드를 포함해 클라이언트가 만료 여부를 직접 확인할 수 있도록 지원.

---

### 4. **개발자 편의성**

#### 인증/인가를 맡지 않은 개발자들이 편리하게 요청을 보내도록 하기 위해:

- **HTTP 클라이언트를 추상화**: 인증/인가 처리가 클라이언트 코드의 각 부분에서 중복되지 않도록, 위에서 설명한 **인터셉터**를 적용한 HTTP 클라이언트를 제공.
    
    - 예: `apiClient.get('/resource')`처럼 간단히 호출해도 토큰 처리와 재요청이 내부적으로 자동 처리되도록.
- **에러 핸들링 통합**: 인터셉터나 전역 에러 처리 로직에서, 토큰 만료 시 사용자 재로그인 화면으로 전환하는 등의 공통 처리를 추가.
    

---

### 결론

클라이언트가 인증과 관련된 복잡한 처리를 자동화하려면 **요청 인터셉터**를 활용하는 것이 좋다.  
서버에서는 클라이언트가 이러한 처리를 쉽게 구현할 수 있도록 명확한 API와 에러 상태 코드를 제공해보자.  
이 방식으로 협력하면, 인증/인가 로직을 맡지 않은 클라이언트 개발자들도 편리하게 작업할 수 있을 것이다! 🙌