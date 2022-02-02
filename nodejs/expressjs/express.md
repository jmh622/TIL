> express 정리

# message body의 json 데이터 받는 방법

미들웨어 함수: `express.json()`

해당 미들웨어 함수는 express v4.16.0 이상에서 사용이 가능하다. (이전 버전은 `body-parser`를 이용해야 할 것이다. `maybe?`)

(대부분의 경우는) `request`의 헤더가 `Content-Type: application/json` 이고 `json` 데이터만을 변환한다.

변환된 데이터는 `request` 객체의 `body` 프로퍼티에 채워진다. 변환할 데이터가 없거나 `Content-Type`이 일치하지 않으면 빈 객체를 반환하거나 오류가 발생한다.

_example_

```javascript
const express = require('express');

const app = express();
app.use(express.json());

app.post('/users', async (req, res) => {
  const { name, age } = req.body;

  if (!name || !age) {
    // req.body 로부터 받은 데이터 유효성 검사
  }

  res.json({ id: await userService.save(name, age) });
});
```

# form 태그로부터 넘어온 데이터 받는 방법

미들웨어 함수: `express.urlencoded()`

해당 미들웨어 함수는 express v4.16.0 이상에서 사용이 가능하다.

form 태그로부터 발생한 요청의 `Content-Type`은 `application/x-www-form-urlencoded` 이고 이 요청의 데이터를 받아올 때는 해당 미들웨어를 사용해야 한다.

변환된 데이터는 `request` 객체의 `body` 프로퍼티에 채워진다. 변환할 데이터가 없거나 `Content-Type`이 일치하지 않으면 빈 객체를 반환하거나 오류가 발생한다.

_example_

```javascript
const express = require('express');

const app = express();
app.use(express.urlencoded());

router.post('/', async (req, res) => {
  const { username, password } = req.body;

  if (!username || !password) {
    // req.body 로부터 받은 데이터 유효성 검사
  }

  const user = await userService.findByUsername(username);

  if (!user) {
    return res.sendStatus(404);
  }

  req.session.user = user;

  res.redirect('/');
});
```

# 세션 이용하기

미들웨어: `express-session`

_설치_

```bash
npm i express-session
```

_example_

```javascript
const express = require('express');
const session = require('express-session');

const app = express();
app.use(
  session({
    secret: 'studying express',
  })
);
```

express앱에서 세션을 이용할 수 있게 해주는 미들웨어다.

기본 저장소는 메모리이다. 따라서 운영환경에서는 별도의 세션서버나 db저장소를 이용해야 한다.

현재(2022.02.02) github stars 기준 1등은 레디스(`connnect-redis`), 2등은 몽고DB(`connect-mongo`)이다.

## 세션 옵션

- cookie
- genid
- name
- proxy
- resave
- rolling
- saveUninitialized
- secret
- store
- unset

### cookie

_기본값_

`{ path: '/', httpOnly: true, secure: false, maxAge: null }`

- domain
  - `Domain` 특성에 대응
  - 기본값은 설정되어 있지 않으며, 이 경우 대부분의 클라이언트는 현재 도메인에만 적용되는 것으로 간주
- expires
  - `Expires` 특성에 대응. Date형
  - 기본값은 설정되어 있지 않으며, 이 경우 비영구 쿠키로 간주하고 세션쿠키로 간주
  - `maxAge` 와 함께 설정되어 있으면 마지막 항목으로 설정됨
  - 공식 문서에서는 `maxAge`를 이용하라고 권고
- httpOnly
  - `HttpOnly` 특성에 대응. boolean형. 기본값은 `true`.
- maxAge
  - `Expires` 특성에 대응. number형 (단위는 밀리초)
  -
- path
  - `Path` 특성에 대응.
  - 기본값은 `/`
- sameSite
  - `SameSite` 특성에 대응. boolean형 또는 string형.
  - `true`, `false`
  - `lax`, `none`, `strict`
- secure
  - `Secure` 특성에 대응. boolean형.
  - 기본값은 설정되어 있지 않다.
