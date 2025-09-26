# Today I Learned: AJAX의 발전 과정과 핵심 용어 정리

**Date:** 2025-09-26  
**Tags:** `#AJAX`, `#JavaScript`, `#XMLHttpRequest`, `#jQuery`, `#FetchAPI`, `#Async`

---

## 1. 순수 JavaScript (Vanilla JS)를 이용한 전통적인 AJAX (`XMLHttpRequest`)

라이브러리 없이 AJAX를 구현하는 가장 기본적인 방법입니다. 구형 브라우저 호환성까지 고려해야 해서 코드가 다소 복잡합니다.

### 핵심 4단계

1. **객체 생성**: `new XMLHttpRequest()`로 요청 객체를 만듭니다. (구형 IE는 `new ActiveXObject()` 사용)
2. **요청 초기화**: `request.open("GET", "URL", true)` 메서드로 요청 방식, 주소, 비동기 여부를 설정합니다.
3. **콜백 함수 지정**: `request.onreadystatechange`에 응답 상태가 변할 때마다 실행될 함수를 지정합니다.
4. **요청 전송**: `request.send(null)`으로 서버에 요청을 보냅니다.

### 응답 처리

콜백 함수 내에서 아래 두 가지를 반드시 확인해야 합니다:
- `request.readyState === 4`: 요청이 모두 완료되었는지 확인
- `request.status === 200`: HTTP 상태 코드가 '성공(OK)'인지 확인

```javascript
// 전체 XMLHttpRequest 구현 예시
var request = new XMLHttpRequest();

// 응답 처리 콜백 함수
function updatePage() {
    if (request.readyState == 4) { // 요청이 완료되면
        if (request.status == 200) { // 성공적으로 완료되면
            var serverData = request.responseText;
            // document.getElementById(...) 등을 이용해 DOM 조작
            console.log("받은 데이터:", serverData);
            document.getElementById('sold').textContent = serverData;
        } else {
            alert("에러 발생: " + request.status);
        }
    }
}

// AJAX 요청 실행
request.open("GET", "calcServerAjax.jsp", true);
request.onreadystatechange = updatePage;
request.send(null);
```

---

## 2. jQuery를 이용한 AJAX (`$.ajax`)

`XMLHttpRequest`의 복잡함을 추상화하여 매우 간결하고 직관적인 코드로 AJAX를 구현할 수 있게 해주는 라이브러리입니다.

### 특징

- **간결함**: `$.ajax()` 함수 하나로 대부분의 기능을 처리합니다.
- **직관적인 콜백**: `success`와 `error` 옵션으로 성공/실패 시의 로직을 명확하게 분리할 수 있습니다.
- **크로스 브라우징**: jQuery가 내부적으로 브라우저 호환성 문제를 해결해줍니다.
- **편리한 DOM 조작**: `$('#id').text(data)` 와 같이 DOM을 쉽게 제어할 수 있습니다.

```javascript
// jQuery AJAX 예시
$.ajax({
    url: "calcServerAjax.jsp",
    method: "GET",
    dataType: "text",
    success: function(data) {
        // 성공 시 로직
        console.log("성공! 받은 데이터:", data);
        $('#sold').text(data); // DOM 업데이트
    },
    error: function(xhr, status, error) {
        // 실패 시 로직
        console.error("에러 발생:", status, error);
    }
});
```

---

## 3. 최신 JavaScript의 Fetch API (`async/await`)

`XMLHttpRequest`를 대체하기 위해 나온 현대적인 JavaScript 표준 통신 API입니다. **Promise**를 기반으로 작동하여 비동기 코드를 더 깔끔하게 작성할 수 있습니다.

### 특징

- **Promise 기반**: `fetch()`는 Promise 객체를 반환합니다. `.then()`으로 후속 처리를 연결하거나 `async/await`와 함께 사용할 수 있습니다.
- **`async/await`**: 비동기 코드를 동기 코드처럼 순차적으로 작성하게 만들어 가독성을 극대화합니다.
- **에러 처리**: `try...catch` 구문을 통해 동기 코드처럼 자연스럽게 에러를 처리할 수 있습니다.
- **의존성 없음**: 브라우저에 내장된 기능이라 별도의 라이브러리가 필요 없습니다.

```javascript
// Fetch API와 async/await 예시
async function loadDataModernWay() {
    try {
        const response = await fetch('calcServerAjax.jsp');
        
        if (!response.ok) { // HTTP 상태가 200-299 범위가 아닐 때
            throw new Error(`HTTP 에러: ${response.status}`);
        }
        
        const data = await response.text();
        console.log("성공! 받은 데이터:", data);
        document.getElementById('sold').textContent = data; // DOM 업데이트
        
    } catch (error) {
        console.error("요청 실패:", error);
    }
}

// 함수 호출
loadDataModernWay();
```

