> **`.dockerignore` 파일에 `build/` 폴더가 포함되어 있어서**  
> Docker가 `build/libs/look-0.0.1-SNAPSHOT.jar` 파일을 **컨텍스트에 포함하지 않음** → COPY 실패

---

## 🧩 왜 이런 일이 생기냐면

Docker는 `docker build` 실행할 때 **`.dockerignore`에 지정된 항목은 빌드 컨텍스트에 포함하지 않음.**  
그래서 `.dockerignore`에 아래처럼 돼 있으면:

```
build/
```

👉 `build/libs/look-0.0.1-SNAPSHOT.jar`은 Docker 입장에선 "없는 파일"이 되어 COPY 실패!

---

## ✅ 해결 방법

### ✔ 다단계 빌드(Multi-stage)로 해결 (보통은 나중에 적용)

`.dockerignore`에 build가 있어도, Gradle로 jar 빌드도 Docker 안에서 하면 `.dockerignore`에 영향 안 받아.

```Dockerfile
# 1단계: Gradle 빌드
FROM gradle:8.5-jdk21 AS builder
COPY --chown=gradle:gradle . /app
WORKDIR /app
RUN gradle bootJar

# 2단계: 실제 실행 이미지
FROM openjdk:21-jdk-slim
COPY --from=builder /app/build/libs/*.jar app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

→ 이 방법은 `.dockerignore` 안 건드려도 되고, **최적화에도 좋음.**

---

## 🔧 추천

- **지금은 `.dockerignore`에서 `build/` 제거**하고 진행하고,
    
- 나중에 안정화되면 다단계 빌드로 구조 개선해도 좋아!
    

---

수정 후 다시 해보면 바로 Docker build 성공할 거야.  
완료되면 ECR 푸시까지 도와줄게! 😎  
혹시 `.dockerignore` 보여줘도 체크해줄 수 있어.