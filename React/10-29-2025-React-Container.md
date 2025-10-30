# TIL 2025-10-30: React 아키텍처 - 컨테이너/프리젠터 패턴

오늘은 React의 "컨테이너/프리젠터 (Container/Presenter)" 패턴과 전역 상태 관리를 중심으로 학습했다. `LoginManager` (Context API)를 통해 로그인 상태를 확인하고, `03UserPage`가 `UserService`를 통해 데이터를 가져와(`Container`) `03UserInfo`에게 `props`로 전달(`Presenter`)하는 구조를 구현했다.

---

## 🚀 오늘 구현한 아키텍처 (컨테이너/프리젠터)

오늘의 목표는 학원 예제와 동일하게, 부모 컴포넌트가 데이터 로직을 전담하는 패턴을 구현하는 것이었다.

### 1. `03UserPage.js` (컨테이너)

* **역할:** 데이터 로직 및 상태 관리.
* `useContext(LoginUserContext)`로 `login.userId`와 `loginCheckOK`를 가져와 로그인 상태를 확인하고 리다이렉트 처리.
* `useState`로 `user` 객체 상태를 관리.
* `useEffect` 내에서 `login.userId`를 기반으로 `UserService.getUser()`를 호출하여 `user` 상태를 업데이트.
* `return` 문에서 `03UserInfo` 컴포넌트를 렌더링하며, 조회한 `user` 상태를 `props`로 전달 (`<UserInfo user={user} />`).

```javascript
// 03UserPage.js (Container)

const UserPage = () => {
  // 1. 전역 로그인 상태 (로그인 체크용)
  const { login, loginCheckOK } = useContext(LoginUserContext);

  // 2. 로컬 상태 (API로 가져온 유저 정보 저장용)
  const [user, setUser] = useState({ userId: "", userName: "", ... });

  // 3. 데이터 조회(fetch) 함수 정의
  const fetchUser = async () => {
    try {
      const userData = await UserService.getUser(login.userId);
      setUser(userData); // 4. 상태 업데이트
    } catch (error) {
      console.error(error);
    }
  };

  // 5. 컴포넌트 마운트 시 데이터 조회
  useEffect(() => {
    if (login.userId) {
      fetchUser();
    }
  }, [login.userId]);

  // 6. 로그인 체크
  if (!loginCheckOK) return null;
  if (!login.userId) return <Redirect to="/login" />;

  // 7. 자식 컴포넌트(프리젠터)에 props로 데이터 전달
  return (
    <div className="ViewGood">
      {/* ... (링크들) ... */}
      <Switch>
        <Route path="/user" exact>
          <UserInfo user={user} />
        </Route>
      </Switch>
    </div>
  );
};
```
### 2. `03UserInfo.js` (프리젠터)

* **역할:** UI 렌더링 전담 (Dumb Component).
* 자체적으로 `Context`나 `UserService`를 사용하지 않는다.
* 부모(`03UserPage`)로부터 `props` (예: `{ user }`)를 받아 화면에 그리기만 한다.

// 03UserInfo.js (Presenter)

// props로 { user } 객체를 구조분해 할당으로 받음
const UserInfo = ({ user }) => {
  console.log(user); // 부모로부터 받은 props

  return (
    <div className="ViewGood">
      <center>
        <h2>[ 내 정보 ]</h2>
        {/* props로 받은 user 객체의 속성을 표시 */}
        <p><strong>아이디:</strong> {user.userId}</p>
        <p><strong>이름:</strong> {user.userName}</p>
        {/* ... (기타 정보) ... */}
      </center>
    </div>
  );
};

## 🤔 아키텍처 비교 (논의)

현재의 "컨테이너/프리젠터" 패턴은 데이터 흐름이 부모 -> 자식으로 명확하다.

하지만 "내 정보 수정", "회원정보 목록보기" 등 하위 기능을 추가할 때 한계가 있다. 학원 예제처럼 `/user` `exact` 라우트만 있으면 `/user/updateUser` 같은 하위 경로에서 `UserInfo` 컴포넌트가 사라지게 된다.

이 문제를 해결하기 위해, 다음 리팩토링 단계에서는 `03UserPage`가 **"중첩 라우팅(Nested Routing)"**을 처리하도록 변경할 예정이다. (즉, `03UserPage`가 레이아웃과 하위 `<Switch>`를 가지고, `03UserInfo`가 스스로 데이터를 가져오는 스마트 컴포넌트가 되는 방식)

---

### ESLint 문제 해결

```markdown
## 🔧 문제 해결: ESLint 경고 (02Logout.js)

`02Logout.js`에서 `react-hooks/exhaustive-deps` 경고가 발생했다.

* **문제:** `useEffect`가 외부 함수 `handleLogout`을 의존성 배열(`[]`) 없이 호출했다.
* **해결:** `handleLogout` 함수 자체를 `useEffect` 안으로 옮겼다. `useEffect`가 의존하는 `setChangeLogin`과 `history`는 의존성 배열에 명시했다.

```javascript
// 02Logout.js (ESLint 수정 후)

const Logout = () => {
  const { setChangeLogin } = useContext(LoginUserContext);
  const history = useHistory();

  useEffect(() => {
    // 1. 비동기 함수를 useEffect 내부에 정의
    const processLogout = async () => {
      try {
        await UserService.logoutUser(); // 서버 세션 파괴
        setChangeLogin(null);           // 클라이언트 Context 초기화
        history.push('/login');         // 리다이렉트
      } catch (error) {
        console.error('로그아웃 에러:', error);
        setChangeLogin(null);
        history.push('/login');
      }
    };

    // 2. 내부에서 함수 호출
    processLogout();

  }, [setChangeLogin, history]); // 3. 실제 의존성 명시

  // 렌더링 (로그아웃 처리 중...)
  return (
    <div className="ViewGood" style={{ 
      display: 'flex', 
      justifyContent: 'center', 
      alignItems: 'center', 
      minHeight: '400px' 
    }}>
      <div>
        <h2>[ Logout ] 화면 준비중</h2>
        <p>로그아웃 처리 중입니다...</p>
      </div>
    </div>
  );
};
```
