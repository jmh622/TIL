# JSX

```jsx
const element = <h1>Hello, world!</h1>;
```

이 문법은 문자열도, HTML도 아니다.

이를 JSX라고 하며 자바스크립트 확장 문법이다.

JSX는 React 엘리먼트를 생성한다.

## JSX에 표현식 포함하기

`name`이라는 변수를 JSX에 포함시키려면 변수를 중괄호로 감싸주면 된다.

```jsx
const name = 'Jung MyoungHoon';
const element = <h1>Hello, {name}</h1>;
```

변수 외에도 모든 자바스크립트 표현식을 넣을 수 있다. (`2 + 2`, `user.firstName`, `formatName(user)` 등)

JSX를 여러 줄로 나눠 쓸 수도 있다. 자동 세미콜론 삽입을 방지하기 위하여 괄호로 묶어주는 것을 권장한다.

```
const name = 'Jung MyoungHoon';
const element = (
  <h1>
    Hello, {name}
  </h1>
);
```

## JSX 속성 정의

따옴표를 이용하여 문자열 리터럴로 정의할 수 있다.

```jsx
const element = <a href="https://www.reactjs.org"> link </a>;
```

중괄호를 사용해 자바스크립트 표현식을 사용할 수 있다. 중괄호 사용 시 따옴표를 입력하면 안된다.

```jsx
const element = <img src={user.avatarUrl}></img>;
```

## 사용자 입력 주입 공격 방지

React DOM은 JSX에 삽입된 모든 값을 렌더링하기 전에 이스케이프하므로, XSS 공격을 방지할 수 있다.

```jsx
const title = response.potentiallyMaliciousInput;
// 이것은 안전합니다.
const element = <h1>{title}</h1>;
```

# Rendering Elements

엘리먼트는 컴포넌트와 다른 개념이다. 엘리먼트는 컴포넌트의 구성 요소이다.

엘리먼트는 일반 객체(plain object)이다.

## DOM에 엘리먼트 렌더링하기

리액트로 구현된 애플리케이션은 일반적으로 하나의 루트 DOM 노드가 존재한다.

```jsx
<div id="root"></div>
```

해당 div에 포함된 모든 엘리먼트는 리액트 DOM에서 관리한다. 이때 이 div를 루트 DOM 노드라고 한다.

리액트 엘리먼트를 루트 DOM 노드에 렌더링하는 방법은 `ReactDOM.render()`를 이용하면 된다.

```jsx
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

## 이미 렌더링된 엘리먼트 업데이트

리액트 엘리먼트는 `불변객체`이다. 엘리먼트를 생성한 이후에는 자식이나 속성 변경이 불가능하다.

UI를 업데이트하는 유일한 방법은 새로운 엘리먼트를 생성하는 것이다.

```jsx
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(element, document.getElementById('root'));
}

setInterval(tick, 1000);
```

리액트 DOM은 해당 엘리먼트와 그 자식 엘리먼트를 이전의 엘리먼트와 비교하고 DOM을 원하는 상태로 만드는데 필요한 경우에만 DOM을 업데이트한다.

따라서 위 예시의 경우에 div 포함한 전체를 엘리먼트로 계속 만들지만 리액트 DOM은 변경된 내용인 시간 텍스트 노드만 업데이트한다.

# Component, Props

컴포넌트를 통해 UI를 재사용 가능한 개별적인 여러 조각으로 나누고, 각 조각을 개별적으로 살펴볼 수 있다.

개념적으로 컴포넌트는 자바스크립트 함수와 유사하다. `props`라는 임의의 입력을 받은 후 리액트 엘리먼트를 반환한다.

## 함수 컴포넌트

컴포넌트를 정의하는 가장 간단한 방법은 자바스크립트 함수를 작성하는 것이다.

```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

## 클래스 컴포넌트

`ES6`의 클래스를 사용하여 컴포넌트를 정의할 수 있다.

