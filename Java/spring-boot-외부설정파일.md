## 개요

Spring Boot의 외부 설정파일은 기본적으로 `application.properties` 가 생성되어 있다.

이 파일 그대로 사용해도 되지만 `YAML` 을 이용하는 편이 훨씬 쉽고 간편하다.

## 비교

```properties
environments.dev.url=https://dev.example.com
environments.dev.name=Developer Setup
environments.prod.url=https://another.example.com
environments.prod.name=My Cool App
```

```yml
environments:
  dev:
    url: 'https://dev.example.com'
    name: 'Developer Setup'
  prod:
    url: 'https://another.example.com'
    name: 'My Cool App'
```

yml로 작성하는 것이 중복을 제거하고 가독성도 더 좋은 것을 알 수 있다.

## application.yml

기존의 `application.properties` 를 제거한 후, 같은 위치에 `application.yml` 이라는 파일을 생성해준다.

(기본 위치는 `/src/main/resources/`)

## yml 파일을 사용하기 위한 준비

yml 파일을 설정파일로 이용하기 위해서는 `SnakeYAML` 이라는 라이브러리가 있어야 한다.

프로젝트에 `spring-boot-starter` 의존성이 있다면 `SnakeYAML` 은 자동으로 설치된다.
