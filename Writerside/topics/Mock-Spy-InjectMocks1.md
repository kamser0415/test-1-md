# @Mock,@Spy,@InjectMocks
**순수 Mockito로 검증하기**
[mockito-mock-methods](https://www.baeldung.com/mockito-mock-methods)

MockBean 한계  
: `MockBean`은 스프링 컨테이너에 실제 객체 대신 가짜 객체(`Mock`)가 등록됩니다. 
스프링 컨테이너를 사용하지 않는 단위테스트 일 경우 사용할 수가 없습니다.  
  
## 프로그래밍식 방식
`Mockito` 프레임워크를 사용하여 가짜 객체를 프로그래밍식으로 작성하여 테스트하는 방법입니다.  

가짜 객체를 만드는 방법
```Java
MailSendClient mailSendClient = Mockito.mock(MailSendClient.class);
```
`Stubbing`하여 개발자가 원하는 행위를 하게 만듭니다.
```Java
Mockito.when(mailSendClient.sendEmail(anyString(),anyString(),anyString(),anyString()))
            .thenReturn(true);
```

테스트 코드
```Java
@DisplayName("발신자,수신자,제목,내용으로 메일을 전송하여 성공하면 히스토리를 기록한다.")
@Test
void sendMail(){
    //given
    MailSendClient mailSendClient = Mockito.mock(MailSendClient.class);
    MailSendHistoryRepository sendHistoryRepository = Mockito.mock(MailSendHistoryRepository.class);

    MailService mailService = new MailService(mailSendClient, sendHistoryRepository);

    //Stubbing
    Mockito.when(mailSendClient.sendEmail(anyString(),anyString(),anyString(),anyString()))
            .thenReturn(true);

    //when
    Boolean result = mailService.sendMail("", "", "", "");

    //then
    Assertions.assertThat(result).isTrue();
}
```

## 선언 방식
애노테이션을 활용하여 간편하게 구현하는 방법도 있습니다. 
대신 테스트 코드 클래스 레벨에 추가 애노테이션으로 확장기능을 사용해야합니다.
```Java
@ExtendWith(MockitoExtension.class)
class MailServiceTest {
    @Mock
    MailSendHistory mailSendHistory;
    @Mock
    private MailSendHistoryRepository mailSendHistoryRepository;
    @InjectMocks// 해당 클래스의 생성자 인자를 보고 @Mock으로 된 애들을 DI해준다.
    private MailService mailService;
}
```  
@ExtendWith(MockitoExtension.class)
: 테스트 프레임워크가 동작할 때 확장 기능으로 `Mockito`를 사용한다는 의미입니다.

@Mock
: `Mockito.mock(MailSendClient.class)`와 동일한 코드입니다.  

@InjectMocks
: 해당 애노테이션이 붙은 클래스를 생성시 필요한 의존성이 `@Mock`으로 선언된 경우 주입해줍니다.
   
## @Mock 주의사항
`@Mock`으로 대체된 오브젝트의 매서드는 `Mockito.when`으로 행위를 지정하지 않는 이상
해당 반환 타입의 **기본 초기화 값**을 반환합니다.  
```Java
public static <T> T mock(Class<T> classToMock) {
    return mock(classToMock, withSettings());
}
```  
공식 문서 설명
: 일반적으로 빈 값을 반환합니다.  
Answer를 사용하면 스텁되지 않은 호출의 반환 값을 정의할 수 있습니다.  
이 구현은 먼저 전역 설정을 시도하고 전역 설정이 없으면 0, 빈 컬렉션, null 등을 반환하는 **기본 응답을 사용**합니다.  

## 호출 횟수나 타임아웃등 검증하기
`Mockito.verify()`를 활용하여 전달된 Mock 클래스의 특정 행위에 대한 기록을 검증할 수 있습니다.  
예를 들면 호출 횟수나, 타임아웃 등이 해당됩니다.
문법
```Java
verify(mock, timeout(100).times(5)).foo();
verify(mock, timeout(200).atLeastOnce()).baz();
```
테스트 코드
```Java
@DisplayName("발신자,수신자,제목,내용으로 메일을 전송하여 성공하면 히스토리를 기록한다.")
@Test
void sendMail(){
    //given
    MailSendClient mailSendClient = Mockito.mock(MailSendClient.class,Mockito.withSettings().useConstructor().defaultAnswer(Mockito.CALLS_REAL_METHODS));
    MailSendHistoryRepository sendHistoryRepository = Mockito.mock(MailSendHistoryRepository.class);

    MailService mailService = new MailService(mailSendClient, sendHistoryRepository);

    //Stubbing
    //Spy는 실제 객체가 들어오기때문에 WHEN을 사용하면 안된다.
    Mockito.when(mailSendClient.sendEmail(anyString(),anyString(),anyString(),anyString()))
            .thenReturn(true);

    //when
    Boolean result = mailService.sendMail("", "", "", "");
    
    //기록된 호출 횟수를 비교하여 검증할 수 있습니다.
    Mockito.verify(sendHistoryRepository, Mockito.times(1))
            .save(any(MailSendHistory.class));
    //then
    Assertions.assertThat(result).isTrue();
}
```  

## @Spy
사용 목적
: 실제 객체의 메서드의 기능은 정상 동작하도록 하고, 그중 일부 행위만 원하는 행위로 동작시키고 싶을 때
사용합니다.  

내부 코드
:
```Java
MOCKITO_CORE.mock(
classToSpy, withSettings().useConstructor().defaultAnswer(CALLS_REAL_METHODS));
```  
  
내부 코드를 보면 `mock`으로 동작하는데 응답을 실제 메소드를 응답으로 사용합니다.

**프로그래밍식**  
```Java
MailSendClient mailSendClient = Mockito.spy(MailSendClient.class);
```  

**선언방식**  
```Java
@Spy
MailSendClient mailSendClient
```  

**Spy 테스트 코드 작성 방법**
```Java
Mockito.doReturn(true)
       .when(mailSendClient)
       .sendEmail(any(String.class), anyString(), any(String.class), any(String.class));
```  
`verify`와 문법이 비슷합니다. 

