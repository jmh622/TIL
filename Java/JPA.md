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
