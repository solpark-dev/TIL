# 오늘 배운 내용 (TIL) - React & Spring

## 1. React Router: 기본 및 중첩 라우팅

### 1-1. 기본 라우팅 (Main, User)

- **`react-router-dom`** 라이브러리를 사용해 SPA(Single Page Application)의 페이지 전환을 구현했습니다.
- **`<Switch>`**: 여러 `<Route>` 중 URL과 일치하는 **첫 번째 하나만** 렌더링합니다.
- **`<Route path="/경로">`**: 특정 경로에 해당하는 컴포넌트를 연결합니다.
- **`<Link to="/경로">`**: HTML의 `<a>` 태그와 비슷하지만, 페이지 전체를 새로고침하지 않고 화면만 전환시킵니다.

### 1-2. 중첩 라우팅 (Nested Routing) (Product)

- `App.js`의 `<Route>`가 `Product.js`를 렌더링하고, `Product.js` **내부에 또 다른 `<Switch>`와 `<Route>`를 배치**하여 하위 경로(예: `/product/01`, `/product/02`)를 관리하는 방식입니다.
- **`useRouteMatch` Hook**:
  - `let { path, url } = useRouteMatch();`
  - **`path`**: 현재 라우트의 **패턴** (예: `/product`). 하위 `<Route>`의 `path`를 동적으로 생성할 때 유용합니다. (예: `path={path + '/product01'}`)
  - **`url`**: 실제 매칭된 **URL** (예: `/product`). 하위 `<Link>`의 `to` 속성을 동적으로 생성할 때 유용합니다. (예: `to={url + '/product02'}`)
  - 이 훅을 사용하면 `App.js`에서 상위 경로(e.g., `/product` -> `/items`)를 변경해도 하위 컴포넌트의 코드를 수정할 필요가 없어 **유지보수성**이 향상됩니다.

## 2. React Router: 동적 데이터 전달 (Purchase)

URL을 통해 컴포넌트에 동적인 데이터를 전달하는 두 가지 주요 방법을 학습했습니다.

### 2-1. URL 파라미터 (Params) (Purchase01)

- **정의**: URL 경로의 일부를 변수처럼 사용합니다. (예: `/purchase/TV/1000`)
- **라우트 설정**: `<Route path="/purchase/:purchaseName/:price">` (경로에 `:` 사용)
- **값 읽기**: `useParams()` 훅을 사용합니다. (`let { purchaseName, price } = useParams();`)
- **용도**: **필수적인 값** (예: 게시글 ID, 사용자 ID 등)을 전달할 때 주로 사용합니다.

### 2-2. 쿼리 스트링 (Query String) (Purchase03)

- **정의**: URL 경로 뒤에 `?key=value&key=value` 형태로 데이터를 전달합니다. (예: `/purchase?purchaseName=TV&price=1000`)
- **라우트 설정**: `<Route path="/purchase">` (일반 경로 설정)
- **값 읽기**:
  1.  `useLocation()` 훅으로 `search` 속성(`?name=TV...`)을 가져옵니다.
  2.  `new URLSearchParams(search)` 객체를 생성합니다.
  3.  `params.get("purchaseName")` 메서드로 값을 추출합니다.
- **용도**: **선택적인 값** (예: 검색어, 필터 옵션, 정렬 방식 등)을 전달할 때 주로 사용합니다.

## 3. UI 라이브러리 연동 (Etc)

- **`react-bootstrap`** 라이브러리와 `bootstrap.css` 파일을 프로젝트에 도입했습니다.
- **그리드 시스템**:
  - **`<Container>`**: 전체 콘텐츠를 감싸고 정렬하는 최상위 컨테이너입니다.
  - **`<Row>`**: 수평 행(가로줄)을 정의합니다.
  - **`<Col>`**: 수직 열(세로칸)을 정의합니다.
