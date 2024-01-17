# 한 눈에 들어오는 Test Fixture 구성하기
Test Fixture란
: 테스트 목적, 테스트 환경을 구성하기 위해 원하는 상태 값으로 고정시킨 객체들을 말합니다.  
  
테스트 프레임워크의 생명주기에 공용 `Test Fixture`를 구성하면 안됩니다.
```Java
class Sample {
    @AfterEach
    void tearDown() {
        // 메서드가 종료될 때마다 실행
    }
    @BeforeEach
    void setUp() {
        // 매서드가 실행되기 전에 매번 실행
    }
    @BeforeAll
    static void beforeAll() {
        // 테스트 코드가 실행되기전에 제일 먼저 한번 실행
    }
    @AfterAll
    static void afterAll() {
        // 테스트 코드가 모두 종료되고 한번 실행됨
    }
}
```  

`@BeforeEach`나 `@BeforeAll`에 `Test Fixture`를 설정하여 
테스트 코드마다 중복된 `Test Fixture`를 줄일 수 있습니다.  
  
## 발생되는 문제
<procedure>
    <step>
        <p>공유 자원과 동일하게 공유 Test Fixture를 변경시 모든 테스트 코드에 영향을 줍니다.</p>
    </step>
    <step>
        <p>테스트 코드가 많아지고 길어질 경우 테스트 환경을 확인하기 위해 매번 공유 `Test Fixture`를 확인해야합니다.</p>
        <p>테스트가 파편화되어 문서로서의 역할을 하기 어렵습니다.</p>
    </step>
</procedure>  
  
## 공유 메서드는 언제 사용해야할까
공유 메서드 작성시 질문하기
: 
1. 각각의 테스트가 공유 메서드를 몰라도 테스트 코드를 이해하는데 문제가 없는가?
2. 공유 메서드를 수정해도 각각의 테스트 코드에 영향이 없는가?
  
예를 들어서 관계 매핑을 해야하는 경우 꼭 필요한데 테스트에는 상관 없는 경우 가능합니다.  
  
> 테스트 환경 구성을 하기위해 `given`절이 길어지더라도 문서로서의 역할을 할 수 있도록 합니다.  
>  
  
## data.sql 금지
1. 공유 메서드와 비슷한 `Test Fixture`의 파편화가 발생됩니다. 
2. `Test Fixture`가 매서드 내에서 관리하는게 아니기 때문에 추가 관리대상이 됩니다.
  
> data.sql을 사용하면 테스트 작성을 하는데 있어 추가 관리 대상이 발생하므로 추천하지 않습니다.  
  
## 테스트에 필요한 값만 받기
```Java
private Product createProduct(ProductType type, String productNumber, int price) {
    return Product.builder()
            .type(type)
            .productNumber(productNumber)
            .price(price)
            .sellingStatus(SELLING)
            .name("메뉴 이름")
            .build();
}
```  
테스트 클래스에 필요한 `Product`를 생성할 때 
테스트시 검증이나 비즈니스로직에 필요없는 필드는 내부로 감추고 
관련된 필드만 명시하는게 좋습니다.  
  
> 같은 `Product`를 생성하는 createProduct 메서드라도 테스트 클래스마다 
> 다른 매개변수를 받게 됩니다.  
  
## 테스트 클래스마다 빌더를 구성한다.  
생성자를 만드는 빌더를 하나의 추상 클래스에 관리하는건 복잡도가 오히려 증가합니다.  
실무에서 사용하는 필드가 수십개인 상황에서 순서만 달라져도 다른 빌더로 인식되기 때문에 
같은 인수를 받더라도 빌드 순서만 달라지면 다른 값으로 인식되기 때문에 오히려 관리하기가 어렵습니다.  
  
코틀린을 사용하면 해결이 됩니다.  

## 정리
테스트마다 별도의 문서로 활용할 수 있도록 파편화되지 않도록 합니다.