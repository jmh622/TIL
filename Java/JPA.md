## Update

- `@Transactional` 이 있으면 Entity의 필드가 변경되면 JPA가 알아서 쿼리를 날려준다
- 업데이트는 변경감지를 이용하자 (merge는 위험)

```java
@Transactional
void update(Item itemParam) { //itemParam: 파리미터로 넘어온 준영속 상태의 엔티티
    Item findItem = em.find(Item.class, itemParam.getId()); //같은 엔티티를 조회한다.
    findItem.setPrice(itemParam.getPrice()); //데이터를 수정한다.
}
```

## 다대일, 일대일 등에서의 지연 로딩

!! 중요

아래 예시는 요청, 응답 시 엔터티를 사용하지 않으면 해결되는 문제이다.

(개발 시 여러 상황이 발생할 수 있으니 참고만 하도록 하자)

> 예시 클래스

```java

@Entity
public class Member {
    @Id
    private Long id;

    @OneToMany
    private List<Order> orders = new ArrayList<>();
}

@Entity
public class Order {
    @Id
    private Long id;

    @ManyToOne(fetch = LAZY)
    @JoinColumn
    private Member member;
}

```

### List<Order> all = orderRepository.findAll(); 을 실행한 경우

**첫 번째 문제**

양방향으로 두 엔터티가 연결이 되어 있기 때문에 하나의 Order를 가져오면 Member를 가져오고 다시 List<Order>를 가져오고... 무한반복에 빠지게 된다.

> 해결 방법

> 둘 중 한 곳에 `@JsonIgnore` 를 추가하여 연결을 끊어줘야 한다.

**두 번째 문제**

`fetch = LAZY` 를 이용할 경우 해당 객체는 실제 객체가 아닌 proxy 객체가 들어가게 되는데, 이때 Jackson 라이브러리가 이 proxy 객체를 처리하지 못하기 때문에 익셉션이 발생하게 된다.

> 해결 방법

> `build.gradle` 에 `implementation 'com.fasterxml.jackson.datatype:jackson-datatype-hibernate5'` 추가
>
> `Hibernate5Module` 를 스프링 빈으로 등록하여 에러 해결

```java
Hibernate5Module hibernate5Module() {
    Hibernate5Module hibernate5Module = new Hibernate5Module();
    return hibernate5Module;
}

@GetMapping("/sample")
public List<Order> orders() {
    List<Order> all = orderRepository.findAll();
    for (Order order : all) {
        order.getMember().getName(); //Lazy 강제 초기화
        order.getDelivery().getAddress(); //Lazy 강제 초기화
    }
    return all;
}
```

> 위 두 가지 문제를 해결하는 더 좋은 방법 : join fetch
>
> jpql의 join fetch 를 이용하는 것이 훨씬 좋다.

```java
// Repository
public List<Order> findAllWithMemberDelivery() {
    return em.createQuery(
        "select o from Order o" +
        " join fetch o.member m" +
        " join fetch o.delivery d", Order.class)
    .getResultList();
}

// Controller
@GetMapping("/sample")
public List<SimpleOrderDto> orders() {
    List<Order> orders = orderRepository.findAllWithMemberDelivery();
    List<SimpleOrderDto> result = orders.stream()
        .map(o -> new SimpleOrderDto(o))
        .collect(toList());
    return result;
}

@Data
static class SimpleOrderDto {
    private Long orderId;
    private String name;
    private LocalDateTime orderDate; //주문시간
    private OrderStatus orderStatus;
    private Address address;

    public SimpleOrderDto(Order order) {
        orderId = order.getId();
        name = order.getMember().getName();
        orderDate = order.getOrderDate();
        orderStatus = order.getStatus();
        address = order.getDelivery().getAddress();
    }
}

```

## 일대다 컬렉션 조회 최적화

### join fetch + distinct(마스터 객체 중복 제거)

- `Order` - `OrderItem` 관계가 일대다
- join fetch 사용시 일대다 관계때문에 `Order` 가 중복으로 발생한다
- jpql의 distinct를 이용해서 `Order` 의 중복을 제거해준다
- 단점
  - sql의 결과가 `OrderItem` 을 기준으로 나오기 때문에 `Order` 에 대해서 페이징 처리 불가
  - 컬렉션에 대한 join fetch는 딱 하나에 대해서만 사용해야 한다

```java
public List<Order> findAllWithItem() {
    return em.createQuery(
        "select distinct o from Order o" +
                " join fetch o.member m" +
                " join fetch o.delivery d" +
                " join fetch o.orderItems oi" +
                " join fetch oi.item i", Order.class)
        .getResultList();
}
```

### hibernate.default_batch_fetch_size (또는 @BatchSize) 이용하여 쿼리 개수 최적화

- 해당 설정을 하게 되면 쿼리가 조건절 IN 을 이용하여 작성된다.
- 따라서 쿼리의 개수가 적어져 성능 최적화

```yml
# application.yml
spring:
  jpa:
  properties:
  hibernate:
  default_batch_fetch_size: 1000
```

```java
public List<Order> findAllWithMemberDelivery(int offset, int limit) {
    return em.createQuery(
        "select o from Order o" +
                " join fetch o.member m" +
                " join fetch o.delivery d", Order.class)
        .setFirstResult(offset)
        .setMaxResults(limit)
        .getResultList();
}
```
