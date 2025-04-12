좋아, 핵심 질문 나왔다 🔥  
**"Google OAuth2 클라이언트 등록"**, 즉 **구글에 내 애플리케이션 등록해서 `client-id`, `client-secret` 받는 과정**이 필요하지.

아래 단계 그대로 따라 하면 돼 👇

---

## ✅ 1. 구글 클라우드 콘솔 접속

👉 [https://console.cloud.google.com/](https://console.cloud.google.com/)  
구글 계정으로 로그인 (Gmail 있으면 돼)

---

## ✅ 2. 새 프로젝트 만들기

왼쪽 상단 드롭다운 > "프로젝트 만들기"

- **이름**: 너 프로젝트 이름
    
- 조직/위치는 생략해도 됨 (개인 계정이면)
    

---

## ✅ 3. OAuth 동의 화면 설정
### 🎯 1단계: [OAuth 동의 화면] 설정 먼저!

**왼쪽 사이드바** → `API 및 서비스 > OAuth 동의 화면`

- **앱 사용자 유형 선택**  
    → **외부** 선택 (로그인할 사람이 내 회사 사람만 아니라면 외부 선택해야 함)
    
- **앱 정보 입력**
    
    - 앱 이름
        
    - 사용자 지원 이메일
        
    - 개발자 연락처 이메일
        
- **앱 도메인**은 없어도 됨 (개발 중이라면 비워도 무방)
    
- **범위(Scopes)** → `기본`만 선택되어 있어도 가능 (Google API 접근이 필요하면 나중에 추가 가능)
    
- **테스트 사용자**:
    
    - 네가 실제 구글 로그인을 테스트할 계정을 넣어줘야 해 (예: 네 Gmail 주소)
        
    - 이걸 안 하면 로그인 시 403 에러 나.
        
- 완료 → 저장
    

---

### 🎯 2단계: OAuth 클라이언트 만들기

이제 이걸 해야 “사용자 유형이 없음” 같은 메시지가 안 나와

**왼쪽 사이드바** → `사용자 인증 정보 > 사용자 인증 정보 만들기 > OAuth 클라이언트 ID`

- **앱 유형**: `웹 애플리케이션`
    
- **이름**: 아무거나
    
- **승인된 리디렉션 URI** 추가:
    
    ```
    http://localhost:8080/login/oauth2/code/google
    ```
    
- 만들기 → **클라이언트 ID / 시크릿 복사**

---

## ✅ 4. OAuth2 클라이언트 ID 생성

왼쪽 메뉴 → **사용자 인증 정보**  
→ "사용자 인증 정보 만들기" > **OAuth 클라이언트 ID**

- **앱 유형 선택**: 웹 애플리케이션
    
- **이름**: 아무거나
    
- **승인된 리디렉션 URI** 추가:
    
    ```
    http://localhost:8080/login/oauth2/code/google
    ```
    
    👉 Spring Security가 내부적으로 리디렉션 받을 기본 주소야 (포트/도메인 맞춰서 수정 가능)
    

---

## ✅ 5. `client-id`, `client-secret` 복사

생성되면 **클라이언트 ID / 클라이언트 보안 비밀번호**가 나와  
→ `application.yml`에 복사해넣기:

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: 복붙
            client-secret: 복붙
            redirect-uri: "{baseUrl}/login/oauth2/code/{registrationId}"
            scope:
              - profile
              - email
```

---

## 🔐 참고: 도메인 바뀌면 redirect URI도 바꿔야 해

예: 실제 서비스 도메인이 `https://yourapp.com`이면

```
https://yourapp.com/login/oauth2/code/google
```

이렇게 등록해야 로그인 성공 후 리디렉션이 돼.

---

다 만들고 나면, **로그인 버튼 클릭 → `/oauth2/authorization/google`로 이동**시키기만 하면,  
Spring이 자동으로 구글 로그인 페이지로 보내고 인증 흐름을 이어서 처리해줘.

---

필요하면 캡처 이미지랑 같이 정리해줄 수도 있어!  
실제로 로그인 연결해서 JWT까지 완성하고 싶으면 거기까지도 같이 가자 🙌


좋아, 지금 막혔다면 아마 **OAuth 동의 화면** 설정을 아직 안 해서야.  
OAuth2를 사용하려면 **클라이언트 ID를 만들기 전에** 무조건 "OAuth 동의 화면"부터 먼저 설정해야 돼.

---


**백엔드(Spring Security)**가 **구글 로그인 전체를 처리**하는 구조로 가보자.
그럼 `oauth2Login()` 기반으로 **백엔드에서 인가 코드 받기부터 access token 교환, 유저 정보 조회까지** 전부 처리하는 걸로 가자.

---

## ✅ 목표 구조 (백엔드가 다 처리)

1. 프론트는 로그인 버튼만 있음 → 백엔드에 요청
    
2. 백엔드가 **구글 로그인 페이지로 리디렉션**
    
3. 사용자가 구글 로그인하면, **구글이 백엔드에 인가 코드 전달**
    
4. 백엔드가 **access token 받고 userinfo 조회**
    
5. 백엔드에서 JWT 발급 → 프론트에 전달 (or 세션 유지)
    

---

## ✅ Step-by-step 구성

### 1. `application.yml` 설정

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: YOUR_CLIENT_ID
            client-secret: YOUR_CLIENT_SECRET
            redirect-uri: "{baseUrl}/login/oauth2/code/{registrationId}"
            scope:
              - profile
              - email
        provider:
          google:
            authorization-uri: https://accounts.google.com/o/oauth2/v2/auth
            token-uri: https://oauth2.googleapis.com/token
            user-info-uri: https://www.googleapis.com/oauth2/v3/userinfo
```

---

### 2. 로그인 URL

**프론트엔드는 이 링크로 리디렉트만 시키면 됨**

```
GET /oauth2/authorization/google
```

---

### 3. 유저 정보를 처리할 서비스

```java
@Service
public class CustomOAuth2UserService implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {
    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2User oauth2User = new DefaultOAuth2UserService().loadUser(userRequest);

        // 여기에 유저 정보 DB 저장 or 회원가입 로직
        System.out.println("Google user info: " + oauth2User.getAttributes());

        return oauth2User; // 나중엔 커스텀 UserPrincipal로 바꾸는 게 좋아
    }
}
```

---

### 4. 시큐리티 설정

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/login**", "/css/**", "/js/**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2Login(oauth2 -> oauth2
                .userInfoEndpoint(userInfo -> userInfo
                    .userService(new CustomOAuth2UserService()) // 위에서 만든 서비스
                )
                // 로그인 성공 후 처리
                .successHandler((request, response, authentication) -> {
                    // 여기서 JWT 발급 가능
                    System.out.println("OAuth2 Login Success");
                    response.sendRedirect("/home"); // 또는 토큰 포함해서 프론트에 전달
                })
            );

        return http.build();
    }
}
```

---

### 5. JWT 발급이 필요하다면?

로그인 성공 핸들러 안에서 `authentication.getPrincipal()`로 유저 정보 꺼내서 JWT 발급하면 돼.

```java
OAuth2User user = (OAuth2User) authentication.getPrincipal();
String email = user.getAttribute("email");
String token = jwtProvider.createToken(email);
response.setHeader("Authorization", "Bearer " + token);
```

---

## ✅ 정리하자면

|할 일|방식|
|---|---|
|구글 로그인 화면 띄우기|프론트가 `/oauth2/authorization/google`로 리디렉트|
|인가 코드 처리|Spring이 자동 처리|
|access token → 유저 정보 조회|`CustomOAuth2UserService`에서 처리|
|로그인 성공 후 작업|`.successHandler()`에서 JWT 발급 등 처리|

---

필요하면 이걸 기반으로 DTO 설계나, JWT 발급 유틸, 리프레시 토큰 구현도 같이 가이드해줄게.  
**이 구조로 가고 싶다** 확정이면, 내가 전체 뼈대를 zip 파일처럼 정리해줄 수도 있어. 계속 같이 가보자!