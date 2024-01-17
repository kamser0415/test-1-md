# Test Fixture 클렌징  
Order Entity

오더 엔티티의 관계 매핑을 보면

다대 다 연관 관계의 중간을 풀어주는 OrderProduct로 매핑을 했는데

Order ↔ OrderProduct → Product

OrderProduct 는 FK를 둘다 가지고 있는 상태다.

Product는 Order와 OrderProduct의 존재를 모르고,몰라야하기때문에 단방향 설계로 되어있다.

OrderProductRepository를 별도로 지우는 작업을 했다.

얘를 지우지 않으면 다른 테스트에 영향을 주니까 지워줬다.

```java
@AfterEach
void tearDown() {
    **orderProductRepository**.deleteAllInBatch();
    productRepository.deleteAllInBatch();
    orderRepository.deleteAllInBatch();
    stockRepository.deleteAllInBatch();
}
Hibernate: 
    delete 
    from
        **order_product**
Hibernate: 
    delete 
    from
        product
Hibernate: 
    delete 
    from
        orders
Hibernate: 
    delete 
    from
        stock
```

> 순서를 변경해서 productRepository를 먼저 삭제한다.
>

```java
@AfterEach
void tearDown() {
    productRepository.deleteAllInBatch();
    **orderProductRepository**.deleteAllInBatch();
    orderRepository.deleteAllInBatch();
    stockRepository.deleteAllInBatch();
}
FKHNFGQYJX3I80QOYMRSSLS3KNO 예외발생
```

Product 의 key (id)가 OrderProduct FK로 참조가 되고있다.

그래서 지을수 없다는 예외가 발생했다.

FK를 가지고 있는 테이블을 먼저 지워야 한다.

deleteAllInBatch()
: 테이블 전체를 지우는 메서드
외래키 등 조건이 순서에 따라 예외가 발생할수 있어서
순서에 대한 고민을 해야한다.

> deleteAllInBatch()
>

```java
// 해당 레포지토리
delete * from table_0
```

> deleteAll()
>

```java
// 해당 리포지토리를 조회
Hibernate: 
    select
        o1_0.id,
        o1_0.create_date_time,
        o1_0.modified_data_time,
        o1_0.order_status,
        o1_0.registered_date_time,
        o1_0.total_price 
    from
        orders o1_0
Hibernate: 
    select
        o1_0.order_id,
        o1_0.id,
        o1_0.create_date_time,
        o1_0.modified_data_time,
        o1_0.product_id 
    from
        order_product o1_0 
    where
        o1_0.order_id=?
Hibernate: 
    delete 
    from
        order_product 
    where
        id=?
Hibernate: 
    delete 
    from
        order_product 
    where
        id=?
Hibernate: 
    delete 
    from
        orders 
    where
        id=?
```

1. 삭제하려는 리포지토리를 조회
2. FK로 연결된 테이블을 찾아서 한번 더 조회 (order ↔ orderProduct)
3. orderProduct를 하나씩 지운다.
4. order를 하나씩 지운다.

장점
: OrderProductRepository를 따로 삭제를 하지 않아도 삭제가 된다.
그래도 순서가 중요해서 Product는 orderProduct의 존재를 모르기때문에
order를 먼저 지워야 FK 예외가 발생하지 않는다.

> 강사
> 대부분 @Transactional로 롤백을 처리하면 위와 같은 데이터 삭제에 대한 순서를 알 필요가 없지만,
사이드 이펙트로 발생하는 실제 객체에 @Transactional의 확인 여부가 필요하고
스프링 배치를 사용하게 된다면 여러 트랜잭션이 섞이기 때문에 사용하기가 어렵다.
FK가 어디에 저장되어있는지 연관관계와 삭제 순서를 신경쓴다면
불필요한 쿼리가 날라가지않는 .deleteAllInBatch()가 쿼리가 깔끔하게 작성되고
FK와 연관된 Entity에 대해서 찾아서 지워주는 방식인 deleteAll()이 있다
필요한 상황에 맞게 사용하는게 좋다.