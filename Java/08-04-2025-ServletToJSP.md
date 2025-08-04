# [TIL] JSP 내장 객체, Scope 그리고 MVC 아키텍처

**Date:** 2025-08-04
**Tags:** #Java #Servlet #JSP #MVC #Scope #Include #Architecture

## 오늘 배운 것

오늘은 서블릿-JSP 기반의 MVC 패턴을 학습한 것에 이어, JSP의 핵심 요소인 **내장 객체의 Scope(스코프)**와 **페이지 Include 방식**에 대해 깊이 있게 공부했다. 각 Scope의 생명주기와 데이터 공유 범위를 이해하고, 정적 Include와 동적 Include의 차이점을 명확히 파악하는 시간이었다.

## 1. 모델 1 아키텍처 vs 모델 2 아키텍처 (복습)

-   **모델 1:** JSP가 요청 처리(Model)와 화면 표시(View)를 모두 담당. 구조가 단순하지만 로직이 섞여 유지보수가 어렵다.
-   **모델 2 (MVC):** 요청을 받는 **Controller(Servlet)**, 비즈니스 로직을 처리하는 **Model(Java Class)**, 결과를 보여주는 **View(JSP)**로 역할을 분리한 구조. 재사용성과 유지보수성이 높다.

> **"서블릿은 요청(Request)을, JSP는 응답(Response)을"**

이 말처럼, 모델 2 아키텍처에서 역할 분담은 명확하다. **Controller(Servlet)가 요청을 받아 처리**하고, 그 결과를 **View(JSP)를 통해 응답**하는 흐름이 핵심이다.

## 2. MVC 패턴과 컨트롤러(Controller)의 역할 (복습)

Controller(Servlet)는 MVC 패턴의 중재자로서 다음 네 가지 핵심 역할을 수행한다.

1.  **클라이언트 요청 판정 및 분석**: 사용자의 요청(URI, 파라미터)을 파악.
2.  **비즈니스 로직(B/L) 수행 요청**: 요청에 맞는 Model(Service 객체 등)을 호출.
3.  [cite_start]**모델(Model)과 뷰(View) 연결**: Model의 처리 결과를 `request` 등의 Scope에 저장하여 View에 전달. [cite: 8, 9, 12]
4.  **네비게이션 / 뷰 선택**: 사용자에게 보여줄 View(JSP)로 `forward` 또는 `redirect`.

## 3. JSP 내장 객체와 Scope의 이해 (Today's Focus)

JSP는 데이터를 공유하기 위해 여러 생명주기를 가진 메모리 영역(Scope)을 제공한다. 이 영역들은 `request`, `session`, `application` 같은 내장 객체를 통해 관리된다.

### 객체 스코프(Object Scope)의 종류와 생명주기

| Scope         | 관련 내장 객체         | 생명 주기 및 공유 범위                                                                        | 예시 사용처                                |
| ------------- | ---------------------- | --------------------------------------------------------------------------------------------- | ------------------------------------------ |
| **Request** | `HttpServletRequest`   | [cite_start]HTTP 요청 하나가 시작되고 끝날 때까지 (요청 ~ 응답 전까지). [cite: 10] 페이지 이동 시 소멸.           | Controller에서 View로 1회성 데이터 전달.   |
| **Session** | `HttpSession`          | 브라우저 한 개당 하나씩 생성. 브라우저를 닫거나 세션 만료 전까지 유지. [cite_start]**("나만 접근")** [cite: 3]   | 로그인 정보, 장바구니 등 사용자별 데이터 유지. |
| **Application** | `ServletContext`       | 웹 애플리케이션(WAS) 시작부터 종료까지 유지. 모든 클라이언트가 공유. [cite_start]**("누구나 접근")** [cite: 14] | 전체 방문자 수, 공지사항 등 공용 데이터 관리. |

-   [cite_start]**Request Scope**: `03RequestScopeTest.jsp`에서 확인했듯, 페이지를 새로고침(새로운 요청)할 때마다 카운터가 `1`로 초기화된다. [cite: 10, 11, 12, 13]
-   [cite_start]**Session Scope**: `01SessionScopeTest.jsp`에서는 브라우저를 끄기 전까지 새로고침해도 방문 횟수가 계속 누적되었다. [cite: 3, 4, 5, 6]
-   [cite_start]**Application Scope**: `02ApplicationScopeTest.jsp`는 다른 브라우저로 접속해도 카운트가 공유되어 계속 증가하는 것을 보여준다. [cite: 14, 15, 16, 17]

### 페이지 포함: 정적 vs 동적 Include

JSP 페이지에 다른 페이지의 내용을 포함시키는 방법은 두 가지가 있다.

-   **정적 Include: `<%@ include file="..." %>`**
    -   **동작 방식**: JSP 파일이 서블릿 코드로 변환되는 시점에 `file` 속성에 지정된 파일의 **소스 코드**가 그대로 삽입된다.
    -   **특징**: 하나의 서블릿으로 합쳐지므로, 부모 JSP에 선언된 지역 변수를 포함된 자식 JSP에서 직접 사용할 수 있다.

-   **동적 Include: `<jsp:include page="..." />`**
    -   **동작 방식**: 사용자가 페이지를 요청하는 시점에 `page` 속성에 지정된 JSP의 **실행 결과(HTML)**가 포함된다.
    -   **특징**: 부모와 자식 JSP가 각각 별도의 서블릿으로 처리된다. [cite_start]따라서 부모 JSP의 지역 변수를 공유할 수 없다. [cite: 7]
    -   [cite_start]`04JSPActionInclude.jsp`에서 선언한 지역 변수 `value`를 `05JSPActionInclude.jsp`에서 사용하려고 하면 에러가 발생하는 이유가 바로 이것이다. [cite: 2]
    -   [cite_start]하지만 `request`, `session`, `application` scope에 저장된 데이터는 공유가 가능하다. [cite: 1, 8, 9] `jsp:include`는 요청을 제어하는 흐름 속에서 다른 페이지의 실행 결과를 가져오는 것이므로, 동일한 `request` 객체를 공유하기 때문이다.

## 오늘 작업에 대한 회고

MVC 패턴에서 Controller가 `request.setAttribute()`로 데이터를 넘겨주고 View(JSP)가 이를 받아 화면을 구성하는 원리를 이제 명확히 알겠다. 데이터가 어떤 Scope에 저장되어야 하는지, 그리고 `<jsp:include>`와 같은 액션 태그가 어떤 원리로 동작하는지를 이해하는 것이 모듈화되고 유지보수하기 좋은 웹 페이지를 만드는 핵심이라는 것을 깨달았다.
