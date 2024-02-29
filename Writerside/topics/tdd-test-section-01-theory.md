# 이론 수업

[출처](https://inf.run/EYKf)

## 실기로 배우는 간단한 테스트 작업 방법 소개  
  
먼저 간단한 계산기를 만들고 테스트 코드를 넣어서 리팩토링을 합니다.
```Java
public class Main {

    public static void main(String[] args) {

        System.out.println("1 + 2 :");
        Scanner scanner = new Scanner(System.in);
        String result = scanner.nextLine();


        String[] parts = result.split(" ");
        long num1 = Long.parseLong(parts[0]);
        long num2 = Long.parseLong(parts[2]);
        String operation = parts[1];

        long answer = 0;
        switch (operation) {
            case "+" :
                answer = num1 + num2;
                break;
            case "-" :
                answer = num1 - num2;
                break;
            case "*" :
                answer = num1 * num2;
                break;
            case "/" :
                answer = num1 / num2;
                break;
            default:
                throw new InvalidOperatorException();
        }

        System.out.println("answer = " + answer);
    }
}
```  
  
**1. java 12버전 인텔리제이 리팩토링**
```Java
 long answer = switch (operation) {
    case "+" -> num1 + num2;
    case "-" -> num1 - num2;
    case "*" -> num1 * num2;
    case "/" -> num1 / num2;
    default -> throw new InvalidOperatorException();
};
```  
  
`switch`문장을 별도의 클래스로 분리하기
```Java
package org.example;

public class Calculator {

    public long calculate(long num1, String operator, long num2) {
        return switch (operator) {
            case "+" -> num1 + num2;
            case "-" -> num1 - num2;
            case "*" -> num1 * num2;
            case "/" -> num1 / num2;
            default -> throw new InvalidOperatorException();
        };
    }
}
```  
  
**2. 데이터 받는 부분 리팩토링**  
```Java
public class CalculatorRequestReader {
    public String[] read() {
        System.out.println("1 + 2 :");
        Scanner scanner = new Scanner(System.in);
        String result = scanner.nextLine();
        return result.split(" ");
    }
}
```
  
**3. 데이터 구조화**  
데이터를 단순 기본형 타입으로 나누어서 관리하는 것보다 구조화로 관리할 수 있으면 
리팩토링을 통해서 구조화를 하는 것이 좋다.  
  
추가로 모든 데이터 필수이므로 final을 추가하여 `VO`로 선언합니다.
```Java
public class CalculationRequest {
    private final long num1;
    private final long num2;
    private final String operation;

    public CalculationRequest(String[] parts) {
        this.num1 = Long.parseLong(parts[0]);
        this.num2 = Long.parseLong(parts[2]);
        this.operation = parts[1];
    }

    public long getNum1() {
        return num1;
    }

    public long getNum2() {
        return num2;
    }

    public String getOperation() {
        return operation;
    }
}
```  
  
추가로 검증 로직을 추가합니다.


### 불필요한 값을 모두 받을 필요가 없다.
**검증로직**
```Java
public CalculationRequest(String[] parts) {
    if (parts.length != 3) {
        throw new RedRequestException();
    }
    if (parts[1].length() != 1) {
        throw new InvalidOperatorException();
    }
    if (isInvalidOperator(parts)) {
        throw new InvalidOperatorException();
    }
    this.operation = parts[1];
    this.num1 = Long.parseLong(parts[0]);
    this.num2 = Long.parseLong(parts[2]);
}
```
```Java
private static boolean isInvalidOperator(String[] parts) {
    return  !parts[1].equals("+") && 
            !parts[1].equals("-") && 
            !parts[1].equals("*") && 
            !parts[1].equals("/");
}
```  
여기서 `isInvalidOperator` 메서드는 입력값으로 Operation이 제대로 된 문자열인지 검증을 하는 기능을 합니다.
불필요한 값을 받을 필요가 없기 때문에 아래와 같이 리팩토링 합니다.
```Java
private static boolean isInvalidOperator(String operation) {
    return !operation.equals("+") &&
            !operation.equals("-") &&
            !operation.equals("*") &&
            !operation.equals("/");
}
```

**4. 추가 리팩토링**  
```Java
public class CalculationRequest {
    private static final Set<String> VALID_OPERATORS = Set.of("+", "-", "*", "/");

    private final long num1;
    private final long num2;
    private final String operation;

    public CalculationRequest(String[] parts) {
        if (parts == null || parts.length != 3) {
            throw new IllegalArgumentException("Request must contain exactly three parts: num1, operator, num2");
        }
        String operator = parts[1];
        if (operator == null || operator.length() != 1 || !VALID_OPERATORS.contains(operator)) {
            throw new IllegalArgumentException("Invalid operator. Valid operators are: " + VALID_OPERATORS);
        }
        this.operation = operator;
        try {
            this.num1 = Long.parseLong(parts[0]);
            this.num2 = Long.parseLong(parts[2]);
        } catch (NumberFormatException e) {
            throw new IllegalArgumentException("Both numbers must be valid long values", e);
        }
    }

    public long getNum1() {
        return num1;
    }

    public long getNum2() {
        return num2;
    }

    public String getOperation() {
        return operation;
    }
}
```  
문자열을 비교하는 것보다 `Set`자료형을 활용해서 검증하는 게 더 유지보수에 유리할 수 있습니다.  
그리고 if문이 사이에 있는 검증은 별도의 메서드로 분리해서 처리하는것도 가독성 좋게 합니다.  
  
## 테스트란 ?
테스트는 어떤 시스템이 제대로 동작하는지 검증하는 과정입니다.  
  
1. 인수 테스트: 사람이 직접 사용하면서 준비된 체크리스트를 체크하는 인수테스트
2. 자동 테스트: 미리짜여진 코드를 비교하는 것.  
  
### 샘플
```Java
@Test
@DisplayName("이메일 회원가입을 할 수 있다")
public void email(){
    //given
    UserCreateRequest userCreateRequest = UserCreateRequest.builder()
        .email("foo@localhost.com")
        .password("1234")
        .build();
    
    FakeRegisterEmailSender registerEmailSender = new FakeRegisterEmailSender();
    
    //when
    UserService sut = UserService.builder()
        .registerEmailSender(RegisterEableSender)
        .userRepository(userRepository)
        .build();
        
    sut.register(userCreateRequest);
    
    //then
    User user = userRepository.getEmail("foo@localhost.com");
    assertThat(user.isPending()).isTrue();
    assertThat(registerEmailSender.findLatestMessage("foo@localhost.com").isPresent()).isTrue();
    assertThat(registerEmailSender.findLatestMessage("foo@localhost.com").get()).isEqualTo("~~~");
``` 
  
샘플 코드를 보면 상황이 주어지고, 테스트하려는 기능에 대한 응답 결과를 테스트합니다.  
**결과값과 기댓값을 코드로 비교하는 것이 자동화 테스트입니다.**  
  
### Spring Data Jpa 샘플 테스트
```Java
@ExtendWith(SpringExtension.class)
@DataJpaTest
@TestPropertySource("classpath:test-application.properties")
class UserRepositoryTest {
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    @DisplayName("사용자 정보를 save로 저장할 수 있다.")
    void save1(){
        //given
        UserEntity userEntity = new UserEntity();
        userEntity.setEmail("kok2020@naver.com");
        userEntity.setStatus(UserStatus.PENDING);
        userEntity.setCertificationCode("hash123456789");
        userEntity.setAddress("Seoul");
        userEntity.setLastLoginAt(Clock.systemUTC().millis());
        
        //when
        userEntity = userRepository.save(userEntity);
        
        //then
        assertThat(userEntity.getId()).isNotNull();
        
    }
}
```  
  
저장이 제대로 되었는지 확인하기 위해 테스트 코드를 작성합니다.  
  

## TDD (테스트 주도 개발)
  
**TDD로 작성하려는 서비스 메소드**  
```Java
@Service
@RequiredArgsConstructor
public class PostService {

    public PostEntity getPostById(Long id) {
        throw new RuntimeException("Method is not implements");
    }
}
```  
  
**when부분을 제외하고 준비**   
```Java
@SpringBootTest
@TestPropertySource("classpath:test-application.properties")
class PostServiceTest {
    @Autowired
    private PostService postSerice;
    
    @Autowired
    private UserRepository userRepository;

    @Autowired
    private PostRepository postRepository;
    
    @Test
    @DisplayName("게시물을 아이디로 조회 할 수 있다.")
    void selectById(){
        //given
        UserEntity userEntity = userRepository.save(new UserEntity());
        PostEntity postEntity = new PostEntity();
        postEntity.setWriter(userEntity);
        postEntity.setContent("hello world");
        postEntity.setCreatedAt(Clock.systemUTC().millis());
        postEntity.setModifiedAt(Clock.systemUTC().millis());
        postEntity = postRepository.save(postEntity);
        
        //when
        PostEntity result = postService.getPostById(postEntity.getId());
        
        //then
        assertThat(postEntity.getContent()).isEqualTo("hello world");
    }   
}
```  
  
테스트는 `RED`로 실패하게 됩니다.  
여기서 중요한 건 레드 단계에서는 테스트가 실패하는지까지 제대로 확인하는게 중요합니다.  
  
**GREEN 단계: 실제구현**  
```Java
@Service
@RequiredArgsConstructor
public class PostService {

    public PostEntity getPostById(Long id) {
        return postRepository.findById(id)
            .orElseThrow(()-> new ResourceNotFoundException("Post",id));
    }
}
```   
  
**BLUE : 리팩토링**  
초록불을 본 이후로 부터는 리팩토링을 합니다.  
  
### 장점  
#### 깨지는 테스트를 먼저 작성해야하기 때문에, 인터페이스를 먼저 만드는 것이 강제된다.
인터페이스를 생각하지 않고 먼저 구현체를 만드는 실수가 줄어듭니다.  
TDD에 따르면 레드 단계에서는 일단 컴파일되는 코드를 먼저 만들고 테스트가 실패하는지까지 확인해야합니다.  
그래서 개발자가 구현체를 만드는 것보다 인터페이스를 만드는 것에 집중하게 됩니다.  
객체들이 어떤 책임을 지고 이 객체는 어디까지 해줘야 하는지 그런걸 먼저 생각하게 만드는 겁니다.  
  
인터페이스를 주목해서 만든다는 건 객체지향의 핵심 원리중 하나인 행동에 집중할 수 있습니다.  
  
TDD가 What/Who 사이클을 고민하게 도와줍니다.  
What/Who 사이클은 **어떤 행위를 누가 수행할지 결정하는 과정** 입니다.  
  
TDD로 개발하는 코드는 프로젝트의 대부분의 코드를 커버하는 테스트가 생기게 됩니다.  
신규 개발을 하더라도 부담없이 개발할 수 있는거죠.  
이미 테스트 코드가 작성되어 전체 테스트 코드를 돌리면 문제를 파악할 수 있습니다.  
  
### 단점
#### 초기 개발 비용이 부담될 수 있습니다.  
#### 난이도가 있다.
  
## 개발자의 고민  
  
### 무의미한 테스트  
이미 검증이 완료된 `getPostById()`를 테스트하기 위해 60줄이넘는 테스트 코드를 작성해야합니다.  
`Jpa`를 테스트하기위해서 스프링 컨테이너도 실행되어야하고 , H2 데이터베이스도 연동되어야하기 때문에 
무거운 테스트가 됩니다.  
  
그런데 TDD 테스트 코드를 작성하는 장점이 인터페이스를 먼저 만들기 때문에 행동에 집중할 수 있다는 것인데  
행동은 메서드나 함수가 아닙니다. 중요한 로직을 잘 구분해서 그 코드에 테스트 코드를 넣는게 좋습니다.  
  
### 느리고 쉽게 깨지는 테스트  
위 처럼 데이터가 제대로 저장되었는지 확인하기 위해서 `300ms`가 필요합니다. 
여기서 동일한 방식의 테스트가 200개가 된다면 1분이 넘는 시간이 테스트 코드에 소비됩니다.  
  
H2 DB를 사용하다보니 테스트가 종종 깨질 수 도 있습니다.  

### 테스트가 불가능한 코드
시간과 같은 동일한 값이 들어가지 않는 경우에 수정된 값을 확인할 수 없기 때문에 
`mock`을 사용하게 되는데 그건 설계가 잘못되었다는 의미로 구조를 변경해야합니다.  
  
