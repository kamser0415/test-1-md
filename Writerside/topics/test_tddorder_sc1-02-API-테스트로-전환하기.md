# 상품등록 api 개발
목표 
: 사용자가 HTTP 요청으로 상품 등록 요청을 보냈을때 , 이제 상품을 메모리에 저장하고 
201 created 스테이스 코드로 응답하는 API 테스트로 전환합니다.  

## 환경설정
> 스프링 부트 3.2.1 기준 rest-Assured 5.3.2를 사용
```Gradle
testImplementation 'io.rest-assured:rest-assured:5.3.2'
```  

## RestAssured 통합환경 
```Java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class ApiTest {

    @LocalServerPort
    private int port;

    @BeforeEach
    void setUp() {
        RestAssured.port = port;
    }
}
```
+ `LocalServerPort`는 `@Value("${local.server.port}")`와 동일
+ `SpringBootTest.WebEnvironment.RANDOM_PORT`는 프로퍼티 환경의 Port를 랜덤하게 하겠다는 의미  
+ RestAssured.port = port는 바인딩된 `RANDOM_PORT`를 매번 받아서 웹 요청을 만듭니다.  
  
## API 요청 응답 객체 수정하기
```Java
@SpringBootTest
public class ProductApiTest2 {
    @Autowired
    private ProductService productService;
    @Test
    void 상품등록(){
        productService.addProduct(createProductRequest());
    }
    private static AddProductRequest createProductRequest() {
        final String name = "상품명";
        final int price = 1000;
        final AddProductRequest request = new AddProductRequest(name, price, DiscountPolicy.NONE);
        return request;
    }
}
```
여기서 `RestAssured`를 통해서 API 요청과 결과를 테스트할 수 있도록 변경합니다.

```Java
@Test
void 상품등록(){
    productService.addProduct(createProductRequest());

    //API 요청
    final ExtractableResponse<Response> response = RestAssured.given()
            .log().all()
            .contentType(MediaType.APPLICATION_JSON_VALUE)
            .body(createProductRequest())
            .post("/products")
            .then()
            .log().all()
            .extract();

    Assertions.assertThat(response.statusCode()).isEqualTo(HttpStatus.CREATED.value());
}
```

## RED
```Java
org.opentest4j.AssertionFailedError: 
expected: 201
 but was: 404
Expected :201
Actual   :404
```
테스트 코드가 실패합니다.

## GREEN
해당 서비스 컴포넌트가 웹 요청을 처리할 수 있도록 매핑 애노테이션을 추가합니다.
```Java
@RestController
@RequestMapping("/products2")
public class ProductService2 {
    private final ProductPort productPort;

    ProductService2(ProductPort productPort) {
        this.productPort = productPort;
    }

    @PostMapping
    public ResponseEntity<Void> addProduct(@RequestBody final AddProductRequest request) {
        final Product product = new Product(request.name(), request.price(), request.discountPolicy());
        productPort.save(product);
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }
}
```

테스트가 성공합니다.

## 리팩토링
```Java
@Test
void 상품등록(){
    productService.addProduct(createProductRequest());

    //API 요청

    final ExtractableResponse<Response> response = requestCreateProduct();

    Assertions.assertThat(response.statusCode()).isEqualTo(HttpStatus.CREATED.value());
}

private static ExtractableResponse<Response> requestCreateProduct() {
    return RestAssured.given()
            .log().all()
            .contentType(MediaType.APPLICATION_JSON_VALUE)
            .body(createProductRequest())
            .post("/products2")
            .then()
            .log().all()
            .extract();
}
```  
테스트 코드를 리팩토링을 했습니다.

## 정리
테스트 순서
: 
1. 먼저 웹 요청 환경을 만들고 등록이 안된 주소로 일단 전송해본다.
2. 테스트가 실패한다.
3. 빠르게 테스트를 통과할 수 있도록 웹 요청 매핑을 한다.
4. 테스트 통과후 , 테스트 코드와 테스트 객체를 리팩토링 합니다.