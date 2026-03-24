상황은 30만건씩


## 1. SQL 레벨에서 중복을 무시하기

DB가 MariaDB/MySQL이라면 가장 직접적인 방법이에요.

**`INSERT IGNORE`** — UK 위반 시 해당 row를 조용히 skip합니다.

```sql
INSERT IGNORE INTO your_table (idx, front, ...) VALUES (?, ?, ...), (?, ?, ...) ...
```

**`ON DUPLICATE KEY UPDATE`** — 중복 시 upsert(덮어쓰기)가 필요하면 이쪽.

```sql
INSERT INTO your_table (idx, front, ...)
VALUES (?, ?, ...)
ON DUPLICATE KEY UPDATE front = VALUES(front)
```

Spring Boot에서 MyBatis 쓰고 있으면 mapper XML에서 바로 적용 가능하고, JPA라면 `@Query` nativeQuery로 처리하면 돼요.

## 2. 애플리케이션 레벨에서 사전 필터링

벌크 인서트 전에 기존 키를 조회해서 중복을 미리 제거하는 방식이에요.

```java
// 1) 삽입 대상 키 셋 추출
Set<CompositeKey> incoming = batch.stream()
    .map(e -> new CompositeKey(e.getIdx(), e.getFront()))
    .collect(Collectors.toSet());

// 2) DB에 이미 있는 키 조회
Set<CompositeKey> existing = repository.findExistingKeys(incoming);

// 3) 차집합만 인서트
List<Entity> toInsert = batch.stream()
    .filter(e -> !existing.contains(new CompositeKey(e.getIdx(), e.getFront())))
    .toList();
```

다만 이건 조회 비용이 추가되고, 조회~삽입 사이 race condition 가능성도 있어서 완벽한 방어는 아니에요.

## 3. 로그 레벨 튜닝 (증상 완화)

근본 해결은 위 1~2번이지만, 당장 로그 폭탄을 줄이려면:

```yaml
# application.yml
logging:
  level:
    org.hibernate.engine.jdbc.spi.SqlExceptionHelper: ERROR
    # 또는 MyBatis 쓴다면
    org.apache.ibatis: WARN
```

이러면 UK 위반 exception의 SQL 전문이 DEBUG/WARN으로 수만 줄 찍히는 건 막을 수 있어요.

---

**추천 조합**: `INSERT IGNORE`(또는 `ON DUPLICATE KEY UPDATE`)로 DB 레벨에서 근본 차단 + 로그 레벨 조정으로 이중 방어. 사전 필터링은 데이터 건수가 정말 많아서 DB 부하 자체를 줄여야 할 때 추가하면 돼요.

혹시 JPA로 하고 있는지 MyBatis로 하고 있는지에 따라 구현 방식이 좀 달라지는데, 어떤 쪽이에요?


---
---


오라클이면 접근이 좀 달라지죠. Oracle에서 쓸 수 있는 방법 정리해볼게요.

## 1. MERGE문 (가장 정석)

중복이면 skip, 없으면 insert — Oracle에서 `INSERT IGNORE` 역할을 하는 패턴이에요.

```xml
<insert id="bulkUpsert" parameterType="list">
    <foreach collection="list" item="item" separator="">
        MERGE INTO your_table t
        USING (
            SELECT #{item.idx} AS idx, #{item.front} AS front
            FROM DUAL
        ) s
        ON (t.idx = s.idx AND t.front = s.front)
        WHEN NOT MATCHED THEN
            INSERT (idx, front, col1, col2)
            VALUES (s.idx, s.front, #{item.col1}, #{item.col2})
    </foreach>
</insert>
```

`WHEN MATCHED THEN UPDATE` 절을 안 넣으면 중복은 그냥 무시돼요. 에러도 안 나고 로그도 안 남아요.

## 2. INSERT + WHERE NOT EXISTS

MERGE보다 가볍게 가고 싶으면:

```xml
<insert id="bulkInsert" parameterType="list">
    <foreach collection="list" item="item" separator="">
        INSERT INTO your_table (idx, front, col1, col2)
        SELECT #{item.idx}, #{item.front}, #{item.col1}, #{item.col2}
        FROM DUAL
        WHERE NOT EXISTS (
            SELECT 1 FROM your_table
            WHERE idx = #{item.idx} AND front = #{item.front}
        )
    </foreach>
</insert>
```

## 3. LOG ERRORS INTO (대량 벌크 특화)

Oracle 고유 기능인데, 에러가 나는 row를 별도 에러 테이블로 빼고 나머지는 정상 처리하는 방식이에요. 수만 건 벌크에서 성능이 제일 좋아요.

```sql
-- 1) 에러 테이블 먼저 생성 (1회)
BEGIN
    DBMS_ERRLOG.CREATE_ERROR_LOG(
        dml_table_name => 'YOUR_TABLE',
        err_log_table_name => 'YOUR_TABLE_ERR_LOG'
    );
END;
```

