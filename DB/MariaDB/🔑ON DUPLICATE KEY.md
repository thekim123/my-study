### `ON DUPLICATE KEY UPDATE`에서 키가 `email`인지 어떻게 알 수 있을까?**

`ON DUPLICATE KEY UPDATE`는 **"고유(UNIQUE) 키나 기본 키(PRIMARY KEY) 충돌 시 수행할 동작"**을 정의하는 MySQL/MariaDB의 SQL 구문이다.

해당 키가 **충돌 기준**으로 사용되는 이유는 테이블 스키마에 있다.

---

## 📝 **1. 테이블 정의에서 단서 찾기**

아래의 간단한 user 테이블을 참고해보자.
```sql
CREATE TABLE user (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(100) UNIQUE,
    name VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### 📌 **주목할 부분**

1. **PRIMARY KEY (`id`)**
    - 각 행을 고유하게 식별한다.
2. **UNIQUE (`email`)**
    - `email` 필드가 `UNIQUE`로 설정되어 있다.

---

## 🛠️ **2. `ON DUPLICATE KEY UPDATE`의 작동 방식**

- **MariaDB/MySQL의 규칙:**
    
    - `ON DUPLICATE KEY UPDATE`는 **PRIMARY KEY**나 **UNIQUE KEY**가 충돌할 때 실행된다.
    - 이 경우, `id`나 `email`이 기존 데이터와 충돌할 경우, `UPDATE` 절이 실행된다.
- **이 예시에서는 `email`이 UNIQUE로 설정되어 있으므로:**
    
    - `INSERT`를 시도할 때 `email` 값이 이미 존재한다면 `ON DUPLICATE KEY UPDATE` 절이 실행된다.

---

## 📊 **3. 핵심 정리**

1. `email` 필드가 `UNIQUE` 제약 조건을 가지고 있다.
2. `ON DUPLICATE KEY UPDATE`는 `PRIMARY KEY` 또는 `UNIQUE KEY` 중 충돌이 발생한 키를 기준으로 작동한다.
3. 위 테이블에서는 `email`이 `UNIQUE` 제약 조건을 가지고 있기 때문에 `email` 충돌 시 업데이트가 수행된다.

---