```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

## 컴포넌트 렌더링

DOM 태그로 리액트 엘리먼트를 만드는 방법 외에도 사용자 정의 컴포넌트로도 만들 수 있다.

```jsx
const element = <Welcome name="Sara" />;
```

리액트는 사용자 정의 컴포넌트로 작성한 엘리먼트를 발견하면 JSX 어트리뷰트와 자식을 해당 컴포넌트에 단일 객체로 전달한다. 이 단일 객체를 `props`라고 한다.

```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Sara" />;
ReactDOM.render(element, document.getElementById('root'));
```

컴포넌트 이름은 항상 대문자로 시작해야 한다. 컴포넌트명을 소문자로 작성하면 리액트는 DOM 태그로 처리한다.

`ReactDom.render()`를 이용하면 `<Welcome name="Sara" />`의 `name`이 `props`에 들어가게 된다. 따라서 결과적으로 `<h1>Hello, Sara</h1>`가 반환되게 된다.

## 컴포넌트 합성

컴포넌트는 자신의 출력에 다른 컴포넌트를 참조할 수 있다.

다음과 같이 `Welcome`을 여러 번 렌더링하는 `App` 컴포넌트를 만들 수 있다.

```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

function App() {
  return (
    <div>
      <Welcome name="Sara" />
      <Welcome name="Cahal" />
      <Welcome name="Edite" />
    </div>
  );
}

ReactDOM.render(<App />, document.getElementById('root'));
```

## props는 읽기 전용

리액트는 매우 유연하지만 한 가지 엄격한 규칙이 있다.

컴포넌트는 `props`를 수정해서는 안된다.

모든 리액트 컴포넌트는 자신의 `props`를 다룰 때 반드시 순수 함수처럼 동작해야 한다.

# State와 생명주기

이전의 Clock 컴포넌트에 state와 생명주기를 적용하면 다음과 같다.

클래스 생성자에서 `props`를 부모생성자에 전달해줘야 한다.

```jsx
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = { date: new Date() };
  }

  componentDidMount() {
    this.timerID = setInterval(() => this.tick(), 1000);
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date(),
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(<Clock />, document.getElementById('root'));
```

## State

컴포넌트와 관련된 일부 데이터가 시간에 따라 변경될 경우 사용.

`state`와 `props`의 가장 중요한 차이점은 `props`는 부모 컴포넌트로부터 전달받고, `state`는 컴포넌트에 의해 관리된다는 것이다.

컴포넌트는 `props`를 변경할 수 없지만, `state`는 변경할 수 있다.

**state는 캡슐화되어 있다.**

**state는 클래스 생성자 내에서 초기화해줄 수 있다.**

```jsx
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = { date: new Date() };
  }
}
```

**state를 직접적으로 수정하면 안된다.**

```jsx
// Wrong
this.state.comment = 'Hello';

// Correct
this.setState({ comment: 'Hello' });
```

**state를 업데이트할 때는 `setState`를 이용해야 한다.**

| state는 비동기적으로 업데이트될 수 있기 때문에 state를 수정할 때 직접 접근하지 않아야 한다.

```jsx
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});

// Correct
this.setState((state, props) => ({
  counter: state.counter + props.increment,
}));
```

**`setState`는 state 전체를 덮어씌우는 것이 아니라 병합이다.**

```jsx
constructor(props) {
  super(props);
  this.state = {
    posts: [],
    comments: []
  };
}

