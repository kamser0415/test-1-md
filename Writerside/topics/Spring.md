# Spring

## EJB  
**EJB (Enterprise Java Bean)** 은 기업 환경의 시스템을 구현하기 위한 서버측 컴포넌트 모델이다.  
  
**장점**  
+ 정형화된 비즈니스 계층을 제공하여 논리적,물리적 분리는 지원한다
+ 선언적인 트랜잭션 관리와 다양한 클라이언트 지원이 가능하다.
+ 분산 기능을 지원하여 비즈니스 객체를 여러 서버에 분산시킬 수 있다.
  
**단점**  
+ EJB가 제공하는 컨테이너에 종속적이게 된다.
+ 테스트에 어려움이 있다.  
+ 실행속도가 느리다.  
  
## POJO
+ 실행속도가 느리다보니 EJB를 위한 변형된 패턴들이 나타나면서 객체지향적인 코드에 벗어나게됨
+ EJB를 사용하는 대형 벤더들이 자신만의 기술을 추가하게되었고 EJB 컨테이너 사이에 이식성이 떨어짐  
    
특정 기술에 의존적이지 않고 객체지향적인 코드 작성과 자바 코드의 높은 이식성을 구현할 수 있는 순수 Java 기술로 코드를 개발하는 방식이 등장함  
  
## 기술
+ **핵심 기술**: 스프링 DI 컨테이너, AOP, 이벤트
+ **웹 기술**: 스프링 MVC, 스프링 WebFlux
+ **데이터 접근 기술**: 트랜잭션, JDBC,ORM 지원, XML 지원
+ **기술 통합**: 캐시, 이메일, 원격접근, 스케줄링
+ **테스트**: 스프링 기반 테스트 지원  
  
## 확대  
+ 스프링 시큐어리티
+ 스프링 클라우드
+ 스프링 배치  

**스프링과 그 생태계가 점점 커짐**  
+ 다양한 오픈 소스의 등장으로 수 많은 라이브러리를 함께 사용해야함
+ 스프링으로 프로젝트를 시작할 때 필요한 설정이 점점 늘어남
+ 스프링으로 프로젝트를 시작하는 것이 점점 어려워짐
+ 시작도 하기 전에 복잡한 설정 때문에 포기  
  
스프링을 실행하기 위해 라이브러리 버전 관리와 설정, 그리고 스프링 컨테이너에 빈으로 등록해야했습니다.  
새 프로젝트를 시작할 때 반복적인 라이브러리 설정, 빈 등록을 반복적으로 사용하게 됩니다.  
  
  
## 정리  
스프링 프레임워크는 EJB와 같은 기존의 기술들로 인해 발생한 문제였던 EJB에 의존한 코드 작성으로 인한 객체지향적이지 못한 코드 작성과 이식성이 낮은 문제를 해결하기 위해 등장했습니다.  
스프링 프레임워크는 해결 방안으로 POJO를 기반으로 컨테이너를 제공하고 객체지향적인 코드 작성을 돕기 위해 `DI`와 `AOP`등을 활용하여 코드의 분리와 재사용성을 높여줍니다.  