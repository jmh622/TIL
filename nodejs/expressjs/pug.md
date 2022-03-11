# PUG

## 설치

```bash
npm i pug
```

## app 설정

```js
// src/server.js
import express from 'express';
const app = express();

app.set('view engine', 'pug');
// app의 main 파일이 루트경로가 아닌 src 폴더 내에 있기 때문에
// views 설정 변경
app.set('views', process.cwd() + '/src/views');
```

## javascript 사용

`#{}`을 이용하여 pug 안에서 자바스크립트를 사용할 수 있다.

```pug
fotter &copy; #{new Date().getFullYear()} Wetube
```

## base page

layout을 담당하는 페이지

다른 페이지에서 `extends` 명령어로 레이아웃을 이용할 수 있다.

```pug
//- base.pug
doctype html
html(lang='ko')
    head
        title #{pageTitle} | Wetube
    body
        block content
        include partials/footer.pug


//- home.pug
extends base.pug

block content
    h1 Home!
```

## partial page

다른 페이지에서 `include` 명령어로 partial 페이지를 넣을 수 있다.

```pug
//- footer.pug
fotter &copy; #{new Date().getFullYear()} Wetube

//- base.pug
doctype html
html(lang='ko')
    head
        title #{pageTitle} | Wetube
    body
        block content
        include partials/footer.pug
```

## variables

자바스크립트 변수를 pug 내에서 사용할 수가 있다.

- `#{}`를 이용하는 방법
- `tag=var`를 이용하는 방법
  - 이 방법은 해당 태그에 변수 하나만 할당하는 경우에 사용한다.

```pug
//- 1. #{} 이용
doctype html
html(lang='ko')
    head
        title #{pageTitle} | Wetube

//- 2. = 이용
h1=pageTitle
```

pug 내에서 자바스크립트 표현식을 사용할 수 있었는데, 내가 추가한 임의의 변수에 값을 어떻게 넣을까?

바로 controller에서 넣어줄 수 있다.

```pug
//- base.pug
doctype html
html(lang='ko')
    head
        title #{pageTitle} | Wetube
    body
        block content
        include partials/footer.pug
```

```js
// controller
export const home = (req, res) => res.render('home', { pageTitle: 'Home' });
```

controller에서 `pageTitle`이라는 변수에 `Home`이라는 값을 할당해주었기 때문에, `title`이 `Home | Wetbue`가 된다.

## condition

javascript와 동일하게 if, else를 사용하면 된다.

```pug
if fakeUser.loggedIn
    li
        a(href='/logout') Log out
else
    li
        a(href='/login') Login
```

## loop

`each` 명령어를 이용하여 반복문을 사용할 수 있다.

또한 array가 비어있는 경우 `else` 명령어를 이용하여 처리할 수 있다.

```pug
ul
    each video in videos
        li=video
    else
        li Sorry, nothing is found
```

## mixins

mixins는 html을 재사용할 수 있도록 하는 일종의 partial이다.

`include`로 mixins파일을 추가해주고 `+`와 해당 mixins에서 설정한 name을 이용하여 추가해줄 수 있다.

```pug
//- /mixins/video.pug
mixin video(data)
    div
        h4=data.title
        ul
            li #{data.rating} / 5.
            li #{data.comments} comments.
            li Posted #{data.createdAt}.
            li #{data.vies} views.

//- home.pug
extends base.pug
include mixins/video

block content
    h2 Welcome here you will see the trending videos
    each video in videos
        +video(video)
    else
        div Sorry, nothing is found

```
