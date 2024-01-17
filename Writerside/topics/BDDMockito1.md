# BDDMockito
행동 주도 개발
: given/when/then으로 함수 단위가 아니라 행동 위주로 검증하는 방식  
  
## 현재 코드
```Java
@DisplayName("발신자,수신자,제목,내용으로 메일을 전송하여 성공하면 히스토리를 기록한다.")
@Test
void sendMail(){
    //given
    Mockito.when(mailSendClient.sendEmail(anyString(),anyString(),anyString(),anyString()))
            .thenReturn(true);
}
```  
`//given`절에 `when()`이라는 문법을 사용합니다.  
테스트 코드를 준비하는 과정에 `when()`이라는 구절보다 `given()`이라는 구절로 표현할 수 있게 도와주는 게 
`BDDMockito`입니다.  

## 수정 코드  
```Java
@DisplayName("발신자,수신자,제목,내용으로 메일을 전송하여 성공하면 히스토리를 기록한다.")
@Test
void sendMailWithBDD(){
    //given
    BDDMockito.given(mailSendClient.sendEmail(anyString(),anyString(),anyString(),anyString()))
            .willReturn(true);
}
```  

## 정리
`Mockito.when()`이라는게 테스트 대상의 행위를 검증하기위해 준비과정에 일부입니다. 
그래서 `when()`보다 `BDDMockito.given()`이라는 구문이 테스트의 준비과정 일뿐이라는걸 표현해줍니다.  
  
```Java
public class BDDMockito extends Mockito
```  
내부 로직은 Mockito를 그대로 사용하고 `BDD`스타일로 메소드명을 변경했을 뿐이기에 
동일한 방식으로 동작합니다.