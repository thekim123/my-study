상황은 이렇다.
(게시글ID, 이미지이름)을 PK로 둔 테이블이 있다. 이 테이블에 이 이미지에 대한 검색 순위같은걸 저장하는 것인데, Insert할 때 고유식별자로 하다보니 가끔 이미지이름이 같은 것이 들어가면서 Duplicated Key Exception이 발생하는 이슈이다. 그리고 이로 인해 로그가 폭증해서 문제가 되었다. 

사실 같은 사람이면 이 사람의 가장 높은 순위인 row가 필요한 것이기 때문에 멱등 쿼리를 사용해서 insert했다. 고려했던 다른 하나는 insert할 때 이것이 30만건이다 보니 mybatis foreach문에서는 이것을 한번에 수용하지 못했다. 워낙 구버전이다보니 이것에 대해 다른 조치는 불가능했고, 그래서 5000건씩 나눠서 bulk 인서트를 수행하는 방식으로 변경했다.

## 1. SQL 레벨에서 중복을 무시하기
DB가 MariaDB/MySQL이라면 가장 직접적인 방법이다.
LLM에 처음 물어봤을 때 MariaDB/MySQL을 기준으로 알려주었다.

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

## 2. 애플리케이션 레벨에서 사전 필터링
벌크 인서트 전에 기존 키를 조회해서 중복을 미리 제거하는 방식이다. 
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

처음에는 이 방법을 먼저 떠올렸지만, 조회 비용이 추가되고, 조회~삽입 사이 race condition 가능성도 있어서 완벽한 방어는 아니라고 생각했다.

---
오라클이면 쿼리가 좀 다르다.
## 1. MERGE문 (가장 정석)

중복이면 skip, 없으면 insert — Oracle에서 `INSERT IGNORE` 역할을 하는 패턴이다다.
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
Oracle 고유 기능인데, 에러가 나는 row를 별도 에러 테이블로 빼고 나머지는 정상 처리하는 방식이에요. 수만 건 벌크에서 성능이 제일 좋다고 한다. 나는 이렇게까지는 필요가 없었다. 이 insert 작업이 비즈니스에서 중요한 비중을 차지하고 있지도 않았고 그저 맨 윗순이만 뽑아서 저장하면 그만인 상황이었다.
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

UK 위반 row는 `your_table_err_log`에 조용히 들어가고, 정상 row는 그대로 insert 된다. 애플리케이션 로그에는 아무것도 안 찍힌다.

---
그럼 MERGE와 INSERT + NOT EXISTS 중엔 뭐가 나을까? 

## MERGE가 더 나은 점
**원자성이 보장된다.** MERGE는 USING절의 데이터와 대상 테이블을 매칭하는 게 하나의 연산이라, 동시에 같은 키로 insert가 들어와도 UK 위반이 안 생겨요.

반면 `INSERT + WHERE NOT EXISTS`는 SELECT 시점과 INSERT 시점 사이에 미세한 틈이 있어서, 동시 요청이 들어오면 둘 다 "없네?" 하고 insert 시도 → UK 위반이 발생할 수 있어요.

## INSERT + NOT EXISTS가 괜찮은 경우
벌크 인서트가 단일 스레드로만 돌아가고, 동시에 같은 키가 들어올 일이 없다면 사실 이 문제는 안 생겨요. 그러면 가독성 좋은 쪽을 쓰면 돼요.

## 정리하면

|                | MERGE               | INSERT + NOT EXISTS   |
| -------------- | ------------------- | --------------------- |
| 동시성 안전         | O                   | X (race condition 가능) |
| 나중에 upsert로 확장 | WHEN MATCHED 추가하면 끝 | 구조를 갈아엎어야 함           |
| 가독성            | 좀 verbose함          | 직관적                   |
| 성능             | 거의 동일               | 거의 동일                 |

**동시 요청 가능성이 조금이라도 있으면 MERGE가 안전하다.** 그리고 나중에 "중복이면 update 해줘"라는 요구사항이 추가될 때 MERGE는 `WHEN MATCHED THEN UPDATE` 한 줄만 넣으면 되니까 확장성도 좋다.

---

음 근데 만약에 insert하는 쿼리내에서 중복이 생기면 uk 예외가 터질것이다. 그래서 사용한 것은 **ROW_NUMBER**이다:

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

이렇게 하면 UK 기준으로 중복 중 첫 번째만 남기고, 나머지 컬럼에 집계함수 안 써도 된다..