# 2025년 9월 5일 TIL

## Spring DI 복습 및 MyBatis 기초 학습

오늘은 Spring의 DI(의존성 주입) 핵심 개념을 복습하고, 데이터베이스 연동을 위한 MyBatis 프레임워크의 기초를 다졌다. 특히 전통적인 JDBC 방식의 문제점을 명확히 인지하고, MyBatis가 이를 어떻게 해결하는지에 집중했다.

---

### 1. Spring DI (Dependency Injection) 복습

- **IoC 컨테이너**: `BeanFactory`(lazy-loading)와 `ApplicationContext`(eager-loading)의 차이점을 다시 확인했다. `ApplicationContext`가 더 많은 기능을 제공하여 일반적으로 사용된다.
- **DI 구현**: XML 설정 파일을 통해 Setter Injection(`<property>`)과 Constructor Injection(`<constructor-arg>`)을 구현하는 방법을 복습했다.
- **외부 설정 분리**: `<context:property-placeholder>`를 사용하여 DB 접속 정보와 같은 민감하거나 변경이 잦은 설정을 `.properties` 파일로 분리하여 유지보수성을 높이는 방법을 학습했다.

---

### 2. MyBatis의 필요성: JDBC와의 비교

- **전통적 JDBC의 문제점 (Pain Points)**
  1.  **반복적인 상용구 코드**: `try-catch-finally`를 이용한 DB 연결 및 자원 해제 코드가 반복적으로 필요하다.
  2.  **SQL과 코드의 결합**: SQL 쿼리가 자바 코드 내에 문자열 형태로 존재하여, SQL 변경 시 자바 코드 수정 및 재컴파일이 필요하다.
  3.  **수동적인 데이터 매핑**: `ResultSet`의 결과를 `while` 루프를 돌며 직접 VO 객체에 `set` 해주는 과정이 번거롭고 실수할 가능성이 높다.

- **MyBatis를 통한 해결**
  - MyBatis는 위의 문제점들을 해결해주는 **SQL Mapper** 프레임워크다.
  - 번거로운 JDBC 연결/종료 과정을 자동화하고, SQL을 XML 파일로 완벽하게 분리하며, 조회 결과를 VO 객체에 **자동으로 매핑**해준다.

---

### 3. MyBatis 핵심 구성 요소 및 동작 흐름

MyBatis는 설정과 역할을 명확히 분리하여 동작한다.

1.  **`mybatis-config.xml` (설계도)**: DB 접속 정보, 트랜잭션 규칙, `typeAlias`(별칭), Mapper 파일 위치 등 MyBatis 프레임워크의 전체적인 동작을 설정한다.
2.  **`UserMapper.xml` (SQL 창고)**: 실제 실행될 SQL 쿼리를 `namespace`와 `id`로 그룹화하여 관리한다. SQL과 자바 코드의 분리가 이루어지는 곳이다.
3.  **Java Application (실행)**: `SqlSessionFactory`를 통해 `SqlSession`을 얻고, `session.selectOne("namespace.id", parameter)`와 같은 형태로 Mapper에 정의된 SQL을 호출하여 사용한다.

---

### 4. MyBatis `SELECT` 심화 학습

CRUD 중 Read 기능에 집중하여 다양한 조회 방법을 학습했다.

-   **전체 목록 조회**: `selectList("UserMapper01.findUserList")`
-   **단일 항목 조회**: `selectOne("UserMapper01.findUser", "user01")`
-   **조건부 검색**: `LIKE` 연산자를 사용하여 이름에 특정 문자가 포함된 사용자를 검색했다.
-   **동적 SQL**: `<where>`, `<if>` 태그를 사용하여 파라미터 값의 유무에 따라 WHERE 조건절이 동적으로 변경되는 쿼리를 작성했다. 이는 매우 유연하고 강력한 기능이다.

---

### 5. 핵심 개념: `parameterType` vs `resultType`

메서드의 파라미터와 리턴 타입에 비유할 수 있는 핵심 속성이다.

| 구분 | `parameterType` | `resultType` |
| :--- | :--- | :--- |
| **방향** | Java → SQL | SQL → Java |
| **역할 (비유)** | SQL에 전달하는 **입력(파라미터)** 타입 | SQL 실행 결과를 담을 **결과(리턴)** 타입 |
| **주요 사용 타입**| `string`, `int`, `user`(별칭), `map` | `user`(별칭), `hashmap`, `int` |
| **동작 방식**| `#{...}` 부분에 값을 채워 넣음 | 조회 결과를 객체에 자동으로 매핑 |

---

### 오늘의 느낀 점

전통적인 JDBC 방식의 불편함을 코드로 직접 확인하고 나니, MyBatis와 같은 프레임워크의 필요성을 절실히 느낄 수 있었다. 특히 SQL을 XML로 분리하여 코드의 가독성과 유지보수성을 높이고, 번거로운 매핑 작업을 자동화해주는 부분이 인상 깊었다. 프레임워크는 단순히 기능을 제공하는 것을 넘어, '관심사의 분리(SoC)'라는 좋은 설계 원칙을 따르도록 유도한다는 점을 다시 한번 깨달았다.
