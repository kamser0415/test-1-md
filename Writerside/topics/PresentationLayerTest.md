# PresentationLayerTest

외부 세계의 요청을 가장 먼저 받는 계층
: 클라이언트가 전달한 값, 프론트 엔트쪽에서 넘겨주는 값
  
> 파라미터에 대한 최소한의 검증을 수행한다.  
> 전체 비즈니스 로직을 테스트하는 게 아니라 전달 받은 값이 
> 유요한지 테스트하는게 제일 중요하다.    
> 
  
<procedure>
     <step>
        <p><strong>최소한의 데이터만 조건만 검사한다:</strong></p>
        <p>최소한의 데이터만을 조건으로 검사하여 유효성을 확인합니다.</p>
    </step>
    <step>
        <p><strong>메세지도 구체적이지 않고 데이터가 안들어왔다는 표시만 한다:</strong></p>
        <p>메시지가 구체적이지 않고 데이터가 없다는 표시만을 나타내는 메시지를 표시합니다.</p>
    </step>
    <step>
        <p><strong>특수 형태 Validation은:</strong></p>
        <p>서비스 레이어나 도메인 객체(Product 등)를 생성하는 시점에 생성자에서 검증을 하거나,</p>
        <p>더 안쪽 레이어에서 검증하는 것이 적절하다고 생각합니다.</p>
    </step>
    <step>
        <p><strong>성격에 따라서 어느 레이어에서 검증을 할지 모든 검증을:</strong></p>
        <p>한 레이어 계층에서 할 필요는 없습니다.</p>
    </step>
    <step>
        <p><strong>책임을 분리를 해서 어느 계층에서 검증을 할지 고민을 해보는게 좋다:</strong></p>
        <p>검증을 어느 레이어에서 수행할지를 결정할 때, 책임을 분리하고 각 레이어의 역할을 고민합니다.</p>
    </step>
</procedure>
  
## 테스트 방법  
서비스 계층을 통합테스트로 진행할 때 `Persistence`와`Service`계층을 같이 테스트하려고 
`@SpringBootTest`를 추가하여 스프링 컨테이너를 띄우고 테스트를 진행할 빈을 주입받아서 진행했습니다.  
  
> 컨트롤러 계층은 서비스 계층의 통합테스트 및 단위테스트, 데이터 접속 계층의 단위테스트를 모두 올바르게 작성후
> 통과되었다는 전제가 필요합니다.  
   
컨트롤러 계층만 `고립`시켜서 단위테스트로 진행합니다.  
  
## 테스트 목적  
`when`으로 특정 매서드 호출시 반환 값을 설정할 수 있지만, 여기서는 사용하지 않습니다.  

서비스 계층이 올바른 비즈니스 로직을 수행할 수 있도록 필터링된 데이터를 전달하는 계층입니다. 
따라서 내부의 비즈니스 로직을 테스트를 하는게 목적이 아니라 전달 받은 데이터의 유효성에 대해서 테스트를 진행합니다.  
  
컨트롤러 계층을 테스트하기 위해서 서비스 빈을 주입받아야 빈이 생성할 수 있는데 
이때 사용하는 라이브러리가 `Mockito`입니다.  

## Mock
Mock  
: 하나의 계층을 테스트하기 위해 불필요한 준비 과정을 줄이기 위해 사용하는 라이브러리입니다.  
  
### 테스트시 필요한 애노테이션의 차이점
| Annotation        | 공통점                 | 차이점                                                                                      |
|-------------------|---------------------|------------------------------------------------------------------------------------------|
| `@WebMvcTest`     | - 스프링 MVC 컨트롤러 테스트  | - 웹 계층에 초점 - `@Controller`, `@ControllerAdvice`와 함께 사용 - 다른 빈들은 로드되지 않음 (mocking을 통해 대체) |
| `@SpringBootTest` | - 전체 애플리케이션 컨텍스트 로드 | - 통합 테스트 - 애플리케이션의 전반적인 동작 확인 - 모든 빈들이 로드됨 - 테스트 속도가 느릴 수 있음                             |
| `@DataJpaTest`    | - JPA 관련 빈들만 로드     | - 데이터베이스와 관련된 테스트 - JPA 리포지토리 테스트 - 테스트 속도가 빠름 - 실제 데이터베이스 연결 없이도 테스트 가능                 |

### MockMvc
가짜 객체를 사용해서 Spring MVC 동작을 재현할 수 있는 테스트 프레임워크입니다.  
    
`OrderController`를 테스트를 진행합니다. 
```Java
@WebMvcTest(controllers = OrderController.class)
class OrderControllerTest {
    @MockBean  // 스프링 컨테이너에 해당 빈을 대신 등록합니다.
    OrderService orderService;
    @Autowired // controller API를 테스트 할 수 있도록 도와줍니다.
    MockMvc mockMvc;
    @Autowired // JSON으로 직렬화 , 역직렬화를 하는 라이브러리
    ObjectMapper objectMapper;
}
```    
  
테스트를 진행할 컨트롤러가 필요한 의존 객체(`OrderService`)를 `@MockBean`을 통해서 아무 기능이 없는 프록시 빈 오브젝트를 등록합니다.  
  
### ValidationTest  
컨트롤러는 비즈니스 로직을 실행할 때 필요한 유요한 값을 검증하는 목적입니다.
```Java
@DisplayName("상품 번호 리스트로 주문을 생성한다.")
@Test
void t1() throws Exception {
    //given
    OrderCreateRequest request = OrderCreateRequest.builder()
            .productNumbers(List.of("001", "002"))
            .build();
    OrderResponse build = OrderResponse.builder().build();
    Mockito.when(orderService.createOrder(ArgumentMatchers.any(), ArgumentMatchers.any()))
                    .thenReturn(build);

    //when //then
    mockMvc.perform(MockMvcRequestBuilders.post("/api/v1/order/new")
            .contentType(MediaType.APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(request)))
            .andDo(MockMvcResultHandlers.print())
            .andExpect(MockMvcResultMatchers.jsonPath("$.code").value(200))
            .andExpect(MockMvcResultMatchers.jsonPath("$.status").value("OK"))
            .andExpect(MockMvcResultMatchers.jsonPath("$.message").value("OK"))
            .andExpect(MockMvcResultMatchers.jsonPath("$.data").exists());
}
```
  
#### jsonPath로 직접 값 비교
```Java
.andExpect(MockMvcResultMatchers.jsonPath("$.code").value(200))
.andExpect(MockMvcResultMatchers.jsonPath("$.status").value("OK"))
.andExpect(MockMvcResultMatchers.jsonPath("$.message").value("OK"))
.andExpect(MockMvcResultMatchers.jsonPath("$.data").exists());
```
  
#### 응답 객체로 반환후 검증
```Java
void test() throws Exception {
OrderCreateRequest request = OrderCreateRequest.builder()
        .build();

ResultActions resultActions = mockMvc.perform(MockMvcRequestBuilders.post("/api/v1/order/new")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
        .andDo(MockMvcResultHandlers.print());

MvcResult mvcResult = resultActions.andReturn();
String jsonResponse = mvcResult.getResponse().getContentAsString();
ApiResponse response = objectMapper.readValue(jsonResponse, ApiResponse.class);

// 이제 response 객체를 사용하여 필요한 검증 수행
Assertions.assertThat(response.getCode()).isEqualTo(400);
Assertions.assertThat(response.getStatus()).isEqualTo("BAD_REQUEST");
Assertions.assertThat(response.getMessage()).isEqualTo("BAD_REQUEST");
Assertions.assertThat(response.getData()).isNull();
}
```