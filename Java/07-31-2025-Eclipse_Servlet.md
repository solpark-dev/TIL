
# [Java] Stream API 기본 사용법 (2025-07-31)

## 🧐 오늘 무엇을 배웠나요?

- Eclipse에 Tomcat 9.0을 설치
- 
- 기존의 `for`문이나 `Iterator`를 사용하는 방식과 비교했을 때 코드가 얼마나 간결해지는지 직접 확인했다.

## 📚 핵심 내용 정리

### 1. 스트림(Stream)이란?
- 데이터의 흐름을 의미하며, 배열이나 컬렉션의 요소들을 하나씩 참조하여 처리할 수 있는 기능이다.
- 원본 데이터를 변경하지 않고, 별도의 스트림을 생성하여 작업을 수행한다.

### 2. 주요 연산
- **중간 연산 (Intermediate Operation):** 여러 번 연결할 수 있으며, 스트림을 반환한다.
  - `filter(조건)`: 조건에 맞는 요소만 필터링
  - `map(함수)`: 각 요소를 특정 형태로 변환
  - `sorted()`: 정렬
- **최종 연산 (Terminal Operation):** 마지막에 한 번만 사용할 수 있으며, 스트림이 아닌 다른 결과를 반환한다.
  - `collect()`: 스트림의 요소를 수집하여 컬렉션으로 반환
  - `forEach()`: 각 요소를 순회하며 작업 수행
  - `count()`: 요소의 개수 반환


HelloServlet
ojdbc14.jar lib에 등록
C:\work\class730\Day27\jw04\part01\login(SQL).txt db 생성

### 3. 예제 코드
```java
List<String> names = Arrays.asList("Kim", "Park", "Lee", "Choi");
List<String> result = names.stream()
    .filter(name -> name.startsWith("K")) // "K"로 시작하는 이름만 필터링
    .map(String::toUpperCase) // 대문자로 변환
    .collect(Collectors.toList()); // 리스트로 수집

System.out.println(result); // [KIM]


