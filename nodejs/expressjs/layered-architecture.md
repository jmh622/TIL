# 폴더 구조

다음은 해외 블로그에서 가져온 프로젝트 구조이다.

```
src
│   app.js          # App entry point
└───api             # Express route controllers for all the endpoints of the app
└───config          # Environment variables and configuration related stuff
└───jobs            # Jobs definitions for agenda.js
└───loaders         # Split the startup process into modules
└───models          # Database models
└───services        # All the business logic is here
└───subscribers     # Event handlers for async task
└───types           # Type declaration files (d.ts) for Typescript
```

```
my-project/
├── node_modules/
├── config/
│   ├── utils.js
├── components/
├── modules/
│   │   ├── profile/
│   │   │   ├── routesProfile.js
│   │   │   ├── serviceProfile.js
│   │   │   ├── dalProfile.js
│   │   │   ├── index.js
│   │   ├── notification/
│   │   │   ├── routesNotification.js
│   │   │   ├── serviceNotification.js
│   │   │   ├── dalNotification.js
│   │   │   ├── index.js
│   │   ├── alerts/
│   │   │   ├── routesAlert.js
│   │   │   ├── serviceAlert.js
│   │   │   ├── dalAlert.js
│   │   │   ├── index.js
├── pages/
│   ├── profile.js
│   ├── index.js
├── public/
│   ├── styles.css
├── app.js
├── routes.js
├── package.json
├── package-lock.json
└── README.md
```

여러 폴더가 있지만 처음에는 다음 네 가지 정도만 있어도 될 것 같다.

- api
  - controller 역할을 하는 레이어인 것 같다.
  - 나에게는 controller 라는 이름이 더 익숙하니 나는 controller 로 사용하겠다.
- services
  - 비즈니스 로직을 담당하는 레이어
- models
  - 엔터티를 관리하는 레이어
- dal (data access layer)
  - database에 접근하는 레이어

결국, 레이어 명칭이나 구조는 언어마다, 프레임워크마다, 사람마다 다 달라도 역할의 구분은 대동소이하다.

그래서 최종적으로 나는 nodejs 토이 프로젝트를 진행할 때 다음과 같은 구조를 구성할 것이다.

```
project
├── controller
├── service
├── repository
├── model
```

이 구조는 `spring web mvc` 에서 사용하는 `layered architecture` 이다.

레이어 명칭은 회사마다 다를 수도 있기도 하기 때문에 이름은 기존에 사용하던, 익숙한 이름을 사용하기로 했다.

# controller

컨트롤러에는 비즈니스 로직이 들어가면 안된다.

컨트롤러에 비즈니스 로직이 들어가게 되는 순간 `layered architecture` 의 의미는 사라지게 되고 유지보수도 하기 힘들며 가독성도 떨어진다.

따라서 컨트롤러는 오직 라우팅, 서비스 레이어에 요청을 보내고 결과를 반환하는 역할만 수행하도록 하자.

# service

비즈니스 로직을 담당하는 레이어.

단순 데이터 조회 및 반환만 하는 controller의 경우에는 이 service 레이어가 필요하지 않을 수도 있다.

이 레이어에 request, response와 같은 객체를 전달하지 않도록 해야 한다. 마찬가지로 response와 관련된 어떠한 정보(header, status code 등)도 반환하지 말아야 한다.

DB에 직접적으로 접근하지 않도록 해야 한다.

# repository

repository는 어떤 DB를 사용하느냐, ORM을 사용하느냐 등 여러 조건에 따라 코드가 달라질 수 있다.

(추후 정리)

# models

model 역시 repository와 마찬가지로 어떤 DB를 사용하느냐, ORM을 사용하느냐 등 여러 조건에 따라 코드가 달라질 수 있다.

(추후 정리)

# SOLID 원칙 중 OCP, DIP 적용하기

# References

- [Layered Architecture for NodeJs](https://ctrly.blog/nodejs-layered-architecture/)
- [Bulletproof node.js project architecture](https://softwareontheroad.com/ideal-nodejs-project-structure/)
