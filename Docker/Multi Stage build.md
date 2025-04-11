도커에 멀티 스테이지 빌드라는게 있다라고 한다. 원래는 이런게 없었는데 업데이트 되었다고 한다.(물론 옛날에 됫겠지) 이게 뭐냐면 간단하게 말하면 `FROM` 을 몇 번 쓰고 `RUN`을 한 후에 다시 `FROM`이 호출이 되면 그건 다른 스테이지로 구분한다는 것이다. 그래서 앞 스테이지는 '임시 작업 공간'이 된다. 결과물만 복사해 오는 것이다.

```dockerfile
# 1단계: 빌드 용 JDK + Gradle
FROM gradle:8.5-jdk21-alpine as build
...
RUN ./gradlew bootJar

# 2단계: 최종 런타임 (JRE만 포함)
FROM eclipse-temurin:21-jre-alpine
COPY --from=build /app/build/libs/app.jar /app/app.jar
ENTRYPOINT ["java", "-jar", "app/app.jar"]
```
![[Pasted image 20250411171231.png]]

## 🔥 여기서 중요한 건:


> ✅ **최종 이미지에는 “두 번째 FROM” 이후 내용만 들어간다!**  
> → 앞 단계 (`build`)는 **중간 빌드에만 사용되고, 결과물만 복사됨**


---

## ✅ 이름 붙여서 스테이지 재사용할 수도 있음

```dockerfile
FROM gradle:8.5-jdk21-alpine AS builder
# 빌드 작업...  
FROM eclipse-temurin:21-jre-alpine
COPY --from=builder /app/build/libs/*.jar /app/app.jar
```

`COPY --from=builder`와 같이 사용하여 재사용할 수 있다. 이 경우 **이름 붙인 스테이지에서 결과물 가져오는 것**이다.

