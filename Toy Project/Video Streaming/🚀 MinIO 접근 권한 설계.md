
**프론트엔드 → ReadOnly**, **백엔드/AI → ReadWrite**로 역할을 분리하면 **보안**과 **효율성** 모두를 잡을 수 있다. 아래에서 각각의 접근 권한을 정리하고, 고려해야 할 부분을 검토해 보겠다.

---

## 📚 **1️⃣ MinIO 권한 설계**

### 🔑 **1. 프론트엔드 (ReadOnly 권한)**

- **권한:** `readonly`
    
- **설명:**
    
    - 프론트엔드는 MinIO에서 **파일을 읽기만 가능**하도록 권한을 부여한다.
    - 사용자가 직접 파일을 조회하거나 이미지를 로드할 때 유용하다.
    - 파일 업로드나 수정 권한은 절대 주지 않는다.
- **예시 정책 (IAM Policy):**
    
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "s3:GetObject"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::your-bucket-name/*"
    }
  ]
}
```

---

### 🔑 **2. 백엔드/AI 서비스 (ReadWrite 권한)**

- **권한:** `readwrite`
    
- **설명:**
    
    - 백엔드와 AI 시스템은 **파일 읽기, 쓰기, 수정, 삭제**가 모두 가능해야 한다.
    - 파일을 업로드하거나 수정, 삭제 등의 기능을 수행한다.
    - 비즈니스 로직에 따라 접근 권한을 보다 세분화할 수 있다.
- **예시 정책 (IAM Policy):**
    
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::your-bucket-name/*"
    }
  ]
}
```

---

### 🔒 **3. 버킷 수준 접근 정책 (Bucket Policy)**

MinIO에서는 버킷 단위로 접근 권한을 설정할 수 있다.

- **버킷 이름:** `app-bucket`
- **프론트엔드 사용자:** `frontend-user` (ReadOnly)
- **백엔드 사용자:** `backend-user` (ReadWrite)

---

## ⚙️ **2️⃣ MinIO 설정 예시**

### **1. 사용자 생성 및 권한 할당**

#### ✅ **프론트엔드 사용자 (ReadOnly)**

```bash
# 사용자 생성
mc admin user add myminio frontend-user frontend-password

# 정책 할당
mc admin policy set myminio readonly frontend-user
```

#### ✅ **백엔드 사용자 (ReadWrite)**

```bash
# 사용자 생성
mc admin user add myminio backend-user backend-password

# 정책 할당
mc admin policy set myminio readwrite backend-user
```

#### ✅ **버킷 정책 설정 (선택적)**

```bash
mc alias set myminio http://localhost:9000 admin admin123
mc admin policy add myminio frontend-policy frontend-policy.json
mc admin policy add myminio backend-policy backend-policy.json
```

---

### 📂 **2. 프론트엔드와 백엔드 연결**

#### ✅ **프론트엔드 (ReadOnly로 직접 접근)**

- 사용자가 파일을 업로드하는 것이 아니라, **프론트엔드는 MinIO로부터 리소스를 직접 읽기**만 한다.
- 보안: 프론트에서 민감한 데이터에 접근하지 못하도록 필수적으로 권한 제한

```typescript
const url = 'http://minio:9000/app-bucket/image.jpg';
const headers = {
  'Authorization': 'Bearer frontend-password'
};

// 이미지 가져오기
fetch(url, { headers })
  .then(response => response.blob())
  .then(blob => {
    const img = URL.createObjectURL(blob);
    document.getElementById('image').src = img;
  });
```

---

#### ✅ **백엔드 (ReadWrite로 접근)**

- 파일 업로드, 삭제, 수정 등은 **백엔드(Spring Boot)**가 수행한다.
- 백엔드는 API를 통해 MinIO와 통신한다.

**예시 코드 (Spring Boot):**

```java
@Autowired
private MinioClient minioClient;

// 파일 업로드
public void uploadFile(String bucket, String fileName, InputStream inputStream) {
    minioClient.putObject(
        PutObjectArgs.builder()
            .bucket(bucket)
            .object(fileName)
            .stream(inputStream, inputStream.available(), -1)
            .contentType("application/octet-stream")
            .build()
    );
}
```

---

## 🛡️ **3️⃣ 보안 고려사항**

1. **액세스 키 노출 주의**
    - 프론트엔드에서 **프라이빗 액세스 키나 시크릿 키**가 노출되지 않도록 주의해야 한다.
2. **Pre-Signed URL 사용 (선택적)**
    - 파일 업로드나 특정 파일 접근 권한을 잠깐 부여해야 할 경우, **Pre-Signed URL**을 사용한다.

**예시 (Pre-Signed URL):**

```java
public String getPresignedUrl(String bucket, String objectName) {
    return minioClient.getPresignedObjectUrl(
        GetPresignedObjectUrlArgs.builder()
            .method(Method.GET)
            .bucket(bucket)
            .object(objectName)
            .expiry(60 * 60) // 1시간 유효
            .build()
    );
}
```

3. **HTTPS 활성화**
    - MinIO는 `SSL/TLS`를 지원하므로 HTTPS를 활성화하여 데이터를 안전하게 전송한다.

---

## 📊 **4️⃣ 장단점 분석**

| **구분**    | **프론트엔드 직접 접근**               | **백엔드 중계 접근**               |
| --------- | ----------------------------- | --------------------------- |
| **장점**    | 네트워크 부하 감소, 프론트에서 파일 로드 속도 빠름 | 보안 강력, 모든 트래픽이 백엔드 통제 하에 있음 |
| **단점**    | 권한 관리 실수 시 보안 이슈 발생 가능        | 백엔드 부하 발생 가능                |
| **사용 사례** | 파일 읽기만 필요한 경우 (예: 이미지 로딩)     | 민감한 파일 처리, 비즈니스 로직 포함       |

---

## ✅ **5️⃣ 결론 및 추천**

1. **프론트엔드:**
    
    - **ReadOnly 권한 부여**
    - 이미지나 정적 파일을 직접 MinIO에서 로드
2. **백엔드/AI:**
    
    - **ReadWrite 권한 부여**
    - 파일 업로드, 삭제, 수정은 백엔드에서 수행
3. **Pre-Signed URL:**
    
    - 임시로 특정 파일에 접근 권한 부여

---