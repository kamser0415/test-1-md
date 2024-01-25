# 스프링 부트와 API Test

## Spring
강사님과 강의를 보기전 작성한 코드 비교
```Java
@SpringBootTest
public class ProductServiceWithBootTest {

    @Autowired
    private ProductService2 productService;

@Test
void 상품수정() {
    //given
    final long updateProductId = 1L;
    final String updateName = "수정된이름";
    final int updatePrice = 2000;
    final DiscountPolicy updateDisCountPolicy = DiscountPolicy.PERCENT_10;

    final AddProductRequest productRequest = ProductSteps.createProductRequest();
    productService.addProduct(productRequest);

    //when
    productService.updateProduct(updateProductId, new UpdateProductRequest(updateName, updatePrice, updateDisCountPolicy));

    //then
    final Product findProduct = productPort.getProduct(updateProductId);
    Assertions.assertThat(findProduct).isNotNull()
            .extracting("name","price","discountPolicy")
            .contains(updateName,updatePrice,updateDisCountPolicy);
}
```
현재 코드 문제점
:
SpringBootTest → 인수 테스트로 넘어갈 때 콭드를 다시 작성해야합니다.

```Java
@Test
void 상품수정() {
    //given
    final long updateProductId = 1L;
    final String updateName = "수정된이름";
    final int updatePrice = 2000;
    final DiscountPolicy updateDisCountPolicy = DiscountPolicy.PERCENT_10;

    productService.addProduct(createProductRequest());

    //when
    //강사님 코드
    productService.updateProduct(updateProductId, 수정요청객체());
    //수정후 코드
    productService.updateProduct(updateProductId, createUpdateProductRequest(updateName, updatePrice, updateDisCountPolicy));

    //then
    final ResponseEntity<GetProductResponse> findProduct = productService.getProduct(updateProductId);
    Assertions.assertThat(findProduct.getBody()).isNotNull()
            .extracting("name", "price", "discountPolicy")
            .contains(updateName, updatePrice, updateDisCountPolicy);
}
```
강사님 코드 수정
: 
인수 테스트라도 검증하는 부분할 때 어떤 값으로 수정을 요청했는지 알 수가 없고,
저 then이 맞는지 틀린지 검증을 확인을 하려면 `ProductSteps`라는 공통 유틸 클래스로 어떤 값이 저장되는지 확인해야합니다.  


## 인수테스트
API 인수테스트를 할 때, 검증부터 작성합니다. 
필요한 검증에 대한 데이터를 저장하고 준비하기 때문에 불필요한 `Test Fixture`가 없어집니다.  

**검증에 사용할 정보를 저장합니다.**  
```Java
@Test
@DisplayName("상품 수정")
void 상품수정() {
    //given
    
    //when
    
    //then
     var getResponse = RestAssured.given().log().all()
                        .when().get("/products2/{productsId}",productId)
                        .then().log().all()
                        .extract();
    
    assertThat(updateResponse.statusCode()).isEqualTo(HttpStatus.OK.value());
    assertThat(getResponse.body().jsonPath().getString("name")).isEqualTo(updateName);
    assertThat(getResponse.body().jsonPath().getInt("price")).isEqualTo(updatePrice);
    assertThat(getResponse.body().jsonPath().getString("discountPolicy")).isEqualTo(updateDisCountPolicy.name());
}
```

