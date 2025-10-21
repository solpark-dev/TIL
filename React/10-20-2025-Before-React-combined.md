# Today I Learned (TIL) - React 핵심 개념 정리

## 1. React 컴포넌트 (Components)

React는 UI를 독립적이고 재사용 가능한 조각인 '컴포넌트'로 나누어 관리한다.

### 1.1. 함수형 컴포넌트 (Function Components)

- JavaScript 함수로 컴포넌트를 정의한다.
- 최근 React에서 권장하는 방식이며, 화살표 함수로도 작성이 가능하다.
- **예시 (`Nav.js`)**
  ```javascript
  function Nav(){
      console.log("Nav");
      return(
          <nav className='ViewGood'>
              <h1>[Nav Componet]</h1> 
              {/* ... */}
          </nav>
      );
  }
  export default Nav;
  ```
- **예시 (`Article.js` - 화살표 함수)**
  ```javascript
  const Article=()=>{
    console.log("Article");
    return(
      <article className='ViewGood'>
        <h1>[Article Componet]</h1>
        {/* ... */}
      </article>
    );
  }
  export default Article;
  ```

### 1.2. 클래스형 컴포넌트 (Class Components)

- `React.Component`를 상속받는 ES6 클래스로 컴포넌트를 정의한다.
- `render()` 메서드 안에서 JSX를 반환해야 한다.
- **예시 (`Header.js`)**
  ```javascript
  import React, { Component } from "react";

  class Header extends Component{
      render(){
          console.log("Header");
          return(
              <header className='ViewGood'>
                  <h1>[Header Componet]</h1>
                  {/* ... */}
              </header>  
          );
      }
  }
  export default Header;
  ```

## 2. JSX (JavaScript XML)

JavaScript를 확장한 문법으로, UI가 어떻게 생겨야 하는지 설명하기 위해 사용된다.

- **주요 규칙**
  1.  **하나의 Root 태그:** 모든 JSX 표현식은 반드시 하나의 최상위 태그(Element)로 감싸야 한다. (예: `<div>...</div>`)
  2.  **JavaScript 표현식:** JSX 내부에서 JavaScript 변수나 코드를 사용하려면 중괄호 `{}`를 사용한다.
  3.  **주석:** JSX 내의 주석은 `{/* ... */}` 형식을 사용한다.
  4.  **태그 닫기:** 모든 태그는 반드시 닫혀야 한다. (예: `<Header />` 또는 `<Header></Header>`)

## 3. Props (속성)

`props`는 부모 컴포넌트에서 자식 컴포넌트로 데이터를 전달하기 위한 속성이다. (읽기 전용)

- **데이터 전달 (부모: `02AppProps.js`)**
  - `data1`, `data2`라는 이름으로 문자열 값을 전달한다.
  - 여는 태그와 닫는 태그 사이의 내용은 `children` prop으로 전달된다.
  ```jsx
  <View data1="[props로]" data2="[상위에서]">[하위로 데이터전송]</View>
  ```
- **데이터 수신 (자식: `View02.js`)**
  - 자식 컴포넌트는 파라미터로 `props` 객체를 받는다.
  - `props.data1`, `props.data2`, `props.children` 등으로 값에 접근한다.
  ```javascript
  const View02 = (props) => {
      console.log(props);
      return (
          <div className={"ViewGood"}>
              <h1>[View02 Component]</h1>
              상위컴포넌트 data 받기 : data1={props.data1} &nbsp;&nbsp; data2={props.data2}
              <br/>
              상위컴포넌트 data 받기: children={props.children}
          </div>
      );
  };
  ```

## 4. State (상태)와 `useState` 훅

`state`는 컴포넌트 내부에서 변하는 동적인 데이터를 관리한다. `state`가 변경되면 컴포넌트가 리렌더링(re-rendering)된다.

- **`useState` 훅 사용 (`03AppState032.js`)**
  - `useState`는 React에서 state를 사용하기 위한 훅(Hook)이다.
  - `const [상태값, 상태설정함수] = useState(초기값);` 형태로 사용한다.
  - 숫자, 객체, 배열 등 다양한 형태의 데이터를 state로 관리할 수 있다.
  ```javascript
  import {useState} from "react";

  function App() {
      // 숫자 state
      const [no, setNo] = useState(1);
      // 객체 state
      const [product, setProduct] = useState({no:100, name:'TV'});
      // 배열 state
      const [users, setUsers] = useState([
          { id: 'user1', pwd: 1 },
          { id: 'user2', pwd: 2 }
      ]);

      // ...

      // state 값을 props로 자식에게 전달
      return (
          <div className='ViewGood'>
              {/* ... */}
              <Header no={no} product={product} users={users}></Header>
          </div>
      );
  }
  ```

## 5. 리스트 렌더링 (List Rendering)

배열 데이터를 화면에 목록 형태로 렌더링할 때는 JavaScript의 `.map()` 메서드를 주로 사용한다.

- **`.map()` 사용 (`03AppState032.js`, `Header032.js`)**
  - 배열(예: `users`)을 순회하며 각 요소를 JSX 엘리먼트(예: `<li>`)로 변환한다.
- **`key` 속성**
  - `.map()`으로 리스트를 생성할 때는 각 엘리먼트에 고유한 `key` prop을 지정해야 한다.
  - React가 리스트의 변경 사항을 효율적으로 식별하기 위해 `key`가 필요하다. (여기서는 `index`를 사용)
- **예시 (`Header032.js`)**
  ```javascript
  const Header032 = (props) => {
      // ...
      const usersView = props.users.map((user, index) => {
          // key={index} 사용
          return <li key={index}>{user.id} {user.pwd}</li>
      })

    return (
      <header className='ViewGood'>
          {/* ... */}
          {usersView} {/* map()의 결과(JSX 배열)를 렌더링 */}
      </header>
    );
  };
  ```

## 6. 커스텀 훅 (Custom Hooks)

컴포넌트 간에 재사용 가능한 로직을 분리하여 `use`로 시작하는 함수로 만드는 것을 '커스텀 훅'이라고 한다.

- **커스텀 훅 정의 (`useMessage.js`)**
  - `use`로 시작하는 함수를 만든다.
  - 필요한 로직을 수행하고, 컴포넌트에서 사용할 값이나 함수를 반환한다.
  ```javascript
  const useMessage = (message) => {

      const displayMessage = (prefix) => {
          console.log(`${prefix}: ${message}`);
      };

      // 객체 형태로 반환
      return {message, displayMessage};
  };
  export default useMessage;
  ```
- **커스텀 훅 사용 (`02-1AppUseHook.js`)**
  - `import` 하여 일반 함수처럼 호출한다.
  - 반환된 값을 구조 분해 할당(destructuring)하여 사용한다.
  ```javascript
  import useMessage from './util/useMessage';

  function App() {
      // ...
      // 커스텀 훅 사용
      const {displayMessage} = useMessage("Hello, react Hook!");
      displayMessage("접두어 : ");

      return (
          // ...
      );
  }
  ```
