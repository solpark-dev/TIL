# TIL 2025-10-30: React 컴포넌트 점진적 확장 및 Tailwind CSS 도입

오늘은 React에서 **컨테이너/프리젠테이셔널 패턴**을 사용해 페이지를 점진적으로 확장하는 과정을 학습했다. `04UserPage` (회원가입)에서 시작해 `05UserPage` (정보 수정 추가), `06UserPage` (회원 목록 추가)로 기능을 완성했다.

이후, 기존 Bootstrap 스타일링을 제거하고 `061` 버전의 사본 파일을 만들어 **Tailwind CSS**를 도입하는 리팩토링을 진행했다. 이 과정에서 `npm` 버전 호환성 문제, React 훅(Hook)의 생명주기, 비동기 데이터 처리 등 핵심적인 버그들을 디버깅했다.

---

## 1. 주요 학습 내용

### ### 1. 점진적인 컴포넌트 확장 (04 → 05 → 06)

학습 예제의 `if (pathname === ...)` 조건부 렌더링 방식을 기반으로 기능을 확장했다.

* **`04UserPage` (기반 구축):**
    * **컨테이너/프리젠테이셔널 패턴 도입:** `04UserPage.js` (컨테이너)가 `user` 상태와 `handleChange`, `handleSubmit` 등 모든 로직을 관리했다. `04AddUser.js` (프리젠테이셔널)는 `props`를 받아 폼 UI만 그리는 "껍데기" 역할을 했다.
    * **로그인 제어:** `LoginUserContext`를 사용해 `loginCheckOK`가 `false`면 `null`을, `!login.userId`면 `<Redirect to="/login" />`을 반환하는 방어적 코드를 구현했다.
    * **라우팅:** `if (pathname === '/user')`일 때 `AddUser`를, 그 외의 경우 `return`문에서 `<Route path='/user' exact>`로 `UserInfo`를 렌더링하는 (나중에 수정이 필요한) 복합 라우팅 구조를 가졌다.

* **`05UserPage` (기능 추가: 정보 수정):**
    * **컴포넌트 추가:** "껍데기" `05UpdateUser.js`를 생성했다.
    * **로직 추가:** `05UserPage.js`에 `handleSubmitUpdate`라는 새 이벤트 핸들러를 추가했다.
    * **상태 재사용:** `useEffect`에서 `UserService.getUser`를 호출해 가져온 `user` 상태를 "정보 수정" 폼의 초기값으로 재사용했다. `handleChange` 핸들러도 공용으로 사용했다.
    * **라우팅 추가:** `if (pathname === '/user/updateUser')` 블록을 추가하여 `UpdateUser` 컴포넌트를 렌더링했다.

* **`06UserPage` (기능 추가: 회원 목록):**
    * **컴포넌트 추가:** `06UserList.js`와 `06UserListItem.js`라는 껍데기 컴포넌트 2개를 생성했다.
    * **상태 추가:** `const [users, setUsers] = useState([])` (회원 목록) 상태를 추가했다.
    * **로직 추가:** `useEffect` 내에 `UserService.getUserList()`를 호출하는 `fetchUserList` 함수를 추가하고, `users` 상태를 업데이트했다.
    * **라우팅 추가:** `if (pathname === '/user/userList')` 블록을 추가하여 `UserList` 컴포넌트를 렌더링하고 `users={users}` prop을 전달했다.

### ### 2. 스타일링: Bootstrap to Tailwind CSS

기존 `bootstrap.min.css` 의존성을 제거하고 Tailwind CSS로 전환했다.

* **설치:** `react-scripts 5.0.1` 버전은 **Tailwind v3**와 호환되므로, `devDependencies`에 `tailwindcss: ^3.4.18` 등 호환되는 버전을 명시했다.
* **설정:** `npx tailwindcss init -p`로 `tailwind.config.js` 생성 후, `content` 배열에 `./src/**/*.{js,jsx}` 경로를 스캔하도록 설정했다.
* **적용:** `App.js`에서 Bootstrap 임포트를 제거하고 `tailwind.css` (사본)를 임포트했다. `tailwind.css`에는 `@tailwind base`, `@tailwind components`, `@tailwind utilities` 3줄만 남겼다.
* **리팩토링:** `061` 버전 사본 파일들을 생성하고, `ViewGood` 클래스와 인라인 스타일을 모두 제거한 뒤 `max-w-2xl`, `mx-auto`, `bg-point` 등 유틸리티 클래스로 UI를 재구성했다.