---

## 4. 핵심 요약 및 비교

| 구분 | `XMLHttpRequest` (전통 방식) | `jQuery $.ajax` | `Fetch API` (현대 방식) |
|:---|:---|:---|:---|
| **문법** | 다소 복잡하고 장황함 | 간결하고 직관적인 객체 방식 | Promise 기반, `async/await`로 가독성 높음 |
| **비동기 처리** | 이벤트 핸들러(`onreadystatechange`)와 콜백 | 콜백 함수 (`success`, `error`) | Promise, `async/await` |
| **에러 처리** | `if/else`로 `status` 코드를 직접 확인 | `error` 콜백 | `try...catch`와 `response.ok` |
| **의존성** | 없음 (순수 JS) | **jQuery 라이브러리 필요** | 없음 (브라우저 내장) |
| **결론** | AJAX의 기본 원리 이해에 좋음 | 과거에 많이 사용되었으며 여전히 유용함 | **현재 표준**, 특별한 이유가 없다면 사용 권장 |

---

## 💡 알아두면 좋은 주요 용어 정리

### AJAX (Asynchronous JavaScript and XML)
**비동기 방식**으로 서버와 데이터를 교환하여, **웹 페이지 전체를 새로고침하지 않고** 일부만 동적으로 변경하는 기술입니다. 이름에는 XML이 들어가지만, 현재는 주로 **JSON** 형식으로 데이터를 주고받습니다.

### 비동기(Asynchronous) vs 동기(Synchronous) 통신
- **동기**: 요청을 보낸 후, 서버로부터 **응답이 올 때까지 모든 작업이 멈추고 기다리는** 방식입니다. (예: 페이지 전체가 하얗게 변하며 로딩)
- **비동기**: 요청을 보내놓고, 응답을 기다리지 않고 **즉시 다음 작업을 수행**하는 방식입니다. 서버로부터 응답이 오면 그때 약속된 작업(콜백 함수)을 처리합니다. (예: 로딩 중에도 다른 버튼을 누르거나 스크롤 가능)

### DOM (Document Object Model)
웹 페이지(HTML 문서)의 콘텐츠와 구조를 **객체**로 표현한 모델입니다. JavaScript는 이 DOM을 통해 HTML 요소에 접근하고 내용을 변경하거나(`textContent = ...`), 새로운 요소를 추가하는(`appendChild`) 등의 조작을 수행합니다.

### 콜백 함수 (Callback Function)
다른 함수의 **인자(argument)로 전달되어, 특정 시점(예: 서버 응답 완료 시)에 실행되는 함수**를 말합니다. 전통적인 AJAX의 `onreadystatechange`에 할당된 `updatePage` 함수가 대표적인 예입니다.

### `XMLHttpRequest` (XHR)
전통적인 AJAX 방식에서 서버와 통신(데이터 요청 및 수신)을 위해 사용되던 JavaScript 객체입니다. 현재는 `Fetch API`에 자리를 내주고 있습니다.

### `Fetch API`
`XMLHttpRequest`를 대체하는 최신 JavaScript 표준 기술입니다. **Promise** 기반으로 설계되어 `async/await`와 함께 사용하면 비동기 코드를 훨씬 깔끔하고 직관적으로 작성할 수 있습니다.

### Promise
**비동기 작업의 최종 완료 또는 실패를 나타내는 객체**입니다. 작업이 끝나면 성공(resolve) 또는 실패(reject) 상태와 그 결과값을 가집니다. '나중에 채워질 데이터에 대한 약속'이라고 이해할 수 있습니다.

### `async / await`
**Promise를 더 쉽게 다루기 위한 문법**입니다. `async`가 붙은 함수는 항상 Promise를 반환하며, 함수 내부에서 `await` 키워드를 사용해 Promise가 완료될 때까지 코드 실행을 잠시 멈추고 기다릴 수 있습니다. 비동기 코드를 동기 코드처럼 보이게 만들어 가독성을 높여줍니다.

### API (Application Programming Interface)
애플리케이션(프로그램)들이 서로 **정해진 규칙에 따라 상호작용**할 수 있도록 하는 매개체입니다. `Fetch API`처럼 브라우저가 제공하는 기능일 수도 있고, 날씨 정보나 지도 정보를 제공하는 외부 서비스일 수도 있습니다.
