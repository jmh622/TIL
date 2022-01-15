# Spring WebFlux

> Spring WebFlux는 리액티브 스트림(Reactive Streams)을 구현한 Reactor를 이용한 Non-blocking 웹앱 프레임워크이다.
>
> 리액티브 스트림을 구현한 구현체는 다양하지만 표준을 잘 지켜 만들었기 때문에 서로 호환이 가능하다.

## 함수형 프로그래밍

> 함수형 프로그래밍(函數型 프로그래밍, 영어: functional programming)은 자료 처리를 수학적 함수의 계산으로 취급하고 상태와 가변 데이터를 멀리하는 프로그래밍 패러다임의 하나이다. 명령형 프로그래밍에서는 상태를 바꾸는 것을 강조하는 것과는 달리, 함수형 프로그래밍은 함수의 응용을 강조한다. 프로그래밍이 문이 아닌 식이나 선언으로 수행되는 선언형 프로그래밍 패러다임을 따르고 있다. (위키백과)
>
> 순수 함수(Pure Function)으로만 작성되어 지며 모든 입력은 입력으로만, 모든 출력은 추력으로만 사용된다.

### 함수형 인터페이스

> `@FunctionalInterface` 어노테이션을 이용하여 인터페이스를 함수형 인터페이스로 만들 수 있다

### 함수형 인터페이스 종류

- Fuction : T -> R
  - 인자 O | 리턴값 O
  - T 형식의 객체를 R 형식의 객체로 변환할 때 사용
- Consumer : T -> void
  - 인자 O | 리턴값 X
- Supplier : () -> T
  - 인자 X | 리턴값 O
- Operator : T -> T
  - 인자 O | 리턴값 O
- Predicate : T -> boolean
  - 인자 O | 리턴값 O
  - T 형식의 객체로 test를 진행할 때 사용
- 다양한 인터페이스 제공
  - DoubleConsumer, IntUnaryOperator, BiPredicate, IntPredicate 등등

## 스트림 (Stream)

**스트림의 특징**

- 여러 연산을 파이프라인으로 연결하여 쉽게 처리할 수 있도록 도와준다
- 소스로부터 데이터를 읽기만 할 뿐, 소스 자체를 변경하지 않는다
- 일회용이다. 따라서 추가적으로 스트림이 필요하다면 다시 생성해서 사용해야 한다

**스트림의 메서드**

- filter
- limit
- map
- skip
- flatMap
- anyMatch
- allMatch
- noneMatch
- findAny
- reduce
- 등등

## 리액티브 스트림

> Flow API가 JAVA 9 버전에 추가되었으니 JAVA 11 버전을 추천

### 4개의 인터페이스 제공

- Publisher
  - 데이터를 통지하는 생산자
  - 메서드
    - Subscriber의 구독을 받기 위한 subscribe만 존재
- Subscriber
  - 데이터를 받아 처리하는 소비자
  - 메서드
    - onSubscribe : 데이터를 최초 통지할 때 호출
    - onNext : 데이터를 통지할 때 마다 호출
    - onError : 데이터 통지에서 에러가 발생했을 때 호출
    - onComplete : 데이터 통지가 끝났을 때 호출
- Subscription
  - 전달 받을 데이터의 개수를 요청하고 구독을 해지
  - 메서드
    - request : 데이터 요청 개수 설정
    - cancel : 구독 해지
- Processor
  - Publisher와 Subscriber의 모든 기능을 가지고 있음

### 백프레셔 (Back Pressure)

> 데이터 통지의 속도가 Subscriber에서 데이터를 처리하는 속도보다 빠를 때 데이터 통지량을 제어하는 기능
>
> 구독방식은 Publisher가 Subscriber에게 직접 데이터를 전달하는 것이 아니라, Subcriber가 데이터를 pull 받는 방식이다. 따라서 Subscriber가 처리할 수 있는 만큼의 데이터 양을 요청하므로써 장애를 방지 시킨다.
>
> 배압이라고도 함

### Back Pressure Strategy

- Buffer
  - 통지할 수 있을 때까지 데이터를 버퍼링한다
- Drop
  - 통지불가할 때의 데이터는 삭제한다
- Latest
  - 최신 데이터만 버퍼링한다
- Error
  - MissingBackpressureException 을 일으킨다
- None
  - 아무 처리를 하지 않는다

### Hot & Cold

**Cold**

> 구독을 할 때마다 매번 독립적인 데이터가 생성된다.
>
> 구독할 때마다 데이터를 통지하는 다른 타임라인이 생성된다.

**Hot**

> 구독자에 상관없이 데이터 통지 타임 라인이 하나이다.
>
> 구독자는 구독 시점에 통지되고 있는 데이터부터 받을 수 있다.

### Flowable vs Observable

- Flowable
  - Reactive Streams 인터페이스를 구현
  - 백프레셔 기능 있음
  - pull 기반
- Observable
  - Reactive Streams 인터페이스를 구현한 구현체가 아님
  - 백프레셔 기능 없음
  - push 기반

### 리액티브 스트림 메서드

- map
- flatMap
- concatMap
- startWith
- concat
- zip
- isEmpty
- contains
- all
- count
- reduce
- scan
- 등등

## 기타

### 메서드 레퍼런스

`클래스명::메서드명` 표현식

- 종류
  - 클래스명::정적메서드명
  - 객체변수::메서드명
  - 클래스명::메서드명
  - 클래스명::new

### References

- [라인 기술블로그 - Armeria로 Reactive Streams와 놀자! – 1](https://engineering.linecorp.com/ko/blog/reactive-streams-with-armeria-1/)
- [에디의 기술블로그](https://brunch.co.kr/@springboot/154)