componentDidMount() {
  fetchPosts().then(response => {
    this.setState({
      posts: response.posts
    });
  });

  fetchComments().then(response => {
    this.setState({
      comments: response.comments
    });
  });
}
```

병합은 얕게 이루어지기 때문에 `this.setState({comments})`는 `this.state.posts`에 영향을 주진 않지만 `this.state.comments`는 완전히 대체된다.

## 생명주기

`componentDidMount`는 컴포넌트가 렌더링되면 실행된다.

`componentWillUnmount`는 컴포넌트가 DOM으로부터 삭제된다면 실행된다.

## Virtual DOM

가상 돔(VDOM)은 UI의 이상적인 또는 "가상"적인 표현을 메모리에 저장하고 ReactDOM과 같은 라이브러리에 의해 "실제" DOM과 동기화하는 프로그래밍 개념이다. 이 과정을 재조정(Reconciliation)이라고 한다.

자바스크립트를 통해 DOM을 업데이트하려면 변경된 부분만 업데이트하는 것이 아니라 해당하는 전체 DOM을 다 업데이트하게 된다. 따라서 비용이 많이 발생하게 된다. 이를 해결하기 위해 가상 돔의 개념이 생겨났고 가상 돔은 변경하려는 부분만 업데이트하기 때문에 적은 비용으로 최적의 성능을 낼 수 있다.

# 이벤트 처리

- 이벤트 명칭은 camelCase를 사용
- jsx를 사용하여 문자열이 아닌 함수로 이벤트 핸들러 전달
- 기본 동작을 방지하려면 무조건 `preventDefault`를 호출해야 함
- 핸들러 내에서 `this`를 사용하려면 `Function.prototype.bind()`를 사용하여 `this`를 바인딩해주어야 함
  - 이 외의 방법이 있긴 하지만 기본적으로 이 방법을 이용
- 루프 등에서 이벤트 핸들러에 인자를 전달하라면 이벤트 핸들러에 추가적인 매개변수를 전달

```js
// 일반적인 예시
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    this.state = { isToggleOn: true };

    // 콜백에서 `this`가 작동하려면 아래와 같이 바인딩 해주어야 합니다.
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState((prevState) => ({
      isToggleOn: !prevState.isToggleOn,
    }));
  }

  render() {
    return <button onClick={this.handleClick}>{this.state.isToggleOn ? 'ON' : 'OFF'}</button>;
  }
}

ReactDOM.render(<Toggle />, document.getElementById('root'));
```

```js
// 추가적인 매개변수 전달
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

# 조건부 렌더링

- 조건부 렌더링은 javascript의 조건 처리와 동일하게 동작
  - if-else
  - 3항 연산자
  - 논리 연산자

```js
// if-else
render() {
  const isLoggedIn = this.state.isLoggedIn;
  return (
    <div>
      {isLoggedIn
        ? <LogoutButton onClick={this.handleLogoutClick} />
        : <LoginButton onClick={this.handleLoginClick} />
      }
    </div>
  );
}
```

```js
// 3항 연산자
render() {
  const isLoggedIn = this.state.isLoggedIn;
  return (
    <div>
      The user is <b>{isLoggedIn ? 'currently' : 'not'}</b> logged in.
    </div>
  );
}
```

```js
// 논리 연산자
function Mailbox(props) {
  const unreadMessages = props.unreadMessages;
  return (
    <div>
      <h1>Hello!</h1>
      {unreadMessages.length > 0 && <h2>You have {unreadMessages.length} unread messages.</h2>}
    </div>
  );
}

const messages = ['React', 'Re: React', 'Re:Re: React'];
ReactDOM.render(<Mailbox unreadMessages={messages} />, document.getElementById('root'));
```

# 오류 처리

렌더링 과정에서 오류가 발생하는 경우 'Error Boundary'를 이용하여 처리할 수 있다.

'Error Boundary(에러 경계)'란 하위 컴포넌트 트리의 어디에서든 자바스크립트 에러를 기록하며 깨진 컴포넌트 트리 대신 폴백 UI를 보여주는 리액트 컴포넌트이다.

에러 경계는 다음과 같은 에러는 포착하지 않는다.

- 이벤트 핸들러
- 비동기적 코드
- 서버 사이드 렌더링
- 자식으로부터가 아닌, 에러 경계 자체에서 발생하는 에러

생명주기 메서드 중 `static getDerivedStateFromError`와 `componentDidCatch`를 이용하여 컴포넌트를 구성하면 된다.

- static getDerivedStateFromError
  - 에러가 발생한 뒤에 폴백 UI를 렌더링 할 때 사용
- componentDidCatch
  - 에러 정보를 기록할 때 사용

```js
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // 다음 렌더링에서 폴백 UI가 보이도록 상태를 업데이트 합니다.
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // 에러 리포팅 서비스에 에러를 기록할 수도 있습니다.
    logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // 폴백 UI를 커스텀하여 렌더링할 수 있습니다.
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children;
  }
}

<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>;
```

# 합성

하나의 컴포넌트 내에서 다른 여러 컴포넌트를 사용하는 것을 합성(Composition)이라고 한다.

컴포넌트의 자식 컴포넌트, 엘리먼트는 `props.children`에 담기게 된다.

클래스 컴포넌트의 경우, 상속을 사용하는 권장되지 않는다.

