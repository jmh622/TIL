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