- Bootstrap의 그리드는 한 `<Row>`를 **총 12개의 컬럼**으로 나누는 것을 기본으로 합니다.
  - (예: `<Col lg={3}>`, `<Col lg={9}>` → Large 화면 크기 이상에서 3:9 (즉, 1:3) 비율로 레이아웃을 분할합니다.)

## 4. React + Spring 비동기 API 연동 (User02)

React(프론트엔드)와 Spring(백엔드) 간의 `axios`를 이용한 비동기 통신(AJAX) 흐름을 학습했습니다.

### 4-1. 백엔드 (Spring - UserRestController)

- `@RestController`를 사용해 `/userAPI/getUser` 경로로 `GET` 요청을 받는 API 엔드포인트를 정의했습니다.
- **`Thread.sleep(3000)`**: DB 조회나 복잡한 비즈니스 로직 처리에 시간이 걸리는 상황을 **의도적으로 시뮬레이션**했습니다. 이로 인해 API 응답이 3초간 지연됩니다.

### 4-2. 프론트엔드 (React - User02)

- **`useState(null)`**: 서버에서 받아올 데이터를 저장할 `user` state를 **`null`**로 초기화했습니다. (데이터가 아직 없음을 의미)
- **`useEffect(...)`**: 컴포넌트가 처음 마운트(렌더링)될 때 `axiosGet()` 함수를 호출합니다.
- **`axios.get(...)`**: 백엔드 API에 비동기 GET 요청을 보냅니다.
- **`Thread.sleep(3000)`으로 인한 로딩 처리 (핵심):**

  ```javascript
  // 1. user state는 초기값 'null'
  const [user, setUser] = useState(null);

  // 2. 렌더링 시 'user'가 null이면 (데이터가 없으면)
  if(! user){
      return null; // 3. 아무것도 렌더링하지 않음 (로딩 상태)
  }

  // 4. (3초 후) axios 응답이 오고 setUser(res.data)가 실행되면
  // 5. 'user' state가 변경되어 컴포넌트가 재렌더링됨
  // 6. 재렌더링 시 'user'에 값이 있으므로 if문을 통과하고 데이터를 화면에 그림
  return (
      <div>{user.userName}</div>
  );
  ```
  - **흐름 요약**:
  1.  `User02` 첫 렌더링 → `user`는 `null` → `axios` 요청 시작 → `if(!user)`가 true이므로 `null` 반환 (3초간 화면이 비어있음).
  2.  (3초 후) Spring 응답 도착 → `axios`의 `.then()` 실행 → `setUser(res.data)`로 state 업데이트.
  3.  State 변경으로 `User02` 재렌더링 → `user`에 값이 있음 → `if(!user)` 통과 → 서버 데이터 화면에 출력.

## 5. 총정리 (Wrap-up)

오늘은 React 애플리케이션의 뼈대를 잡는 **라우팅(Routing)**부터 시작했습니다.

1.  **기본 라우팅**으로 페이지를 나누고, **중첩 라우팅**으로 컴포넌트 내부에서 하위 페이지를 관리했습니다.
2.  **URL 파라미터**와 **쿼리 스트링**을 사용해 라우터를 통해 컴포넌트에 동적 데이터를 전달하는 방법을 배웠습니다.
3.  **React-Bootstrap**을 도입하여 UI 라이브러리와 그리드 시스템을 활용한 레이아웃 구성법을 익혔습니다.
4.  마지막으로, **Spring Boot API**를 연동하고 `axios`를 사용해 **비동기 통신**을 구현했습니다.

특히 `Thread.sleep()`을 통해 API 지연 상황을 시뮬레이션하며, `useState(null)`과 조건부 렌더링(`if(!data) return null;`)을 활용한 **"로딩 상태" 처리**는 프론트엔드 개발의 핵심적인 비동기 처리 패턴을 명확하게 보여주었습니다.

결국 오늘은 **React 클라이언트 자체의 구조화**에서 시작하여 **실제 백엔드와의 데이터 연동**까지, 풀 스택(Full-stack) 애플리케이션의 핵심 흐름을 완성도 있게 학습한 과정이었습니다.
