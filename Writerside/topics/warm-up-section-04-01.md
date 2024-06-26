# JPA 적용하기
  
## 목표
1. 문자열 SQL을 직접 사용하는 것의 한계를 이해하고, 해결책인 JPA,Hibernate,Spring DataJPA가 무엇인지 이해한다.  
2. Spring Data JPA를 이용해 데이터를 생성,조회,수정,삭제할 수 있다.
3. 트랜잭션이 왜 필요한지 이해하고, 스프링에서 트랜잭션을 제어하는 방법을 익힌다.
4. 영속성 컨택스트와 트랜잭션의 관계를 이해하고, 영속성 컨택스트의 특징을 알아본다.  
  

## SQL을 직접 작성하면 어떤 점이 아쉬울까?
1. 문자열을 작성하기 때문에 실수할 수 있고, 실수를 인지하는 시점이 느리다.  
    컴파일 시점에 발견되지 않고, 런타임 시점에 발견된다.
2. 특정 데이터베이스에 종속적이게 된다.  
3. 반복 작업이 많아진다. 테이블을 하나 만들 때마다 CRUD 쿼리가 항상 필요하다.
4. 데이터베이스의 테이블과 객체는 패더라임이 다르다.  
  
  
## JPA  
**JPA(Java Persistence API)** 라는 건 
자바 진영의 ORM입니다.  
  
`Persistence` 영속성이라는 말은 
메모리(`RAM`)에 저장된건 영속적이지 않다고 표현을 하고, 
디스크(`SSD,HDD`)에 저장된건 데이터가 영구적으로 저장되어 영속적이다 라고 표현합니다.  
  
거기서 말하는 `영속성`을 의미합니다.  
  
> 영속성: 서버가 재시작되어도 **데이터는 영구적으로 저장** 되는 속성  
>  
  
`API`는 클라이언트와 서버간의 정해진 규칙(약속)으로 기능을 제공하는 것을 말합니다.  
  
`JPA`를 정리하면, 자바 진영에서 데이터를 영구적으로 저장할 때 사용하는 API(규칙)입니다.  
  
### 총정리
객체와 관계형 DB의 테이블을 짝지어(`Mapping`)하여 데이터를 영구적(`Persistence`)으로 
저장할 수 있도록 정해진 Java 진영의 규칙(`API`)입니다.  
  
`JPA`는 말 그대로 규칙이기 때문에 Interface라고 할 수 있습니다. 
이 규칙을 실제 코드로 구현한 라이브러리를 `Hibernate`라고 합니다.  
  
`JPA`나 `Mybatis`나 내부로직은 `JDBC`를 사용합니다.