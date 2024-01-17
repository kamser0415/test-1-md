# Mock-정리

Test Double, Stubbing
: dummy, fake, stub, spy, mock   

@Mock,@MockBean,@Spy,@SpyBean,@InjectMocks
: 스프링 없이 단위테스트를 할때에는 @Spy,@Mock,@InjectMocks를 사용  

BDDMockito
: Mockio의 하위 클래스로 BDD 메서드명을 사용할 수 있다.  

Classist vs Mockist
: Classist - 우리 시스템 경계는 모킹을 자제하자.
Mockist - 이미 완료된 테스트는 모킹으로 시간 절약하자.