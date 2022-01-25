# 템플릿 엔진 적용하기

`nodejs` 에서 사용할 수 있는 템플릿 엔진의 종류는 굉장히 다양하다.

그 중에서 npm 기준 다운로드 수도 높고 github stars 도 많으며 문법이 어렵지 않은 것으로 택했다.

그것은 바로 `handlebars`

기본적으로 html 구조에 어렵지 않은 `handlebars` 문법으로 어렵지 않게 쓸 수 있을 것 같다.

## 설치

```bash
npm i express-handlebars
```

## 뷰 파일 경로

```
project
├── views
│   ├── home.handlebars
│   ├── layouts
│   │   ├── main.handlebars
├── app.js
```

## app.js 에 적용하기

```javascript
// in app.js
import express from 'express';
import { engine } from 'express-handlebars';

const app = express();

// handlebars 설정
app.engine('handlebars', engine());
app.set('view engine', 'handlebars');

app.get('/', (req, res) => {
  // home.handlebars 를 찾아 html 을 반환
  res.render('home');
}

app.listen(8080);
```

`http://localhost:8080` 으로 접속하게 되면 `res.render()` 에 의해 view 파일을 찾게 된다. 여기서는 `home.handlebars` 를 찾게 될 것이다.

## handlebars 확장자 변경하기

여기서 굉장히 불편한 것이 하나 있다.

바로 확장자!

확장자가 너무 길기 때문에 파일을 생성할 때 여간 귀찮은 것이 아니다.

이는 설정으로 바꿔줄 수 있다.

```javascript
// handlebars 설정
app.engine('hbs', engine());
app.set('view engine', 'hbs');

app.get('/', (req, res) => {
  // home.hbs 를 찾아 html 을 반환
  res.render('home');
}
```

이렇게 바꿔주면 `home.hbs` 파일을 찾게 된다.

하지만! layout 파일은 기본설정에 의해 확장자마저 고정되어 있다.

> /views/layouts/main.handlebars

하지만 이 역시도 설정으로 바꿔줄 수가 있다.

```javascript
// handlebars 설정
app.engine('hbs', engine({ extname: '.hbs' }));
app.set('view engine', 'hbs');
```

이렇게 `engine()` 의 파라미터로 확장자를 설정해줄 수가 있게 된다.

이렇게 되면 layout 파일마저도 확장자를 `hbs` 로 설정할 수 있게 된다.