---

## 2. 겪었던 문제 및 해결 과정

### ### 1. `npx tailwindcss init -p` 명령어 실패

* **문제:** `npm install`은 성공했으나, `'tailwind'은(는) 내부... 명령이 아닙니다.` 오류가 계속 발생.
* **원인:** **버전 호환성 문제.** `npm install`이 기본으로 최신 **Tailwind v4**를 설치했으나, `react-scripts 5.0.1`은 **PostCSS v8** 기반의 **Tailwind v3**가 필요했다.
* **해결:** `package.json`의 `devDependencies`에 `tailwindcss: ^3.4.18`로 버전을 명시하여 v3를 강제 설치했다.

### ### 2. 회원 목록(`getUserList`) 조회 실패

* **문제:** API를 호출해도 `users.length`가 항상 0으로 나옴.
* **원인:** 백엔드 로그 확인 결과, `Search [pageSize=0]`으로 조회되고 있었다. `UserService.getUserList({ currentPage: 1 })`만 호출하여 `pageSize`의 기본값이 0이 되었다.
* **해결:** `06UserPage.js`의 `fetchUserList` 함수에서 `pageSize: 10`을 명시적으로 추가했다.

### ### 3. 라우팅 로직 충돌 (`/user` 경로)

* **문제:** `04UserPage`에서 "마이페이지"(`/user`)로 이동하면 `UserInfo`가 아닌 `AddUser` (회원가입 폼)가 떴다.
* **원인:** `if (pathname === '/user')` 조건이 `return`문의 `<Route path='/user' exact>`보다 먼저 실행되어 `AddUser`를 렌더링했다.
* **해결:** 회원가입 경로를 `/user/addUser`로 변경하고, `if (pathname === '/user/addUser')`로 조건을 수정하여 "내 정보"(`/user`) 경로와의 충돌을 해결했다.

### ### 4. Presentational 컴포넌트 렌더링 에러

* **문제:** `Cannot read properties of undefined (reading 'userId')` 에러 발생.
* **원인:** `04UserPage` 컨테이너가 `04AddUser`를 렌더링할 때 `formData` prop을 전달하지 않았다. `04AddUser`는 `props.formData.userId`를 읽으려다 `props.formData`가 `undefined`라서 에러가 났다.
* **해결:** `<AddUser formData={user} ... />`와 같이 상태(`user`)를 `formData` prop으로 정확히 전달했다.

### ### 5. `useEffect` 의존성 배열 문제

* **문제:** ESLint 경고 (`Missing dependency: 'login.userId'`) 발생.
* **원인:** `useEffect` 내부에서 `login.userId`를 사용함에도, 의존성 배열이 `[]`로 비어있었다. `LoginManager`가 로그인 정보를 비동기로 가져온 후 `login.userId`가 변경되어도 `useEffect`가 다시 실행되지 않아 `fetchUser`가 호출되지 않았다.
* **해결:** 의존성 배열에 `[login.userId, pathname]`을 추가하여 `login.userId`나 `pathname`이 변경될 때마다 `useEffect`가 다시 실행되도록 수정했다.

---

## 3. 회고

-   학습 예제의 `if (pathname === ...)` 라우팅 방식은 기능이 2~3개(`05`, `06`번) 추가되자마자 관리가 매우 복잡해졌다. React Router의 `<Switch>`를 중첩해서 사용하는 것이 더 확장성 있는 설계임을 깨달았다.
-   `04UserPage`는 `user`라는 단일 상태를 "내 정보 조회", "정보 수정 폼", "회원가입 폼"이 **공유**했다. 이 때문에 `useEffect`에서 `pathname`을 체크하며 "지금 폼을 채울지, 비울지"를 결정하는 로직이 추가되어야 했다. 상태를 명확히 분리(e.g., `userInfo`, `addUserForm`)했다면 더 좋았을 것이다.
-   `npm` 패키지 설치 문제는 `node_modules`나 `npm cache`를 지우기 전에, **버전 호환성**(`react-scripts`와 `tailwindcss`)을 먼저 확인하는 것이 훨씬 빠른 해결책이었다.
-   `input`의 `value` prop에 `null`이 전달되어 경고가 발생했다. `value={user.phone || ''}`처럼 항상 `string` 타입을 보장해주는 것이 안전하다.
