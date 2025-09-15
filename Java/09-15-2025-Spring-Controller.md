# 2025-09-15 Today I Learned

## 1. Spring MVC 컨트롤러의 데이터 처리 방법 심화

오늘은 Spring MVC 컨트롤러에서 HTTP 요청 파라미터를 처리하고, 그 결과를 View로 전달하는 다양한 방법을 학습했다. 각 방식은 점진적으로 코드를 더 간결하고 직관적으로 만들어준다.

### ### 요청 파라미터(Parameter)를 받는 방법

1.  **`HttpServletRequest` 객체 직접 사용**
    -   가장 전통적인 방식으로, 서블릿 API를 그대로 이용한다.
    -   `request.getParameter("파라미터명")` 형태로 값을 직접 추출해야 한다.
    -   **예시**:
        ```java
        @RequestMapping("/logon01.do")
        public ModelAndView logon01(HttpServletRequest request) {
            String userId = request.getParameter("userId");
            // ...
        }
        ```

2.  **`@RequestParam` 어노테이션 사용**
    -   개별 파라미터를 메서드의 인자(Argument)로 직접 바인딩해준다.
    -   `required`, `defaultValue` 같은 속성을 통해 유연한 처리가 가능하다.
    -   **예시**:
        ```java
        @RequestMapping("/logon03.do")
        public ModelAndView logon03(@RequestParam("userId") String userId,
                                    @RequestParam("password") String password) {
            // ...
        }
        ```

3.  **`@ModelAttribute` 어노테이션 사용 (커맨드 객체)**
    -   가장 세련되고 강력한 방법.
    -   요청 파라미터들을 지정된 객체(`User`)의 필드에 자동으로 바인딩(Data Binding)해준다.
    -   객체는 자동으로 Model에 추가되어 View로 전달된다.
    -   **예시**:
        ```java
        @RequestMapping("/logon05.do")
        public ModelAndView logon05(@ModelAttribute("user") User user) {
            // user 객체에 userId, password 등이 자동으로 채워짐
            // "user"라는 이름으로 Model에 자동 추가됨
            return new ModelAndView("/user/logonResult.jsp");
        }
        ```

---

## 2. `ModelAndView` vs `String` 리턴 타입 비교

컨트롤러 메서드가 View 정보와 Model 데이터를 반환하는 두 가지 대표적인 방법의 차이점을 명확히 이해했다.

| 구분     | `ModelAndView` 리턴                                           | `String` 리턴 (+ `Model` 파라미터)                               |
| :------- | :------------------------------------------------------------ | :---------------------------------------------------------------- |
| **개념** | Model과 View 정보를 **하나의 객체**에 담아 반환.                  | **View 이름만 `String`으로 반환**하고, Model 데이터는 파라미터로 받은 `Model` 객체에 담는다. |
| **코드** | `ModelAndView mav = new ModelAndView();`<br/>`mav.setViewName("...");`<br/>`mav.addObject("key", value);`<br/>`return mav;` | `model.addAttribute("key", value);`<br/>`return "viewName";`      |
| **결론** | 전통적인 방식.                                                | **현대적인 방식으로, 더 간결하고 유연하며 테스트에 용이해 널리 사용됨.** |

**결론**: `String` 리턴 방식은 **관심사의 분리(SoC)** 원칙에 더 잘 부합하며, 코드가 직관적이어서 현대 스프링 개발의 표준으로 자리 잡았다.

---

## 3. 웹 MVC 아키텍처 발전 과정 요약

오늘까지의 학습을 통해 웹 애플리케이션의 MVC 아키텍처가 어떻게 발전해왔는지 전체적인 흐름을 파악할 수 있었다.

1.  **1단계: 수작업 MVC (Model 2)**
    -   모든 컴포넌트(`DispatcherServlet`, `Controller`, `ViewResolver` 등)를 순수 서블릿/JSP 코드로 직접 구현.
    -   MVC의 기본 동작 원리 이해에 중점.

2.  **2단계: XML 기반 초기 Spring MVC**
    -   스프링 프레임워크를 도입하여 직접 만들던 컴포넌트들을 대체.
    -   `web.xml`과 `servlet-context.xml` 같은 XML 파일에 컨트롤러를 `<bean>`으로 하나씩 명시적으로 등록.

3.  **3단계: 어노테이션 기반 현대 Spring MVC**
    -   XML 설정을 최소화하고, 어노테이션으로 대부분의 설정을 대체.
    -   `@Controller`, `@RequestMapping`, `@RequestParam` 등을 사용하여 자바 코드에서 URL 매핑과 데이터 처리를 직접 수행.
    -   코드가 간결해지고 생산성이 크게 향상됨. **현재 학습을 마친 단계.**
