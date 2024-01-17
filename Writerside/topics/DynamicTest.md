# @DynamicTest
[2.18. Dynamic Tests](https://junit.org/junit5/docs/current/user-guide/#writing-tests-dynamic-tests)

공유 변수를 여러 테스트가 공유를 하게 되면 테스트 끼리 강결합이 생기게 되고 
테스트 순서가 중요해지는 상황이 발생하고 독립성도 보장되지 않습니다.

## 목적
DynamicTest의 목적
: 하나의 환경을 설정해놓고 사용자 시나리오를 테스트  
+ 환경을 설정해놓고 이 환경에 **변화**를 주면서 **중간 중간 검증**을 하고
이런 행위를 했을때 이런 검증을 하고 일련의 시나리오를 테스트 하고 싶을때가 있는데
그럴때 사용하기 좋은게 이 어노테이션입니다.

## 주의사항
> Dynamic Test Lifecycle:  
> @BeforeEach 및 @AfterEach 메서드 및 해당하는 확장 콜백은 @TestFactory 메서드 자체에 대해서만 실행되고, 개별 동적 테스트에 대해서는 실행되지 않습니다.
> 
{ style="warning"}

## 문법
```Java
@TestFactory
Collection<DynamicTest> dynamicTestsFromCollection() {
    return Arrays.asList(
        dynamicTest("1st dynamic test", () -> assertTrue(isPalindrome("madam"))),
        dynamicTest("2nd dynamic test", () -> assertEquals(4, calculator.multiply(2, 2)))
    );
}
```

## 예제
```java
@DisplayName("재고 차감 시나리오")
@TestFactory
Collection<DynamicTest> stockDeductionDynamicTest(){
    //given
    Stock stock = Stock.create("001",1);

    return List.of(
        DynamicTest.dynamicTest("재고를 주어진 개수만큼 차감할 수 있다.",()->{
            //given
            int quantity = 1;

            //when
            stock.deductQuantity(quantity);

            //then
            assertThat(stock.getQuantity()).isZero();
        }),
        DynamicTest.dynamicTest("재고보다 많은 수의 수량으로 차감 시도하는 경우 예외가 발생한다.",()->{
            //given
            int quantity = 1;

            //when //then
            assertThatThrownBy(()->stock.deductQuantity(quantity))
                    .isInstanceOf(IllegalArgumentException.class)
                    .hasMessage("차감할 재고 수량이 없습니다.");
        })
    );
}
```