```xml
<!-- 2) MyBatis mapper -->
<insert id="bulkInsertWithErrorLog">
    INSERT INTO your_table (idx, front, col1, col2)
    VALUES
    <foreach collection="list" item="item" separator="UNION ALL">
        (SELECT #{item.idx}, #{item.front}, #{item.col1}, #{item.col2} FROM DUAL)
    </foreach>
    LOG ERRORS INTO your_table_err_log ('bulk_batch_001')
    REJECT LIMIT UNLIMITED
</insert>
```

UK 위반 row는 `your_table_err_log`에 조용히 들어가고, 정상 row는 그대로 insert돼요. 애플리케이션 로그에는 아무것도 안 찍혀요.

---

**상황별 추천**:

- **건수 적고 단순하게** → `MERGE` (안전하고 직관적)
- **진짜 대량 벌크 (수만~수십만)** → `LOG ERRORS INTO` (성능 최고, 에러 추적도 가능)
- **중복 row를 나중에 분석해야 하면** → `LOG ERRORS INTO`가 에러 테이블에 남겨주니까 딱 맞고

지금 벌크 인서트 건수가 대략 어느 정도 되는 상황이에요?


---
---
솔직히 지금 상황(중복 skip만 하면 되는 경우)에선 둘 다 잘 동작하는데, 차이가 있긴 해요.

## MERGE가 더 나은 점

**원자성이 보장돼요.** MERGE는 USING절의 데이터와 대상 테이블을 매칭하는 게 하나의 연산이라, 동시에 같은 키로 insert가 들어와도 UK 위반이 안 생겨요.

반면 `INSERT + WHERE NOT EXISTS`는 SELECT 시점과 INSERT 시점 사이에 미세한 틈이 있어서, 동시 요청이 들어오면 둘 다 "없네?" 하고 insert 시도 → UK 위반이 발생할 수 있어요.

## INSERT + NOT EXISTS가 괜찮은 경우

벌크 인서트가 단일 스레드로만 돌아가고, 동시에 같은 키가 들어올 일이 없다면 사실 이 문제는 안 생겨요. 그러면 가독성 좋은 쪽을 쓰면 돼요.

## 정리하면

||MERGE|INSERT + NOT EXISTS|
|---|---|---|
|동시성 안전|O|X (race condition 가능)|
|나중에 upsert로 확장|WHEN MATCHED 추가하면 끝|구조를 갈아엎어야 함|
|가독성|좀 verbose함|직관적|
|성능|거의 동일|거의 동일|

**동시 요청 가능성이 조금이라도 있으면 MERGE가 안전해요.** 그리고 나중에 "중복이면 update 해줘"라는 요구사항이 추가될 때 MERGE는 `WHEN MATCHED THEN UPDATE` 한 줄만 넣으면 되니까 확장성도 좋고요.

`NOT EXISTS`가 읽기 편해서 끌리는 거 이해하는데, 운영 환경이면 MERGE 쪽이 더 방어적인 선택이에요.

---
---

맞아요. 500개씩 배치로 넣고 있으면 그 안에 중복이 섞여 있을 때마다 로그 폭탄 맞고 있었던 거니까요.

한 가지 포인트가 있는데, 아까 보여준 MERGE를 foreach로 500번 돌리면 **MERGE문이 500개 나가는 거**라 비효율적이에요. 하나의 MERGE문으로 500건을 한번에 처리하는 게 좋아요.

```xml
<insert id="bulkMerge" parameterType="list">
    MERGE INTO your_table t
    USING (
        <foreach collection="list" item="item" separator="UNION ALL">
            SELECT
                #{item.idx} AS idx,
                #{item.front} AS front,
                #{item.col1} AS col1,
                #{item.col2} AS col2
            FROM DUAL
        </foreach>
    ) s
    ON (t.idx = s.idx AND t.front = s.front)
    WHEN NOT MATCHED THEN
        INSERT (idx, front, col1, col2)
        VALUES (s.idx, s.front, s.col1, s.col2)
</insert>
```

이렇게 하면 USING절에 500개 row가 UNION ALL로 들어가고, MERGE는 **한 번만** 실행돼요. DB 라운드트립도 1번이고, UK 위반 로그도 안 남고요.

다만 Oracle에서 바인드 파라미터 개수 제한이 있을 수 있어서(컬럼 4개 × 500건 = 2000개), 혹시 컬럼이 많으면 배치 사이즈를 좀 줄여야 할 수도 있어요. 500개에 컬럼 수가 적당하면 보통 문제없어요.

---
---
아, 그건 구조적인 차이 때문이에요.

기존 INSERT에서는 아마 `INSERT ALL` 패턴 쓰고 있었을 거예요:

