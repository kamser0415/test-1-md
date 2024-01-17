# PersistenceLayerTest  

## 단위 테스트 성격
리포지토리는 외부 서버를 이용해서 테스트를 하지만 통합테스트보다 **단위 테스트 성격이 강합니다.**   

리포지토리 테스트는 단위 테스트 성격에 가까운 이유  
: 리포리토리에 대한 테스트를 하기 위해 서버를 띄우지만 레이어별로 끊어서 볼땐
Persistence 계층의 역할은 데이터베이스에 액세스를 하는 것입니다.
데이터베이스에 엑세스하는 로직만 가지고 있기 때문에 약간의 단위 테스트 성격을 갖습니다.  

> 리포지토리 테스트는 CRUD 테스트를 하는 목적입니다.
> 비즈니스 로직이 섞여있으면 안됩니다.
{ style="warning"}

## 리포지토리 테스트
리포지토리를 테스트하는 이유
: + 조건이 복잡하거나 길어질 경우
+ 리포지토리 메서드의 인수를 잘못 줄 수도 있다
+ raw JPQL,QueryDSL,MyBatis등으로 구현 기술이 변경될 테스트 코드의 결과는 보장될 수 있기 때문이다.  
  
JPA,MyBatis,JPQL,JDBC
: 간단한 `Native SQL`이나 `JPQL`을 사용해도 테스트 코드는 작성합니다. 
`SQL`에 대한 검증이 필요합니다.
  
<procedure title="리포지토리 테스트 목적">
<step>
<p>작성한 코드가 제대로 된 Query가 제대로 실행되는지 보장을 받기 위해서</p>
</step>
<step>
<p>이후에 어떤 형태로 변경될지 모른다. 그 결과를 항상 보장 받기 위해서 테스트를 작성한다.</p>
</step>
</procedure>   

  
> id 필드는 데이터베이스 정책으로 초기화시 `id`를 포함하지 않아도 데이터베이스에서 넣어준다. 
> 

### 리스트 반환형 테스트 방식
```Java
@DisplayName("원하는 판매상태를 가진 상품들을 조회한다.")
@Test
void findAllBySellingStatusIn(){
    //given
    Product product1 = Product.builder()
            .productNumber("001")
            .type(HANDMADE)
            .name("아메리카노")
            .price(4000)
            .sellingStatus(SELLING_TYPE)
            .build();
    Product product2 = Product.builder()
            .productNumber("002")
            .type(HANDMADE)
            .name("라떼")
            .price(4500)
            .sellingStatus(HOLD)
            .build();
    Product product3 = Product.builder()
            .productNumber("003")
            .type(HANDMADE)
            .name("팥빙수")
            .price(8000)
            .sellingStatus(STOP_SELLING)
            .build();
    //테스트를 만들기 위한 given
    productRepository.saveAll(List.of(product1, product2, product3));

    //when
    List<Product> products = productRepository.findAllBySellingStatusIn(List.of(SELLING_TYPE, HOLD));

    //then
    Assertions.assertThat(products).hasSize(2)
            .extracting("productNumber", "name", "sellingStatus")
            .containsExactlyInAnyOrder(
                    Assertions.tuple("001", "아메리카노", SELLING_TYPE),
                    Assertions.tuple("002", "라떼", HOLD)
            );
}
```  

List 반환형 검증 순서
: + `hasSize`로 저장된 사이즈를 찾는다
+ `extracting` 검증하고자하는 필드만 별도로 추출가능
+ 추출한 데이터를 검증한다
+ 순서 검증 포함 - `containsExactly`  
+ 순서 검증 미포함 -`containsExactlyInAnyOrder`

## @DataJpaTest vs @SpringBootTest

@DataJpaTest  
: + `JPA`에 관련된 빈만 등록한다.
+ 속도가 `@SpringBoottTest`보다 빠르다.
+ 기본은 내장 메모리로 돌아간다.
  
@SpringBootTest  
: + 스프링 부트 구성정보에 포함되는 모든 클래스가 등록된다.  

