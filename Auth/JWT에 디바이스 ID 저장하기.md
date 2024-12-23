### ✅ **3️⃣ 디바이스 ID를 서버가 직접 확인**

- 디바이스 바인딩은 **클라이언트가 전달하는 값만을 믿지 않습니다.**
- 서버는 클라이언트가 보낸 `deviceId` 외에도 **HTTP Header, IP 주소, User-Agent** 등을 확인하여 추가 검증을 수행합니다.

**예시 (검증 흐름):**

1. Access Token의 `deviceId` 확인
2. HTTP Header의 `deviceId` 확인
3. User-Agent 비교
4. IP 주소 비교