```xml
INSERT ALL
<foreach collection="list" item="item">
    INTO your_table (idx, front, col1)
    VALUES (#{item.idx}, #{item.front}, #{item.col1})
</foreach>
SELECT 1 FROM DUAL
```

이건 `INTO ... VALUES` 절이 그냥 나열되는 구조라 separator가 필요 없는 거고요.

MERGE는 USING절에 **하나의 가상 테이블**을 만들어야 해서, 여러 row를 합치려면 `UNION ALL`이 SQL 문법상 필수예요. MyBatis가 느려지는 게 아니라 Oracle SQL 자체가 그렇게 생긴 거라 성능 걱정은 안 해도 돼요.

`UNION ALL`은 실제로 조인이나 정렬을 하는 게 아니고 그냥 row를 이어붙이는 거라 비용이 거의 없어요. 500건 정도면 체감 차이 없습니다.

---
---
좋은 질문이에요. 여기서 **두 가지 경우**를 구분해야 해요.

**DB에 이미 있는 키와 중복** → MERGE가 잘 처리해요. ON 조건에 매칭되면 그냥 skip하니까 문제 없어요.

**500건 내부에서 서로 중복** → 이건 문제가 돼요. 예를 들어 배치 안에 같은 (idx, front) 쌍이 2개 있으면 MERGE가 둘 다 "없네?" 하고 insert 시도해서 **ORA-30926** 에러가 터질 수 있어요.

그래서 USING절에서 중복을 제거해줘야 해요:

```xml
MERGE INTO your_table t
USING (
    SELECT idx, front, col1, col2 FROM (
        <foreach collection="list" item="item" separator="UNION ALL">
            SELECT
                #{item.idx} AS idx,
                #{item.front} AS front,
                #{item.col1} AS col1,
                #{item.col2} AS col2
            FROM DUAL
        </foreach>
    ) GROUP BY idx, front, col1, col2
) s
ON (t.idx = s.idx AND t.front = s.front)
WHEN NOT MATCHED THEN
    INSERT (idx, front, col1, col2)
    VALUES (s.idx, s.front, s.col1, s.col2)
```

아니면 자바 쪽에서 배치 넣기 전에 미리 걸러도 되고요:

```java
List<Entity> deduplicated = batch.stream()
    .collect(Collectors.toMap(
        e -> e.getIdx() + ":" + e.getFront(),
        e -> e,
        (a, b) -> a  // 중복이면 첫 번째 것만
    ))
    .values()
    .stream()
    .toList();
```

**둘 다 해두는 게 가장 안전해요** — 자바에서 1차 필터링 + SQL에서 GROUP BY로 2차 방어.

---
---
맞아요, 핵심은 UK 컬럼인 `idx, front`만 기준으로 중복을 제거하는 거예요.

근데 `GROUP BY`를 UK만으로 하면 나머지 컬럼(col1, col2)에 집계함수를 써야 해서 좀 귀찮아져요:

```sql
SELECT idx, front, MIN(col1) AS col1, MIN(col2) AS col2
FROM ( ... )
GROUP BY idx, front
```

`MIN`이든 `MAX`든 어차피 같은 UK면 아무거나 하나 pick하는 거라 상관없는데, 컬럼 많아지면 하나하나 다 감싸야 해서 번거롭죠.

그래서 **ROW_NUMBER가 더 깔끔해요**:

```xml
MERGE INTO your_table t
USING (
    SELECT idx, front, col1, col2 FROM (
        SELECT
            idx, front, col1, col2,
            ROW_NUMBER() OVER (PARTITION BY idx, front ORDER BY idx) AS rn
        FROM (
            <foreach collection="list" item="item" separator="UNION ALL">
                SELECT
                    #{item.idx} AS idx,
                    #{item.front} AS front,
                    #{item.col1} AS col1,
                    #{item.col2} AS col2
                FROM DUAL
            </foreach>
        )
    ) WHERE rn = 1
) s
ON (t.idx = s.idx AND t.front = s.front)
WHEN NOT MATCHED THEN
    INSERT (idx, front, col1, col2)
    VALUES (s.idx, s.front, s.col1, s.col2)
```

이렇게 하면 UK 기준으로 중복 중 첫 번째만 남기고, 나머지 컬럼에 집계함수 안 써도 돼요. 컬럼이 많을수록 이쪽이 편합니다.

---
---
네, 이 쿼리 하나로 두 가지 다 막아져요.

- **배치 500건 내부 중복** → ROW_NUMBER로 1건만 남김
- **DB에 이미 있는 키와 중복** → ON 조건에 걸려서 skip

UK 위반 에러 안 터지고, 로그 폭탄도 사라질 거예요. 자바 쪽 사전 필터링은 굳이 안 해도 되는데, 해두면 DB로 보내는 데이터 자체가 줄어드니까 성능상 이득은 있어요. 선택사항이에요.