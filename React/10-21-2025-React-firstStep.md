# Today I Learned (TIL) - React 데이터 흐름 심화

오늘은 React에서 컴포넌트 간 데이터를 관리하고 공유하는 두 가지 중요한 패턴을 배웠습니다.

1.  **상태 끌어올리기 (Lifting State Up)**: 자식 컴포넌트의 이벤트를 통해 부모 컴포넌트의 상태를 변경하는 방법.
2.  **Context API**: 'Props Drilling' 문제없이 앱 전반에 걸쳐 데이터를 효율적으로 공유하는 방법.

---

## 오늘 배운 핵심 용어 정리

* **상태 끌어올리기 (Lifting State Up)**
    * 여러 자식 컴포넌트가 공유해야 하는 상태(State)가 있을 때, 해당 상태를 가장 가까운 공통 부모 컴포넌트로 이동시키는 디자인 패턴입니다.
    * 자식은 부모로부터 상태 값과 상태 변경 함수를 모두 Props로 전달받아, 이벤트 발생 시 부모의 함수를 호출하여 부모의 상태를 변경합니다.

* **단일 데이터 소스 (Single Source of Truth)**
    * 특정 상태 데이터는 반드시 하나의 컴포넌트(보통 부모)만이 소유해야 한다는 원칙입니다.
    * '상태 끌어올리기'는 이 원칙을 지키기 위한 방법입니다. 데이터가 한 곳에서만 관리되므로 예측 가능하고 디버깅이 쉬워집니다.

* **Props Drilling (프로퍼티 내리꽂기)**
    * 데이터가 필요한 컴포넌트가 트리의 매우 깊은 곳에 있을 때, 그 데이터를 사용하지 않는 중간의 여러 컴포넌트들을 거쳐 오직 전달만을 목적으로 Props를 계속 내려보내는 비효율적인 상황을 의미합니다.

* **Context API**
    * Props Drilling 문제를 해결하기 위해 React가 제공하는 기능입니다.
    * 컴포넌트 트리 전반에 걸쳐 데이터를 '전역적'으로 공유할 수 있게 해줍니다.

* **`createContext(defaultValue)`**
    * Context 객체를 생성하는 함수입니다.
    * `defaultValue`는 `Provider`를 찾지 못했을 때 사용될 기본값입니다.

* **`Context.Provider`**
    * Context의 값을 하위 컴포넌트들에게 *제공*하는 컴포넌트입니다.
    * `value` prop을 통해 전달할 값을 지정하며, 이 값이 변경되면 하위의 Consumer들이 리렌더링됩니다.

* **`useContext(Context)`**
    * `Provider`가 제공한 값을 *사용*하기 위한 React 훅(Hook)입니다.
    * 가장 가까운 상위 `Provider`의 `value`를 읽어옵니다.

---

## 1. 상태 끌어올리기 (Lifting State Up) - 043 예제

`041`, `042` 예제에서 겪었던 문제 (자식이 부모 상태를 변경 못 함 / 자식이 상태를 복사하여 데이터 불일치 발생)를 해결하는 올바른 React 패턴입니다.

**데이터 흐름:**
자식 컴포넌트 (`Header043`) ➡️ (이벤트 발생) ➡️ Props로 받은 부모 함수 (`changeNo`) 호출 ➡️ 부모 컴포넌트 (`App`) ➡️ (상태 변경) ➡️ 새로운 Props가 자식에게 전달 ➡️ 자식 컴포넌트 (`Header043`) 리렌더링

### 1.1. 부모 컴포넌트 (`04AppEventChangeState043.js`)

* 상태(`no`, `product`, `users`)와 상태 변경 함수(`changeNo`, `changeProduct`, `changeUsers`)를 모두 소유합니다.
* 자식 컴포넌트(`Header`)에게 **상태 값**과 **상태 변경 함수**를 **모두** Props로 전달합니다.

```javascript
// 04AppEventChangeState043.js

function App() {
    // ... (useState로 state 정의) ...

    // ... (state 변경 함수 정의) ...
    const changeNo = ()=>{
        console.log("App 의 ChangeNo()");
        setNo(no+1);
    }
    // ...

    return (
        <div className='ViewGood'>
            {/* ... (App의 버튼들) ... */}

            {/* 자식에게 '상태'와 '상태 변경 함수'를 모두 props로 전달 */}
            <Header no={no} product={product} users={users}
                changeNo={changeNo}
                changeProduct={changeProduct}
                changeUsers={changeUsers}
            />
        </div>
    );
}
```

### 1.2. 자식 컴포넌트 (`Header043.js`)

* 부모로부터 받은 상태 값과 함수들을 Props (구조 분해 할당)로 받습니다.
* **잘못된 방식(`myNo`)**과 **올바른 방식(`no`)**이 공존하여 비교가 가능합니다.
* '상위 컴포넌트...' 버튼의 `onClick` 이벤트에 **Props로 받은 부모의 함수 (`changeNo`)**를 직접 연결합니다.

