# @ParameterizedTest

테스트 코드내에 if/for/분기문등이 읽는 사람이 코드를 생각하게 하는 코드를 지양 해야된다.

**여러가지의 케이스 코드**인데 하나의 테스트 코드에 녹여내고 싶다는 욕망이 있다보니 테스트에 분기, 반복문이 들어가게 됩니다.
여러가지 케이스 테스크 코드라서 나누는게 바람직한지 체크하는게 필요하다.  
1. 해피케이스
2. 예외케이스
3. 경계값


케이스 코드가 하나이지만 단순하게 값만 변경해서 테스트 코드를 작성할 때 사용합니다.
값이나 환경을 변경해가면서 테스트를 여러번 반복할수 있다.
[JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests)
## 기존 방식
```java
@DisplayName("상품 타입이 재고 관련 타입인지를 체크한다.")
@Test
void containsStockType3(){
    //given
    ProductType giveType1 = ProductType.HANDMADE;
    ProductType giveType2 = ProductType.BOTTLE;
    ProductType giveType3 = ProductType.BAKERY;

    //when
    boolean result1 = ProductType.containsStockType(giveType1);
    boolean result2 = ProductType.containsStockType(giveType2);
    boolean result3 = ProductType.containsStockType(giveType3);

    //then
    assertThat(result1).isFalse();
    assertThat(result2).isTrue();
    assertThat(result3).isTrue();
}
```
단일 케이스로 `when`에 대한 메소드가 하나이고 값에 따라 `true`,`false`인지 
`ProductType` 타입에 대해서 확인하는 테스트 코드입니다.  

> 테스트 케이스는 하나이지만 오히려 케이스가 많아보이고 어떤 결과가 발생하는지 알기 어렵습니다.  

## CsvSource 사용  
> 인수가 적을때 사용하기 좋다.
```java
@DisplayName("상품 타입이 재고 관련 타입인지를 체크한다.")
@CsvSource({"HANDMADE,false","BOTTLE,true","BAKERY,true"})
@ParameterizedTest
void containsStockType4(ProductType productType,boolean expected){
    //when
    boolean result = ProductType.containsStockType(productType);
    //then
    assertThat(result).isEqualTo(expected);
}
```
`@ScvSource`의 값이 테스트 코드 매서드의 파라미터로 차례대로 들어옵니다.
지정한 `ProductType`을 받아서 when을 실행하고 검증 결과로 `expected`로 비교 할 수 있습니다.

## MethodSource 사용
> 인수가 많을 경우 사용한다.
```java
private static Stream<Arguments> provideProductTypesForCheckingStockType(){
    return Stream.of(
            Arguments.of(ProductType.HANDMADE,false),
            Arguments.of(ProductType.BOTTLE,false),
            Arguments.of(ProductType.BAKERY,false)
    );
}
@DisplayName("상품 타입이 재고 관련 타입인지를 체크한다.")
@MethodSource("provideProductTypesForCheckingStockType")
@ParameterizedTest
void containsStockType5(ProductType productType,boolean expected){
    //when
    boolean result = ProductType.containsStockType(productType);

    //then
    assertThat(result).isEqualTo(expected);
}
```  
@MethodSource
: 팩토리 매서드,정규화된 매서드 이름을 통해 테스트 코드에 전달
1. 스트림 다양한 유형 사용 가능 : `Stream`, `DoubleStream` 등
2. Arguments의 하나의 세트당 개별 물리적 인수로 처리합니다.  
   즉, 배열의 각 항목이 테스트 메서드의 각 매개변수에 대응됩니다.

보통 프로덕션 코드에서는 `private method`는 맨 아래에 관리를 하지만, 
테스트 코드에서는 `given`의 역할이기 때문에 테스트 코드 위에서 관리합니다.
  
위 2개 방식을 주로 사용합니다.  

## Customizing Display Names
`@CsvSource` 사용할 경우
```Java
@DisplayName("상품 타입이 재고 관련된 타입인지 체크한다.")
@ParameterizedTest(name = "{0} ==> the rank of ''{0}'' is {1}")
@CsvSource({"HANDMADE,false","BOTTLE,true","BAKERY,true"})
void testWithCustomDisplayNames(ProductType productType,Boolean expected) {
   //when
   boolean result = ProductType.containsStockType(productType);
   //then
   assertThat(result).isEqualTo(expected);
}
```
`@MethodSource` 사용할 경우
```Java
static Stream<Arguments> customDisplayNamesSource() {
   return Stream.of(
          Arguments.of(ProductType.HANDMADE, false),
          Arguments.of(ProductType.BOTTLE, true),
          Arguments.of(ProductType.BAKERY, true)
   );
}
@DisplayName("상품 타입이 재고 관련된 타입인지 체크한다.")
@ParameterizedTest(name = "{index} ==> {0} ==> the rank of ''{0}'' is {1}")
@MethodSource("customDisplayNamesSource")
void testWithCustomDisplayNames2(ProductType productType, Boolean expected) {
   //when
   boolean result = ProductType.containsStockType(productType);
   //then
   assertThat(result).isEqualTo(expected);
}
```
![image_9.png](image_9.png)  

## Spock
Spock 장점
: Junit보다 `ParameterizedTest`를 더 명시적으로 작성할 수 있습니다.
• [Spock Data Tables](https://spockframework.org/spock/docs/2.3/data_driven_testing.html#data-tables)