```js
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">{props.title}</h1>
      <p className="Dialog-message">{props.message}</p>
      {props.children}
    </FancyBorder>
  );
}

class SignUpDialog extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.handleSignUp = this.handleSignUp.bind(this);
    this.state = { login: '' };
  }

  render() {
    return (
      <Dialog title="Mars Exploration Program" message="How should we refer to you?">
        <input value={this.state.login} onChange={this.handleChange} />
        <button onClick={this.handleSignUp}>Sign Me Up!</button>
      </Dialog>
    );
  }

  handleChange(e) {
    this.setState({ login: e.target.value });
  }

  handleSignUp() {
    alert(`Welcome aboard, ${this.state.login}!`);
  }
}
```

# Ref

`ref`는 `render` 메서드에서 생성된 DOM 노드나 리액트 엘리먼트에 접근하는 방법을 제공한다.

보통 자식 엘리먼트를 수정하는 경우에는 새로운 `props`를 전달하여 자식을 다시 렌더링해야 한다.

그러나 가끔 일반적이지 않은 경우에는 `ref`를 이용하여 처리한다.

보통의 경우에는 `ref`를 사용하지 않고 부모 컴포넌트에서 `props` 혹은 `state`를 이용하여 처리하도록 하자.

함수 컴포넌트는 인스턴스가 없기 때문에 `ref`를 사용할 수 없다.

## Ref를 사용해야 할 때

- 포커스, 텍스트 선택영역, 미디어의 재생을 관리할 때
- 애니메이션을 직접적으로 실행시킬 때
- 서드 파티 DOM 라이브러리를 리액트와 같이 사용할 때

## Ref 생성

`React.createRef` 메서드를 통해 생성한다.

```js
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.myRef = React.createRef();
  }
  render() {
    return <div ref={this.myRef} />;
  }
}
```

## Ref 접근

`render` 메서드 안에서 `ref`가 엘리먼트에 전달되었을 때, 그 노드를 향한 참조는 `ref`의 `current` 어트리뷰트에 담기게 된다.

```js
const node = this.myRef.current;
```

```js
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    // textInput DOM 엘리먼트를 저장하기 위한 ref를 생성합니다.
    this.textInput = React.createRef();
    this.focusTextInput = this.focusTextInput.bind(this);
  }

  focusTextInput() {
    // DOM API를 사용하여 명시적으로 text 타입의 input 엘리먼트를 포커스합니다.
    // 주의: 우리는 지금 DOM 노드를 얻기 위해 "current" 프로퍼티에 접근하고 있습니다.
    this.textInput.current.focus();
  }

  render() {
    // React에게 우리가 text 타입의 input 엘리먼트를
    // 우리가 생성자에서 생성한 `textInput` ref와 연결하고 싶다고 이야기합니다.
    return (
      <div>
        <input type="text" ref={this.textInput} />
        <input type="button" value="Focus the text input" onClick={this.focusTextInput} />
      </div>
    );
  }
}
```

## Ref 전달하기

드문 경우이지만 자식 컴포넌트의 `ref`를 사용하는 것이 아니라, 반대로 자식 컴포넌트의 `ref`를 부모에게 공개해야 할 때가 있다.

`React.forwardRef` 메서드를 통하여 부모 컴포넌트에게 `ref`를 공개할 수 있다.

```js
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// 이제 DOM 버튼으로 ref를 작접 받을 수 있습니다.
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;
```

# 재조정 (Reconciliation)

state나 props가 갱신되면 `render` 함수는 새로운 리액트 엘리먼트 트리를 반환할 것이다.

이때 리액트는 방금 만들어진 트리에 맞게 가장 효과적으로 UI를 갱신하는 방법을 알아낼 필요가 있다.

리액트는 아래의 두 가지 가정을 기반하여 O(n) 복잡도의 휴리스틱 알고리즘을 구현했다.

1. 서로 다른 타입의 두 엘리먼트는 서로 다른 트리를 만든다.
2. `key` prop을 통해, 여러 렌더링 사이에서 어떤 자식 엘리먼트가 변경되지 않아야 할지 표시해 줄 수 있다.

## 비교 알고리즘 (Diffing Algorithm)

