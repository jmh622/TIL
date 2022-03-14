# State 끌어올리기

동일한 데이터에 대한 변경사항을 여러 컴포넌트에 반영해야 할 경우, 가장 가까운 공통 조상으로 state를 끌어올리는 것이 좋다.

이를 'state 끌어올리기'라고 한다.

## State 끌어올리기 요약

1. 맨 처음에는 하나의 컴포넌트에서 자신의 state로 render를 한다.
2. 그러다가 다른 컴포넌트가 추가되었는 데 이 컴포넌트에서 사용하는 state가 기존에 존재하던 컴포넌트의 state와 동일하다.
3. 각 컴포넌트에서 소유한 state가 동일한 값이지만, 컴포넌트간의 state 공유가 되지 않는다.
4. 동일한 state를 사용하기 위하여 컴포넌트들의 가장 가까운 공통의 조상(부모) 컴포넌트가 해당 state를 가지도록 변경한다.
5. 그럼 자식 컴포넌트들(기존에 동일한 state를 가져야 했던 컴포넌트들)은 부모로부터 해당 값을 props로 전달받게 된다.
6. 부모 컴포넌트가 동일한 값을 props로 전달하기 때문에 서로 동일한 값을 가질 수 있게 된다.

## 코드로 보기

### 최초의 컴포넌트

props로 전달받은 `celsius`값에 따라 물이 끓기에 충분하지 나타내는 컴포넌트가 있다.

```js
function BoilingVerdict(props) {
  if (props.celsius >= 100) {
    return <p>The water would boil.</p>;
  }
  return <p>The water would not boil.</p>;
}
```

그리고 이 `BoilingVerdict` 컴포넌트를 가지는 또다른 컴포넌트가 있다.

이 컴포넌트(`Calculator`)는 사용자로부터 온도(`celsius(state)`)를 입력받아 `BolingVerdict`에게 전달한다.

```js
class Calculator extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = { temperature: '' };
  }

  handleChange(e) {
    this.setState({ temperature: e.target.value });
  }

  render() {
    const temperature = this.state.temperature;
    return (
      <fieldset>
        <legend>Enter temperature in Celsius:</legend>
        <input value={temperature} onChange={this.handleChange} />
        <BoilingVerdict celsius={parseFloat(temperature)} />
      </fieldset>
    );
  }
}
```

### 새로운 요구사항

이 때 새 요구사항으로써 화씨 입력 필드를 추가하고 두 필드 간에 온도값을 동기화하도록 해보자.

이를 위하여 기존의 `Calculator` 컴포넌트 중에서 일부를 빼내어 `TemperatureInput` 컴포넌트를 새로 만들었다.

이 컴포넌트는 props를 통해 `scale`을 전달 받는다. 그리고 사용자의 입력값을 통해 `temperature`를 state로 가지게 된다.

```js
const scaleNames = {
  c: 'Celsius',
  f: 'Fahrenheit',
};

class TemperatureInput extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = { temperature: '' };
  }

  handleChange(e) {
    this.setState({ temperature: e.target.value });
  }

  render() {
    const temperature = this.state.temperature;
    const scale = this.props.scale;
    return (
      <fieldset>
        <legend>Enter temperature in {scaleNames[scale]}:</legend>
        <input value={temperature} onChange={this.handleChange} />
      </fieldset>
    );
  }
}
```

그럼 `Calculator` 컴포넌트는 다음과 같이 변경된다.

```js
class Calculator extends React.Component {
  render() {
    return (
      <div>
        <TemperatureInput scale="c" />
        <TemperatureInput scale="f" />
      </div>
    );
  }
}
```

### state 끌어올리기가 필요한 시점

변경된 이 컴포넌트가 rendering되면 UI상으로는 전혀 문제가 되지 않는다. 하지만 각각의 `TemperatureInput` 컴포넌트는 각각의 state를 가지게 되기 때문에 서로 상호작용하여 온도값을 동기화시킬 수 없다.

따라서 이때 state 끌어올리기가 필요하다.

동일한 값을 공유해야하는 `TemperatureInput` 컴포넌트의 가장 가까운 공통의 조상인 `Calculator` 컴포넌트에서 해당 값(온도값)을 state로 가지고 있어야 하는 것이다.

### 최종 Calculator 컴포넌트

변경된 `Calculator` 컴포넌트는 다음과 같다.

`temperature`와 `scale`을 state로 가지고 있고, 이를 각 `TemperatureInput` 컴포넌트에 알맞은 온도값을 변경하여 props로 전달해준다.

그리고 각 `TemperatureInput` 컴포넌트에서 사용자 입력을 받기 때문에 이를 처리하기 위한 이벤트 처리 함수도 `Calculator` 컴포넌트에서 만들어서 props로 전달해준다. 그 이유는 변경되어야 하는, 공유해야 하는 온도값을 `Calculator` 컴포넌트가 가지고 있기 때문이다.

```js
class Calculator extends React.Component {
  constructor(props) {
    super(props);
    this.handleCelsiusChange = this.handleCelsiusChange.bind(this);
    this.handleFahrenheitChange = this.handleFahrenheitChange.bind(this);
    this.state = { temperature: '', scale: 'c' };
  }

  handleCelsiusChange(temperature) {
    this.setState({ scale: 'c', temperature });
  }

  handleFahrenheitChange(temperature) {
    this.setState({ scale: 'f', temperature });
  }

  render() {
    const scale = this.state.scale;
    const temperature = this.state.temperature;
    const celsius = scale === 'f' ? tryConvert(temperature, toCelsius) : temperature;
    const fahrenheit = scale === 'c' ? tryConvert(temperature, toFahrenheit) : temperature;

    return (
      <div>
        <TemperatureInput scale="c" temperature={celsius} onTemperatureChange={this.handleCelsiusChange} />
        <TemperatureInput scale="f" temperature={fahrenheit} onTemperatureChange={this.handleFahrenheitChange} />
        <BoilingVerdict celsius={parseFloat(celsius)} />
      </div>
    );
  }
}
```

### 최종 TemperatureInput 컴포넌트

그리고 `TemperatureInput` 컴포넌트는 다음과 같이 변경될 것이다.

기존에 state로 가지고 있던 온도값을 props에서 꺼내 쓰게되고, 사용자 입력을 처리하던 함수 역시 props에서 꺼내 쓰게 된다.

```js
class TemperatureInput extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange(e) {
    this.props.onTemperatureChange(e.target.value);
  }

  render() {
    const temperature = this.props.temperature;
    const scale = this.props.scale;
    return (
      <fieldset>
        <legend>Enter temperature in {scaleNames[scale]}:</legend>
        <input value={temperature} onChange={this.handleChange} />
      </fieldset>
    );
  }
}
```
