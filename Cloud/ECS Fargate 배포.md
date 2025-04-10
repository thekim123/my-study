Spring Boot 프로젝트가 이미 있다고 가정한다. 
콘솔로 먼저 **ECS Fargate 배포 → CI/CD 자동화**까지 해보자.  

---

## 🧭 전체 로드맵

1. **Dockerize Spring Boot 프로젝트**
    
2. **ECR 레포지토리 생성**
    
3. **Docker 이미지 빌드 & 푸시**
    
4. **ECS Fargate 배포 (콘솔)**
    
5. **ALB 연결로 외부에서 접속 가능하게**
    
6. **GitHub Actions로 CI/CD 자동화**
    

---

## 1️⃣ Dockerize Spring Boot 프로젝트

먼저 `Dockerfile`부터 만들자. `jar` 하나로 실행되는 구조라고 가정하고 예시 줄게:

```Dockerfile
# 1. OpenJDK 기반 이미지 사용
FROM openjdk:21-jdk-slim

# 2. 앱 JAR 복사
ARG JAR_FILE=build/libs/your-app-name.jar
COPY ${JAR_FILE} app.jar

# 3. 실행
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

👉 `your-app-name.jar`는 실제 빌드한 jar 이름으로 바꿔줘야 해.  
👉 Gradle이라면 `./gradlew bootJar`로 jar 빌드 가능.

---

## 2️⃣ ECR 리포지토리 생성 (AWS 콘솔)

1. AWS 콘솔 → **ECR** → “리포지토리 생성”
    
2. 퍼블릭/프라이빗 중 **프라이빗** 선택
    
3. 이름은 `springboot-demo` 같은 걸로
    

생성되면 ECR 주소가 나와:  
예) `123456789012.dkr.ecr.ap-northeast-2.amazonaws.com/springboot-demo`

---

## 3️⃣ Docker 빌드 & ECR 푸시

🔐 먼저 AWS CLI 로그인:

```bash
aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.ap-northeast-2.amazonaws.com
```

Docker 이미지 빌드:

```bash
docker build -t springboot-demo .
```

ECR용 태그 추가 & 푸시:

```bash
docker tag springboot-demo:latest 123456789012.dkr.ecr.ap-northeast-2.amazonaws.com/springboot-demo:latest
docker push 123456789012.dkr.ecr.ap-northeast-2.amazonaws.com/springboot-demo:latest
```

---

## 4️⃣ ECS Fargate 배포 (콘솔)

1. AWS 콘솔 → **ECS → 클러스터 생성**
    
    - Fargate / Networking only 선택
        
    - 클러스터 이름: `springboot-cluster`
        
2. **Task Definition 생성**
    
    - 런타입: Fargate
        
    - 컨테이너: ECR 이미지 URL 입력
        
    - 포트: 8080
        
3. **서비스 생성**
    
    - 클러스터: `springboot-cluster`
        
    - 서비스 이름: `springboot-service`
        
    - 원하는 개수: 1
        
4. **ALB 연결**
    
    - 로드 밸런서 생성 (새로)
        
    - 리스너: HTTP 80 → ECS 서비스 포워딩
        
    - 보안 그룹 설정: 80 포트 허용
        
5. 잠시 기다리면 배포 완료. ALB URL로 접속 테스트!
    

---