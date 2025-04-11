집에서 ecs에 `Docker` 이미지를 올리는데 너무 느리다 못해 중간에 꺼져서(우리집 인터넷은 많이 느리다.) 이유를 파악하다가 이 `Docker` 이미지 크기 자체가 1GB에 육박하는 것을 발견했다. 그래서 이 용량을 줄이기 위해 여러가지 시도를 하고 그 기록을 적는다. 

결과 Dockerfile은 **멀티 스테이지 빌드**로 구성되어 있고, **이미지 최적화 + 보안성 + 실전 운영에 적합하게** 설계되어 있다.

---

## 📦 전체 구조 요약

| 단계      | 목적         | 설명                                |
| ------- | ---------- | --------------------------------- |
| **1단계** | 커스텀 JRE 생성 | 필요한 Java 모듈만 포함한 JRE를 `jlink`로 만듦 |
| **2단계** | Gradle 빌드  | `bootJar`로 스프링부트 JAR 생성           |
| **3단계** | 최종 런타임 이미지 | 커스텀 JRE + 빌드된 JAR만 포함된 최종 슬림 이미지  |

---
## 🔧 각 단계별 설명
### 🚀 1단계: 커스텀 JRE 생성

```dockerfile
FROM eclipse-temurin:21-alpine as builder-jre

RUN apk add --no-cache binutils

RUN $JAVA_HOME/bin/jlink \
         --module-path "$JAVA_HOME/jmods" \
         --verbose \
         --add-modules ALL-MODULE-PATH \
         --strip-debug \
         --no-man-pages \
         --no-header-files \
         --compress=2 \
         --output /jre
```

- JDK 21의 Alpine 경량 이미지 사용
- `apk add binutils`: `jlink` 실행에 필요한 툴 설치

- `jlink`: 필요한 모듈만 포함해 커스텀 JRE 생성
    - `--add-modules ALL-MODULE-PATH`: 모든 모듈 포함 (최적화 여지 있음)
    - `--compress=2`: 압축
    - 결과는 `/jre` 디렉토리에 생성됨
        

---

### 🔨 2단계: Gradle 빌드

```dockerfile
FROM gradle:8.5-jdk21-alpine AS build
WORKDIR /app

COPY build.gradle settings.gradle ./
COPY gradle ./gradle
RUN --mount=type=cache,target=/home/gradle/.gradle ./gradlew dependencies || true

COPY . .
RUN --mount=type=cache,target=/home/gradle/.gradle ./gradlew bootJar
```

- Gradle 8.5 + JDK 21 환경에서 빌드
    
- `--mount=type=cache`: 의존성 캐시 재사용
    
- `./gradlew bootJar`: 테스트 생략하고 Spring Boot 실행용 JAR 생성
    

---

### 🧊 3단계: 최종 실행 이미지 구성

```dockerfile
FROM alpine:3.18
ENV JAVA_HOME=/jre
ENV PATH="$JAVA_HOME/bin:$PATH"

ARG APPLICATION_USER=appuser
RUN adduser --no-create-home -u 1000 -D $APPLICATION_USER && \
    mkdir /app && chown -R $APPLICATION_USER /app

USER 1000
WORKDIR /app

COPY --from=builder-jre /jre $JAVA_HOME
COPY --chown=1000:1000 --from=build /app/build/libs/*.jar /app/app.jar

ENTRYPOINT ["java", "-Dspring.profiles.active=prod", "-jar", "app.jar"]
EXPOSE 8080
```

- Alpine Linux 기반 최종 이미지
    
- 커스텀 JRE만 복사해서 경량화
    
- **비루트 사용자(appuser)** 사용으로 보안 강화
    
- `/app` 디렉토리에 JAR 복사
    
- `ENTRYPOINT`에서 `spring.profiles.active=prod` 명시
    
- `EXPOSE 8080`: 컨테이너 외부에서 접근할 포트 설정
    

---

## ✅ 주요 특징 요약

| 항목        | 설명                                     |
| --------- | -------------------------------------- |
| 이미지 경량화   | 커스텀 JRE + Alpine 기반, 보통 300MB 내외       |
| 보안        | 루트 계정 미사용, 사용자 생성 후 실행                 |
| 모듈 최적화 가능 | `--add-modules`에 필요한 모듈만 넣으면 이미지 더 줄어듦 |
| 캐시 최적화    | Gradle 의존성 캐시로 빌드 속도 개선                |
| 운영 환경 최적화 | `prod` 프로파일 지정, 불필요한 dev 의존성 미포함       |

---
## ✅ 최종 코드

```dockerfile
#-------------------------------------------------------------  
# 1단계: 커스텀 JRE 생성 (jlink)  
#-------------------------------------------------------------  
FROM eclipse-temurin:21-alpine as builder-jre  
  
RUN apk add --no-cache binutils  
  
RUN $JAVA_HOME/bin/jlink \  
         --module-path "$JAVA_HOME/jmods" \  
         --verbose \  
         --add-modules ALL-MODULE-PATH \  
         --strip-debug \  
         --no-man-pages \  
         --no-header-files \  
         --compress=2 \  
         --output /jre  
  
#-------------------------------------------------------------  
# 2단계: Gradle 빌드  
#-------------------------------------------------------------  
FROM gradle:8.5-jdk21-alpine AS build  
WORKDIR /app  
  
# 캐시 최적화: 의존성 먼저 다운로드  
COPY build.gradle settings.gradle ./  
COPY gradle ./gradle  
RUN --mount=type=cache,target=/home/gradle/.gradle ./gradlew dependencies || true  
  
# 전체 복사 후 빌드  
COPY . .  
RUN --mount=type=cache,target=/home/gradle/.gradle ./gradlew bootJar  
  
#-------------------------------------------------------------  
# 3단계: 최종 실행 이미지 (Alpine + custom JRE)  
#-------------------------------------------------------------  
FROM alpine:3.18  
ENV JAVA_HOME=/jre  
ENV PATH="$JAVA_HOME/bin:$PATH"  
  
ARG APPLICATION_USER=appuser  
RUN adduser --no-create-home -u 1000 -D $APPLICATION_USER && \  
    mkdir /app && chown -R $APPLICATION_USER /app  
  
USER 1000  
WORKDIR /app  
  
COPY --from=builder-jre /jre $JAVA_HOME  
COPY --chown=1000:1000 --from=build /app/build/libs/*.jar /app/app.jar  
  
ENTRYPOINT ["java", "-Dspring.profiles.active=prod", "-jar", "app.jar"]  
EXPOSE 8080
```