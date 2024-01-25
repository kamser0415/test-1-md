# 상품 조회 기능 개발하기

목표
: 상품 조회를 검증하는 테스트 코드를 작성합니다.  

## 코드 작성 순서
1. 상품 조회 검증 `then`
2. 상품 조회 `when`
3. 상품 등록 `given`
  
현재 테스트 코드의 목적은 사용자가 상품 고유 번호로 상품 조회를 요청하면 응답으로 조회된 결과가 옳바른지 검증하는게 목적입니다.   
테스트 코드를 작성하는 방법은 1 → 2 → 3 으로 작성하는 방식보다 
검증에 대한 코드를 먼저 작성하는게 불필요한 `Test Fixture`를 안해도 됩니다.
  
`3 → 2 → 1` 으로 코드를 작성합니다.

## 검증 코드 작성하기
```Java
class ProductServiceTest {

    @Test
    @DisplayName("상품 번호로 조회 요청하면 일치하는 상품이 조회가 된다")
    void searchByProductId(){
        // 상품 등록
        
        // 상품 조회
        
        // 상품 검증
        assertThat(response).isNotNull();
    }
}
```
지금은 단순히 `null` 여부만 검증하고 있지만 클라이언트가 요구하는 응답에 대해 검증할 코드먼저 작성합니다.  
  
## 상품 조회 코드 작성하기
```Java
class ProductServiceTest {

    @Test
    @DisplayName("상품 번호로 조회 요청하면 일치하는 상품이 조회가 된다")
    void searchByProductId(){
        // 상품 등록
        
        // 상품 조회
        final GetProductResponse response;
        
        // 상품 검증
        assertThat(response).isNotNull();
    }
}
```  
그리고 사용자가 원하는 응답 객체의 필드를 이후에 작성합니다.  
```Java
public record GetProductResponse(
        Long id,
        String name,
        int price,
        DiscountPolicy discountPolicy
) {
    public GetProductResponse(Long id, String name, int price, DiscountPolicy discountPolicy) {
        this.id = id;
        this.name = name;
        this.price = price;
        this.discountPolicy = discountPolicy;
    }
}
```  
응답 객체를 생성하고, 서비스가 해당 응답 객체를 반환할 수 있도록 코드를 작성합니다.  
```Java
public GetProductResponse getProduct(final Long productId) {
    Product product = productPort.getProduct(productId);
    return new GetProductResponse(
            productId,
            product.getName(),
            product.getPrice(),
            product.getDiscountPolicy()
    );
}
```  
상품 서비스가 상품 포트에게 상품 고유번호로 상품 조회를 요청하는 메서드를 작성하고, 어댑터에서 
해당 조회 기능을 구현합니다.
```Java
@Override
public Product getProduct(final Long productId) {
    return productRepository.findById(productId)
            .orElseThrow(() -> new IllegalArgumentException("존재없음"));
}
```  
어댑터가 의존하는 구현체는 `MyBatis,JDBC,JPA`등 다양한 방식으로 포트 인터페이스를 구현합니다.  
  
## Test Fixture 작성
이렇게 검증을 먼저 작성할 경우, 검증에 필요한 필드만 외부에 들어나도록 준비 코드를 작성하면 됩니다.  
```Java
@Test
@DisplayName("등록된 상품은 상품 고유번호로 조회할 수 있다.")
void searchProductById() {
    //상품을 등록
    productService.addProduct(ProductSteps.createProductRequest());
    final Long productId = 1L;
    //상품을 등록할건데 이건 계층 테스트가 아니라 인수테스트이기 때문에 반복된 작업은 공통화된 클래스로 작성해도 된다.

    //상품을 조회
    final GetProductResponse response = productService.getProduct(productId);

    //조회 결과를 검증
    assertThat(response).isNotNull();
}
```  
RestAssured는 Rest 웹 서비스를 검증하기 위한 라이브러리입니다. 
대부분 `End to End`전구간 테스트를 목적으로 사용됩니다.  
`@SpringBootTest`를 사용하여 실제 요청을 보내고 전체적인 로직을 테스트가 진행됩니다. 

계층별로 테스트할 경우 검증하려는 테스트 케이스외에는 다른 서비스의 메서드를 호출하지 않습니다. 
하지만 인수테스트는 사용자가 실제 `API`를 통해서 등록하는 과정이 포함되어야하기 때문에 
반복되는 `API`요청을 별도의 클래스로 분리해서 활용할 수 있습니다

**ProductSteps**  
```Java
public class ProductSteps {
    public static ExtractableResponse<Response> requestCreateProduct() {
        return RestAssured.given()
                .log().all()
                .contentType(MediaType.APPLICATION_JSON_VALUE)
                .body(createProductRequest())
                .post("/products2")
                .then()
                .log().all()
                .extract();
    }

    public static AddProductRequest createProductRequest() {
        final String name = "상품명";
        final int price = 1000;
        final AddProductRequest request = new AddProductRequest(name, price, DiscountPolicy.NONE);
        return request;
    }
}
```

## 마무리
```Java
@SpringBootTest
class ProductService2Test {

    @Autowired
    private ProductService2 productService;

    @Test
    @DisplayName("등록된 상품은 상품 고유번호로 조회할 수 있다.")
    void searchProductById() {
        //상품을 등록
        productService.addProduct(ProductSteps.createProductRequest());
        final Long productId = 1L;
        //상품을 등록할건데 이건 계층 테스트가 아니라 인수테스트이기 때문에 반복된 작업은 공통화된 클래스로 작성해도 된다.

        //상품을 조회
        final GetProductResponse response = productService.getProduct(productId);

        //조회 결과를 검증
        assertThat(response).isNotNull();
    }
}
```  
이렇게 서비스 역할이 얇을 경우에는 POJO로 먼저 작성하고 테스트를 하는 것보다 스프링 부트를 바로 적용하여 테스트를 해도 상관없습니다.

## API로 전환하기
API 매핑하기 전에 API 테스트 코드를 먼저 작성하고 테스트 실패를 확인합니다.
```Java
@Test
@DisplayName("상품 조회 인수 테스트:")
void searchTest(){
    //상품 등록
    ProductSteps.requestCreateProduct(ProductSteps.createProductRequest());
    Long productId = 1L;

    final ExtractableResponse<Response> response =
            RestAssured.given().log().all()
            .when()
            .get("/products2/{productId}", productId)
            .then().log().all()
            .extract();

    Assertions.assertThat(response.statusCode()).isEqualTo(HttpStatus.OK.value());
}
```  
테스트가 실패하면 웹 요청 매핑을 합니다.
```Java
@GetMapping("/{productId}")
public ResponseEntity<GetProductResponse> getProduct(@PathVariable final Long productId) {
    final Product product = productPort.getProduct(productId);

    final GetProductResponse response = new GetProductResponse(
            product.getId(),
            product.getName(),
            product.getPrice(),
            product.getDiscountPolicy());
    return ResponseEntity.ok(response);
}
```
이제 테스트 코드가 통과합니다.

## 아쉬운점
테스트를 작성할 때 공통으로 사용하는 api요청은 별도로 재사용하는 건 좋지만 
어떤 상품을 등록했는지 내부 로직에 숨겨 있어서 다른 사람이 이 인수 테스트를 볼때 
다시 공통 모듈로 들어가야 한다는 점이 조금 아쉽습니다.  

그리고 공통 api가 변경된다면 다른 인수 테스트도 전부 실패하기 때문에 
공통 모듈로 분리하는건 좋으나 테스트에 필요한 값은 외부에서 주입받는것이 좋을거라 생각됩니다.  

