# 2025-10-22 TIL: React 비동기 통신과 라우팅

오늘(2025-10-22)은 React 애플리케이션의 핵심 기능인 **서버와의 데이터 통신** 방법과 **페이지 이동 (라우팅)** 방법을 집중적으로 학습했다.

---

## 1. React 비동기 데이터 통신 (Asynchronous Data Fetching)

웹 애플리케이션은 서버로부터 데이터를 받아와야 한다. 이 과정은 시간이 걸리므로 "비동기"로 처리해야 앱이 멈추지 않는다.

### 1-1. 비동기 통신의 3가지 상태

비동기 통신을 UI에 반영하려면 항상 3가지 상태를 관리해야 한다.

-   `loading`: 요청을 보내고 응답을 기다리는 중 (예: "로딩 중..." 스피너 표시)
-   `data`: 통신에 성공하여 서버로부터 받은 데이터
-   `error`: 통신에 실패했을 때의 에러 정보

### 1-2. 방법 1: `fetch` + `.then().catch()` (Promise Chaining)

JavaScript의 내장 `fetch` API와 프로미스 체이닝을 사용하는 클래식한 방식.

```javascript
const fetchGetThen = () => {
  setLoading(true);
  fetch(url)
    .then(response => {
      if (!response.ok) { // 404, 500 등 서버 에러 처리
        throw new Error('서버 응답이 올바르지 않습니다.');
      }
      return response.json(); // JSON 파싱
    })
    .then(data => {
      setData(data); // 성공
    })
    .catch(error => {
      setError(error); // 실패
    })
    .finally(() => {
      setLoading(false); // 항상 실행
    });
};
```

-   **단점:** `.then()`이 중첩되면 코드가 깊어지고 가독성이 떨어질 수 있다. (콜백 지옥과 유사)

### 1-3. 방법 2: `fetch` + `async/await` (권장)

`.then()` 방식을 더 읽기 쉽게 만든 최신 문법. 코드가 동기식(위에서 아래로)으로 실행되는 것처럼 보여 가독성이 매우 좋다.

```javascript
const fetchGetAsync = async () => {
  setLoading(true);
  try {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error('서버 응답이 올바르지 않습니다.');
    }
    const data = await response.json();
    setData(data);
  } catch (error) {
    setError(error);
  } finally {
    setLoading(false);
  }
};
```

-   **특징:** `async` 함수 내에서 `await` 키워드를 사용.
-   **에러 처리:** `try...catch` 구문을 사용해 동기 코드처럼 에러를 처리할 수 있다.

### 1-4. 방법 3: `axios` + `async/await` (강력 권장)

`fetch`를 더 편리하게 사용하도록 만든 인기 라이브러리. (설치 필요: `npm install axios`)

```javascript
const axiosPostAsync = async () => {
  setLoading(true);
  try {
    // 1. JSON.stringify() 필요 없음
    // 2. 'Content-Type' 헤더 자동 설정
    const response = await axios.post(url, postData);
    
    // 3. response.json() 필요 없음 (response.data에 데이터가 바로 들어있음)
    setData(response.data);
  } catch (error) {
    // 4. 4xx, 5xx 에러가 발생하면 자동으로 catch 블록으로 이동
    setError(error);
  } finally {
    setLoading(false);
  }
};
```

-   **`axios`의 장점:**
    1.  JSON <-> 객체 자동 변환
    2.  `Content-Type` 등 기본 헤더 자동 설정
    3.  404/500 등 서버 에러를 `catch`에서 자동으로 잡아줌 (네트워크 에러와 동일하게 처리 가능)
    4.  `fetch`보다 코드가 훨씬 간결해짐. **현업에서 매우 선호됨.**

### 1-5. (비교) jQuery `$.ajax`와 상태 관리

React와 jQuery는 상태 관리 방식이 근본적으로 다르다.

-   **React:** `useState`로 상태(`data`)를 관리. `setData(newData)`를 호출하면 **React가 알아서 UI를 다시 그린다. (선언형)**
-   **jQuery:** `let userList = []`와 같은 일반 변수로 상태를 관리. `userList.push(data)`로 데이터를 추가해도 **UI는 자동으로 바뀌지 않는다.**
    -   반드시 `renderUserList()` 같은 함수를 **수동으로 호출**하여 DOM을 직접 조작(`$('#result').html(...)`)해야 한다. **(명령형)**