```javascript
// Header043.js

// 부모로부터 '값'과 '함수'를 모두 props로 받음
const Header043 =({no, product, users, changeNo, changeProduct, changeUsers}) => {

    // ... '042' 예제의 잘못된 방식 (안티 패턴)
    const [myNo, setMyNo] = useState(no);
    const myChangeNo = ()=>{ setMyNo(myNo+1); }
    // ...

    return (
        <header className='ViewGood'>
            {/* 부모의 'no' state를 보여줌 */}
            <li>{no}</li>

            {/* '042'의 잘못된 방식: 자식의 myNo state만 변경됨 */}
            <button onClick={myChangeNo}>숫자 1씩 증가</button>

            {/* '043'의 올바른 방식: Props로 받은 부모의 함수를 호출 */}
            <button onClick={changeNo}>상위 컴포넌트 숫자 1씩 증가</button>
            <button onClick={changeProduct}>상위 컴포넌트 상품 정보 수정</button>
            <button onClick={changeUsers}>상위 컴포넌트 회원 정보 추가</button>
        </header>
    );
}
```

---

## 2. Context API - 05 예제

'Props Drilling' 문제를 해결하고, 앱 전반에 걸쳐 필요한 데이터를 쉽게 공유합니다.

### 2.1. Step 1: Context 생성 (`Context05.js`)

* `createContext()`를 사용하여 공유할 데이터의 '저장소(Context)'를 만듭니다.
* 이때 **기본값**을 설정합니다. (Provider가 없을 때 이 값이 사용됩니다)

```javascript
// Context05.js
import {createContext} from "react";

// 1. 기본값 1을 가진 NoContext 생성
const NoContext = createContext(1);
export default NoContext;

// 2. 객체를 기본값으로 가진 ProductContext 생성
const ProductContext = createContext(
    {no:100, name:'TV'}
);
export {ProductContext};

// 3. 배열을 기본값으로 가진 UsersContext 생성
export const UsersContext = createContext([
    {id:'user1', pwd: 1},
    {id:'user2', pwd: 2},
]);

// 4. 함수를 기본값으로 가진 addContext 생성
const add = (a,b) => a + b;
export const addContext = createContext(add);
```

### 2.2. Step 2: 데이터 제공 (`05AppContext.js`)

* `Context.Provider` 컴포넌트로 하위 컴포넌트들을 감싸고, `value` prop에 공유할 값을 전달합니다.
* `Provider`는 중첩하여 여러 개의 값을 동시에 제공할 수 있습니다.

```javascript
// 05AppContext.js
import NoContext, { ProductContext } from './05context/Context05';
import View from './05context/View05';

function App() {
    // ...

    return (
        <div className='ViewGood'>
            <h1>05 context연습</h1>

            {/* Provider로 감싸지 않은 View는 '기본값'을 사용함 */}
            <View/>

            {/* 1. NoContext.Provider가 value={1234}를 제공 */}
            <NoContext.Provider value={1234}>
                <View/>
            </NoContext.Provider>

            {/* 2. Provider 중첩 사용 */}
            <NoContext.Provider value={7777}>
                <ProductContext.Provider value={{no:8888, name:'Radio'}}>
                    {/* 이 View는 no=7777, product=Radio를 받음 */}
                    <View/>
                </ProductContext.Provider>
            </NoContext.Provider>
        </div>
    );
}
```

### 2.3. Step 3: 데이터 사용 (`View05.js`, `ViewOther05.js`)

* `useContext()` 훅을 사용하여, Props 없이도 `Provider`가 제공한 값에 즉시 접근할 수 있습니다.
* `ViewOther05`는 `View05`의 자식이지만, `View05`를 거치지 않고(`Props Drilling` 없이) Context 데이터에 직접 접근합니다.

```javascript
// View05.js
import React, {useContext} from 'react';
// import 해오는 것이 중요
import NoContext, {ProductContext, UsersContext, addContext} from "./Context05"; 
import ViewOther from "./ViewOther05";

const View05 = () => {
    // useContext 훅으로 Provider의 value를 바로 꺼내 씀
    const no = useContext(NoContext);
    const product = useContext(ProductContext);
    const users = useContext(UsersContext);
    const add = useContext(addContext);

    return (
        <div className='ViewGood'>
            <h1>View05 Component</h1>
            no: {no}<br/>
            product.no: {product.no}<br/>
            {/* ... */}
            {/* 자식 컴포넌트 호출 (props 전달 안 함) */}
            <ViewOther/> 
        </div>
    );
};
```

```javascript
// ViewOther05.js
import React, {useContext} from 'react';
import NoContext, {ProductContext} from "./Context05";

const ViewOther05 = () => {
    // App -> View05 -> ViewOther05 구조이지만,
    // props 없이 바로 App의 Provider가 제공한 값에 접근 가능
    const no = useContext(NoContext);
    const product = useContext(ProductContext);
    
    return (
        <div className='ViewGood'>
            <h1>ViewOther05 Component</h1>
            no: {no}<br/>
            product.name: {product.name}<br/>
        </div>
    );
};
```
