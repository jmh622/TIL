# Hooks 란?

> 함수 컴포넌트에서 state를 사용할 수 있게 해주는 기술

# State Hook

> state 변수 관리

```js
import React, { useState } from 'react';

function Example() {
  // "count"라는 새 상태 변수를 선언합니다
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

`useState`가 바로 Hook이다.

`count`는 getter, `setCount`는 setter라고 생각하면 이해하기 쉽다.

`useState(0)`의 `0`은 state의 초기값이다.

## 여러 State 변수 선언

```js
function ExampleWithManyStates() {
  // 상태 변수를 여러 개 선언했습니다!
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
  // ...
}
```

## 클래스 컴포넌트의 state와 다른 점

**state 업데이트 함수의 동작이 다르다.**

클래스 컴포넌트의 `this.setState`의 경우 state 변경 시, 병합으로 처리되어 변경되지 않은 state는 자동으로 유지가 된다.

하지만 State Hook의 state setter 함수는 병합이 아닌 **대체(덮어씌움)** 이다.

# Effect Hook

> state가 업데이트됨에 따라 발생하는 side effect 구현

```js
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // componentDidMount, componentDidUpdate와 비슷합니다
  useEffect(() => {
    // 브라우저 API를 이용해 문서의 타이틀을 업데이트합니다
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

## 여러 useEffect 사용

관심사, 로직을 분리하기 위하여 `useEffect`를 여러 개 호출할 수 있다.

```js
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
  // ...
}
```

## 구독 해지

이벤트 해지나 구독 해지를 해야 하는 경우에는 해당 로직을 담고 있는 함수를 반환하면 된다.

```js
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

## 호출 제한

기본적으로 `useEffect`는 렌더링 이후 항상 호출된다.

state가 최초 설정될 때만 호출한다거나, state 최초 설정 시에는 호출하지 않고 업데이트될 때만 호출한다거나 등 호출 제한을 할 수 있다.

### 마운트 시에만 호출(clean-up 함수를 반환한다면 마운트 해제도 포함)

`useEffect` 두 번째 인수로 빈 배열을 넘기면 된다.

```js
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, []); // 마운트 시에만 실행
```

### state 업데이트 시에만 호출

`useEffect` 두 번째 인수로 state가 담긴 배열을 넘기면 된다.

```js
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // count가 바뀔 때만 effect를 재실행합니다.
```

# Hook 사용 규칙

1. 최상위에서만 사용해야 한다.
   1. 조건문, 반복문 등 안에서 Hook 호출은 ❌
2. 함수 컴포넌트와 Custom Hook 내에서만 사용해야 한다.

## 최상위에서만 사용해야 한다.

state와 effect를 여러 개 사용할 수 있다는 것은 앞에서 설명했다.

만약 Hook을 최상위에서 호출하지 않았다면 Hook 로직이 꼬이게 되어 오류가 발생한다.

조건문이나 반복문이 필요하다면 Hook 내에서 처리하도록 하자.

### 올바른 예

```js
function Form() {
  // 1. name이라는 state 변수를 사용하세요.
  const [name, setName] = useState('Mary');

  // 2. Effect를 사용해 폼 데이터를 저장하세요.
  useEffect(function persistForm() {
    localStorage.setItem('formData', name);
  });

  // 3. surname이라는 state 변수를 사용하세요.
  const [surname, setSurname] = useState('Poppins');

  // 4. Effect를 사용해서 제목을 업데이트합니다.
  useEffect(function updateTitle() {
    document.title = name + ' ' + surname;
  });

  // ...
}
```

위 코드는 다음의 실행순서를 가진다.

```
// ------------
// 첫 번째 렌더링
// ------------
useState('Mary')           // 1. 'Mary'라는 name state 변수를 선언합니다.
useEffect(persistForm)     // 2. 폼 데이터를 저장하기 위한 effect를 추가합니다.
useState('Poppins')        // 3. 'Poppins'라는 surname state 변수를 선언합니다.
useEffect(updateTitle)     // 4. 제목을 업데이트하기 위한 effect를 추가합니다.

// -------------
// 두 번째 렌더링
// -------------
useState('Mary')           // 1. name state 변수를 읽습니다.(인자는 무시됩니다)
useEffect(persistForm)     // 2. 폼 데이터를 저장하기 위한 effect가 대체됩니다.
useState('Poppins')        // 3. surname state 변수를 읽습니다.(인자는 무시됩니다)
useEffect(updateTitle)     // 4. 제목을 업데이트하기 위한 effect가 대체됩니다.

// ...
```

### 잘못된 예

```js
function Form() {
  // 1. name이라는 state 변수를 사용하세요.
  const [name, setName] = useState('Mary');

  // 2. Effect를 사용해 폼 데이터를 저장하세요.
  // 🔴 조건문에 Hook을 사용함으로써 첫 번째 규칙을 깼습니다
  if (name !== '') {
    useEffect(function persistForm() {
      localStorage.setItem('formData', name);
    });
  }

  // 3. surname이라는 state 변수를 사용하세요.
  const [surname, setSurname] = useState('Poppins');

  // 4. Effect를 사용해서 제목을 업데이트합니다.
  useEffect(function updateTitle() {
    document.title = name + ' ' + surname;
  });

  // ...
}
```

위 코드는 다음과 같이 처리되며 오류가 발생한다.

```
useState('Mary')           // 1. name state 변수를 읽습니다. (인자는 무시됩니다)
// useEffect(persistForm)  // 🔴 Hook을 건너뛰었습니다!
useState('Poppins')        // 🔴 2 (3이었던). surname state 변수를 읽는 데 실패했습니다.
useEffect(updateTitle)     // 🔴 3 (4였던). 제목을 업데이트하기 위한 effect가 대체되는 데 실패했습니다.
```

# Custom Hook 만들기

> 상태 관련 로직을 컴포넌트 간에 재사용하기 위하여 별도의 Hook을 작성

```js
import React, { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}

function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}

function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return <li style={{ color: isOnline ? 'green' : 'black' }}>{props.friend.name}</li>;
}
```

## 주의사항

1. 이름이 `use`로 시작해야 한다.
2. 각 컴포넌트에서 동일한 Custom Hook을 사용하여 state를 받더라도 각 state는 완전히 독립적이다.

# 그 이외의 Hooks