**결론: React 프로젝트에서는 절대로 jQuery를 섞어 쓰지 않는다.**

---

## 2. React 페이지 라우팅 (`react-router-dom` v5 기준)

React는 본질적으로 하나의 HTML 파일(`index.html`)에서 작동하는 **SPA(Single Page Application)**이다. `react-router-dom` 라이브러리는 SPA에서 여러 페이지를 이동하는 것처럼 "보이게" 만들어준다.

### 2-1. 기본 설정 (`index.js`)

앱 전체를 ``로 감싸서 라우팅 기능을 활성화한다.

```javascript
// index.js
root.render(
  
    
  
);
```

### 2-2. `` 태그 vs `` 컴포넌트 (핵심)

-   **`` 태그:** (예: `...`)
    -   **절대 사용 금지.**
    -   클릭 시 브라우저가 서버에 페이지를 새로 요청한다.
    -   페이지 전체가 **새로고침(Full Page Reload)**되며, React 앱이 처음부터 다시 로드된다. SPA의 장점이 모두 사라진다.

-   **`` 컴포넌트:** (예: `...`)
    -   **반드시 사용.**
    -   페이지를 새로고침하지 않고, 브라우저 주소창의 URL만 바꾼다.
    -   React Router가 URL 변경을 감지하여, 정의된 컴포넌트만 교체한다. (클라이언트 사이드 라우팅)

### 2-3. ``와 `` 컴포넌트

-   **``:** URL 경로와 렌더링할 컴포넌트를 짝지어주는 규칙.
    -   ``
    -   `exact={true}`: URL 경로가 **정확히** `/`일 때만 일치. (이거 없으면 `/component02`도 `/`를 포함하므로 둘 다 렌더링될 수 있음)

-   **``:** (v6에서는 ``로 변경됨)
    -   자식 ``들을 위에서부터 순서대로 검사하여, **일치하는 첫 번째 `` 단 하나만** 렌더링한다.
    -   여러 컴포넌트가 동시에 렌더링되는 것을 방지한다.
    -   프로그래밍의 `switch...case` 문과 동일하다.

### 2-4. 404 Not Found 페이지 처리

`Switch`와 와일드카드(`*`)를 사용해 정의되지 않은 페이지로 접근 시 "404 Not Found" 페이지를 보여줄 수 있다.

```javascript
import NotFound from './commonComponent/NotFound';


    
    
    
    
    {/* 위의 모든 경로와 일치하지 않으면
      맨 마지막에 있는 path="*"가 모든 경로를 잡아낸다.
    */}
    

```

-   **`path="*"`**: "모든 경로"를 의미하며, **반드시 `Switch`의 맨 마지막에 위치해야 한다.**

### 2-5. `useLocation` 훅 (Hook)

`NotFound.js` 컴포넌트 등에서 현재 URL 정보를 가져올 때 사용한다.

```javascript
import { useLocation } from "react-router-dom";

const NotFound = () => {
    const location = useLocation();
    // location.pathname은 현재 잘못된 URL 경로 (예: "/asdf")를 담고 있다.
    return (
        
요청하신 Page는 존재하지 않습니다. {location.pathname}

    );
};
```

---

## 3. 오늘의 핵심 요약

1.  **데이터 통신:** `async/await`는 가독성을 위해 필수이며, `axios`는 `fetch`의 번거로운 작업(JSON 변환, 에러 처리)을 자동화해주는 매우 유용한 라이브러리다.
2.  **상태 관리:** React는 `useState`로 상태를 변경하면 UI가 자동으로 업데이트되지만, jQuery는 변수 변경 후 DOM을 수동으로 조작해야 한다.
3.  **페이지 이동:** React SPA에서는 새로고침을 유발하는 `` 태그 대신, 클라이언트 사이드 라우팅을 위한 `` 컴포넌트를 사용해야 한다.
4.  **라우트 관리:** ``를 사용해 단 하나의 페이지만 렌더링되도록 보장하고, ``를 마지막에 두어 404 페이지를 처리한다.
