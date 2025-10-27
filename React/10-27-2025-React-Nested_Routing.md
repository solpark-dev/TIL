# TIL (Today I Learned)
## 날짜: 2025-10-27

오늘은 React의 `Context API`와 `react-router-dom`을 활용하여 로그인 상태를 전역으로 관리하고, 인증이 필요한 페이지를 보호하는 패턴을 학습했다.

---

### 1. `LoginManager`를 통한 로그인 상태 관리 로직 분리 (관심사의 분리)

`App.js`에 라우팅 로직과 상태 관리 로직이 모두 포함되어 비대해지는 것을 막기 위해, 로그인/인증 처리를 전담하는 `LoginManager` 컴포넌트를 생성했다.

- **`LoginManager.js` (상태 관리자 컴포넌트)**
  - `useState`를 사용해 `login` (로그인 정보), `loginCheckOK` (로그인 확인 완료 여부 플래그) 상태를 관리한다.
  - `useEffect`를 사용해 컴포넌트 마운트 시 `axios`로 `/checkLogin` API를 1회 호출하여 세션 기반의 로그인 상태를 확인한다.
  - `LoginUserContext.Provider`가 되어 `value` prop으로 하위 컴포넌트들에게 `login`, `setChangeLogin`, `loginCheckOK`를 제공한다.
  - `props.children`을 렌더링하여 하위 컴포넌트들을 감싼다.

- **`App.js` (메인 앱)**
  - 로그인 관련 `useState`, `useEffect` 코드가 모두 제거되어 매우 단순해졌다.
  - 전체 컴포넌트를 `<LoginManager>` 태그로 감싸기만 하면, `LoginManager`가 모든 인증 상태를 관리하고 하위 컴포넌트(Header, Main 등)에 공유한다.
- **장점:** 관심사의 분리(SoC)를 통해 `App.js`는 레이아웃과 라우팅에만 집중할 수 있게 되어 코드 가독성과 유지보수성이 향상된다.

---

### 2. 보호된 라우트 (Protected Route) 구현

로그인한 사용자만 접근할 수 있는 페이지(예: 메인 페이지)를 구현하는 방법을 학습했다.

- **`Main.js` (보호된 페이지)**
  - `useContext(LoginUserContext)`를 사용해 `login`, `loginCheckOK` 상태를 가져온다.
  - **핵심 로직 1 (UI 깜빡임 방지):**
    - `if (loginCheckOK === false) return null;`
    - `LoginManager`의 `useEffect` 속 API 호출이 완료되기 전까지(즉, 로그인 상태가 확정되기 전까지) 아무것도 렌더링하지 않는다.
    - 이것이 없으면, 초기 상태(`login.userId`가 `null`) 기준으로 잠시 렌더링되었다가 로그인 페이지로 리디렉션되거나, 그 반대의 깜빡임이 발생한다.
  - **핵심 로직 2 (인증 가드):**
    - `if (!login.userId) { return <Redirect to='/login' />; }`
    - 로그인 확인이 완료되었는데도(`loginCheckOK === true`) `login.userId`가 없으면, 로그인 페이지로 강제 리디렉션시킨다.
  - 위 두 조건을 모두 통과한 경우에만 (로그인 확인이 완료된 && 로그인한 사용자인 경우) 실제 페이지 UI를 렌더링한다.

---

### 3. 중첩 라우팅 (Nested Routing) 및 동적 파라미터 활용

단순 페이지가 아닌, '구매 관리' 페이지처럼 **자체적인 서브 레이아웃과 서브 네비게이션을 갖는** 복잡한 페이지를 구현했다.

- **`App.js` (메인 라우터)**
  - `<Route path='/purchase'><Purchase/></Route>`
  - `/purchase`로 시작하는 모든 하위 경로(예: `/purchase/user03`, `/purchase/purchase02/user01`)의 처리를 `Purchase` 컴포넌트에게 위임한다.

- **`Purchase.js` (서브 라우터 및 레이아웃)**
  - 이 컴포넌트 자체도 `Main.js`와 동일한 **Protected Route** 로직을 가진다. (로그인 안 한 사용자는 `/login`으로 리디렉션)
  - `Purchase01` (왼쪽 메뉴), `Purchase03` (하단) 등 **고정 레이아웃**을 정의한다.
  - 레이아웃의 '컨텐츠 영역'(예: `Col lg={9}`) 내부에 **별도의 `<Switch>`를 사용하여 중첩 라우팅**을 구현한다.
  - 이 중첩 `<Switch>`는 `/purchase` 이후의 URL(예: `/purchase02/:userId`, `/user03`)을 기준으로 오른쪽 컨텐츠 영역만 교체한다.

- **`Purchase01.js` (동적 링크 생성)**
  - `useContext`로 현재 `login.userId`를 가져온다.
  - `<Link to={'/purchase/purchase02/' + login.userId}>`와 같이 **동적인 URL을 가진 링크**를 생성한다.

- **`Purchase02.js` (동적 파라미터 수신)**
  - `react-router-dom`의 `useParams()` 훅을 사용하여 URL의 동적 파라미터 값을 추출한다.
  - `let { userId } = useParams();`
  - 이 `userId` 값을 `axios` API 호출 시 파라미터로 사용하여(예: `.../getUser?name=user01&age=...`), 해당 유저의 특정 데이터를 불러온다.