**when → given 작성**
```Java
@Test
@DisplayName("상품 수정")
void 상품수정() {
    //given
    final long productId = 1L;
    ProductSteps.requestCreateProduct(ProductSteps.createProductRequest());

    final String updateName = "변경된 이름";
    final int updatePrice = 2000;
    final DiscountPolicy updateDisCountPolicy = DiscountPolicy.NONE;

    //when
    var updateResponse = RestAssured
            .given().log().all()
                .contentType(MediaType.APPLICATION_JSON_VALUE)
                .body(ProductSteps.createUpdateProductParam(updateName, updatePrice, updateDisCountPolicy))
            .when()
                .patch("/products2/{productId}", productId)
            .then().log().all()
                .extract();

    //then
    var getResponse = RestAssured.given().log().all()
                    .when().get("/products2/{productsId}",productId)
                    .then().log().all()
                    .extract();

    assertThat(updateResponse.statusCode()).isEqualTo(HttpStatus.OK.value());
    assertThat(getResponse.body().jsonPath().getString("name")).isEqualTo(updateName);
    assertThat(getResponse.body().jsonPath().getInt("price")).isEqualTo(updatePrice);
    assertThat(getResponse.body().jsonPath().getString("discountPolicy")).isEqualTo(updateDisCountPolicy.name());
}
```
## RED
```json
{
    "timestamp": "2024-01-21T04:19:28.743+00:00",
    "status": 405,
    "error": "Method Not Allowed",
    "path": "/products2/1"
}
```
서비스 구현체가 아직 웹 매핑이 안되서 테스트에 실패합니다.

## GREEN
상품 수정 메서드를 매핑합니다.
```Java
@PatchMapping("/{productsId}")
public ResponseEntity<Void> updateProduct(@PathVariable("productsId") final Long productId,@RequestBody final UpdateProductRequest updateProductRequest) {
    //throw error
    final Product product = productPort.getProduct(productId);
    product.update(updateProductRequest.updateName(), updateProductRequest.updatePrice(), updateProductRequest.updateDisCountPolicy());
    productPort.save(product);

    return ResponseEntity.ok().build();
}
```

## 리팩토링
상품 수정 API를 `ProductSteps`에 저장하고, 나머지 API 요청도 모두 `static import`로 정리합니다.
```Java
@Test
@DisplayName("상품 수정")
void 상품수정() {
    //given
    final long productId = 1L;
    requestCreateProduct(createProductRequest());

    final String updateName = "변경된 이름";
    final int updatePrice = 2000;
    final DiscountPolicy updateDisCountPolicy = DiscountPolicy.NONE;

    //when
    var updateResponse = requestUpdateProduct(updateName, updatePrice, updateDisCountPolicy, productId);

    //then
    var getResponse = 상품조회요청(productId);

    assertThat(updateResponse.statusCode()).isEqualTo(HttpStatus.OK.value());
    assertThat(getResponse.body().jsonPath().getString("name")).isEqualTo(updateName);
    assertThat(getResponse.body().jsonPath().getInt("price")).isEqualTo(updatePrice);
    assertThat(getResponse.body().jsonPath().getString("discountPolicy")).isEqualTo(updateDisCountPolicy.name());
}
```

강사님 코드와 다르지만, 
인수 테스트를 할 때도 외부에서 어떤 값으로 요청하는지 테스트 코드만 봐도 확인할 수 있어야한다고 생각합니다.  
  
## 추가수정
```Java
@Test
@DisplayName("상품 수정")
void 상품수정() {
    //given
    final long productId = 1L;
    requestCreateProduct(createProductRequest("상품명", 1000, DiscountPolicy.NONE));

    final String updateName = "변경된 이름";
    final int updatePrice = 2000;
    final DiscountPolicy updateDisCountPolicy = DiscountPolicy.NONE;

    //when
    var updateResponse = requestUpdateProduct(productId, createUpdateProductParam(updateName, updatePrice, updateDisCountPolicy));

    //then
    var getResponse = 상품조회요청(productId);

    assertThat(updateResponse.statusCode()).isEqualTo(HttpStatus.OK.value());
    assertThat(getResponse.body().jsonPath().getString("name")).isEqualTo(updateName);
    assertThat(getResponse.body().jsonPath().getInt("price")).isEqualTo(updatePrice);
    assertThat(getResponse.body().jsonPath().getString("discountPolicy")).isEqualTo(updateDisCountPolicy.name());
}
```