두 개의 트리를 비교할 때, 리액트는 두 엘리먼트의 루트 엘리먼트부터 비교합니다. 이후의 동작은 루트 엘리먼트의 타입에 따라 달라진다.

- 엘리먼트 타입이 다른 경우
- 엘리먼트 타입이 같은 경우
- 같은 타입의 컴포넌트 엘리먼트

### 엘리먼트 타입이 다른 경우

두 루트 엘리먼트의 타입이 다르면, 리액트는 이전 트리를 버리고 완전히 새로운 트리를 구축한다.

트리를 버릴 때 이전 DOM 노드들은 모두 파괴된다. 컴포넌트 인스턴스는 `componentWillUnmount` 함수를 실행한다.

그리고 새로운 트리가 만들어질 때, 새로운 DOM 노드들이 DOM에 삽입된다. 그에 따라 컴포넌트 인스턴스는 `componentDidmount` 함수를 실행한다.

이전 트리와 연관된 모든 state는 사라진다.

```js
<div>
  <Counter />
</div>

<span>
  <Counter />
</span>
```

위와 같은 비교가 일어나면, 이전 `Counter`는 사라지고, 새로 다시 마운트가 될 것이다.

### 엘리먼트 타입이 같은 경우

같은 타입의 두 리액트 DOM 엘리먼트를 비교할 때, 리액트는 두 엘리먼트의 속성을 확인하여 동일한 내역은 유지하고 변경된 속성들만 갱신한다.

```js
<div className="before" title="stuff" />

<div className="after" title="stuff" />
```

이 두 엘리먼트를 비교하면 리액트는 현재 DOM 노드 상에 `className`만 수정한다.

`style`이 갱신될 때, 리액트는 또한 변경된 속성만을 갱신한다.

DOM 노드의 처리가 끝나면, 리액트는 이어서 해당 노드의 자식들을 재귀적으로 처리한다.

### 같은 타입의 컴포넌트 엘리먼트

컴포넌트가 갱신되면 인스턴스는 동일하게 유지되어 렌더링 간 state가 유지됩니다.

리액트는 새로운 엘리먼트의 내용을 반영하기 위해 현재 컴포넌트 인스턴스의 props를 갱신한다. 이 때 `componentDidUpdate`를 호출한다.

그 다음으로 `render`를 호출하고 비교 알고리즘이 이전 결과와 새로운 결과를 재귀적으로 처리한다.

## 자식에 대한 재귀적 처리

DOM 노드의 자식들을 재귀적으로 처리할 때 리액트는 기본적으로 동시에 두 리스트를 순회하고 차이점이 있으면 변경을 생성한다.

```js
<ul>
  <li>first</li>
  <li>second</li>
</ul>

<ul>
  <li>first</li>
  <li>second</li>
  <li>third</li>
</ul>
```

위 예시는 다음과 같이 동작한다.

1. 리액트는 두 트리에서 `<li>first</li>`, `<li>second</li>` 순으로 일치하는 지 확인한다.
2. 마지막으로 `<li>third</li>`를 트리에 추가한다.

```js
<ul>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

<ul>
  <li>Connecticut</li>
  <li>Duke</li>
  <li>Villanova</li>
</ul>
```

위 예시는 다음과 같이 동작한다.

1. 첫 번째 `li`태그부터 내용이 다르므로 전체를 다 변경한다.
2. 단순히 맨 앞에 `<li>Connecticut</li>`만 추가했을 뿐인데 비효율적으로 동작하게 된다.

이러한 비효율은 `key` prop을 통해 해결할 수 있다.

## Keys

자식 엘리먼트들이 `key` prop을 가지고 있다면, 리액트는 `key`를 통해 기존 트리와 이후 트리의 자식들이 일치하는지 확인한다.

```js
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
```

이제는 맨 앞에 `<li key="2014">Connecticut</li>`가 추가만 되었다는 것을 알 수 있다.

key는 형제 사이에서만 유일하면 된다. 전역에서 유일할 필요는 없다.

배열에서 인덱스로 key를 할당할 수도 있다. 하지만 배열이 재배열되거나 하면 key가 변경될 수 있고, 컴포넌트의 state와 관련된 문제가 발생할 수 있다.

따라서 key는 고유하고 변하지 않는 값을 사용해야 한다.
