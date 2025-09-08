# TIL (2025-09-08)

## MyBatis 동적 SQL 심화 및 SQL 모듈화

오늘은 MyBatis의 핵심 기능인 **동적 SQL**을 생성하는 다양한 방법과 **SQL 모듈화**를 통해 코드의 재사용성을 높이는 방법을 학습했다. 이를 통해 단순히 JDBC의 반복 작업을 줄이는 것을 넘어, 비즈니스 로직에 따라 SQL을 유연하게 변경하고 유지보수성을 극대화하는 방법을 이해할 수 있었다.

---

### 1. SQL 모듈화: 코드의 재사용성 극대화 🧩

-   **`<sql>` 태그**: 반복적으로 사용되는 SQL 구문을 '조각(Fragment)'으로 정의한다. 컬럼 목록, 공통 WHERE 조건절 등을 저장하는 변수처럼 사용할 수 있다.
-   **`<include>` 태그**: `<sql>` 태그로 정의한 SQL 조각을 필요한 위치에 삽입한다. `refid` 속성으로 사용할 `<sql>` 조각의 `id`를 지정한다.

-   **장점**: 테이블 변경 등으로 SQL 수정이 필요할 때, `<sql>`로 정의된 **한 곳만 수정**하면 `<include>`를 사용한 모든 쿼리에 반영되므로 유지보수성이 크게 향상된다.

```xml
<sql id="userColumns">
    user_id, user_name, password, age, grade
</sql>

<select id="getUser" ...>
    SELECT <include refid="userColumns"/>
    FROM users
    WHERE user_id = #{value}
</select>
```

---

### 2. 동적 SQL 태그: 상황에 맞는 SQL 생성 🤖

파라미터로 전달된 값의 상태(null 여부, 값의 내용 등)에 따라 실행 시점에 SQL 구문을 동적으로 변경하는 기능이다.

#### `<where>`
-   가장 간단한 동적 `WHERE` 절을 생성한다.
-   내부 `<if>` 조건이 하나라도 만족하면 맨 앞에 `WHERE` 키워드를 붙여준다.
-   조건문 맨 앞에 오는 `AND`나 `OR`를 자동으로 제거해준다.

#### `<set>`
-   `UPDATE` 구문에 최적화된 태그다.
-   맨 앞에 `SET` 키워드를 자동으로 붙여주고, 맨 마지막의 불필요한 콤마(`,`)를 제거해준다.

#### `<trim>`
-   `<where>`, `<set>`의 기능을 모두 포함하는 만능 태그. 접두사/접미사 제어를 통해 어떤 구문이든 동적으로 만들 수 있다.
-   `prefix`: 구문 앞에 붙일 문자열
-   `suffix`: 구문 뒤에 붙일 문자열
-   `prefixOverrides`: 구문 앞의 불필요한 문자열 제거
-   `suffixOverrides`: 구문 뒤의 불필요한 문자열 제거

```xml
<trim prefix="SET" suffixOverrides=",">...</trim>

<trim prefix="(" suffix=")" suffixOverrides=",">...</trim>
```

#### `<choose>`, `<when>`, `<otherwise>`
-   자바의 `switch-case`문처럼 여러 조건 중 **하나만 선택**해서 실행하는 배타적 조건 처리에 사용된다.
-   조건의 우선순위를 정해 검색 로직을 구현할 때 매우 유용하다.

---

### 3. 주요 용어 정리 📚

#### OGNL (Object-Graph Navigation Language)
-   MyBatis의 동적 SQL 태그 내부 `test` 속성에서 사용되는 **표현 언어(Expression Language)**다.
-   `test="userName != null and age > 20"` 와 같이 파라미터로 전달된 자바 객체의 속성(property)에 접근하고, 간단한 로직을 수행할 수 있게 해준다.
-   OGNL 덕분에 XML 내에서 파라미터 객체의 상태를 검사하여 동적으로 SQL을 구성할 수 있다.

#### 정적 SQL (Static SQL)
-   실행 시점에 변하지 않는, **완전히 고정된 형태의 SQL**을 의미한다.
-   애플리케이션 코드에서는 파라미터 값만 바인딩할 뿐, SQL 구문 자체의 구조(컬럼, 테이블, WHERE 조건절 등)는 변경되지 않는다.
-   예: `SELECT * FROM users WHERE user_id = ?`

#### 동적 SQL (Dynamic SQL)
-   파라미터 값에 따라 실행 시점에 **SQL 구문의 구조 자체가 동적으로 변경**되는 SQL을 의미한다.
-   MyBatis의 `<if>`, `<choose>`, `<where>` 등의 태그를 사용해 구현한다.
-   상황에 따라 `WHERE` 조건절이 추가되거나 빠지고, `UPDATE`문의 `SET` 절에 포함될 컬럼이 달라지는 등 유연한 쿼리 생성이 가능하다.

---

### ## 오늘의 느낀점 ✨

MyBatis의 진정한 강력함은 단순히 JDBC의 상용구 코드(boilerplate code)를 제거하는 데 있는 것이 아니라, **동적 SQL** 기능에 있다는 것을 깨달았다. 복잡한 비즈니스 로직에 따른 분기 처리를 자바 코드가 아닌 **XML 매퍼에 위임**함으로써, 코드가 훨씬 깔끔해지고 역할(자바: 비즈니스 로직, XML: 데이터 처리)이 명확하게 분리되었다. 특히 `<sql>`과 `<include>`를 통한 모듈화는 향후 프로젝트의 유지보수성을 고려할 때 반드시 적용해야 할 중요한 패턴임을 알게 되었다.
