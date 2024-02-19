# 과제 1일차

+ 어노테이션을 사용하는 이유 (효과) 는 무엇일까?
+ 나만의 어노테이션은 어떻게 만들 수 있을까?  
  
## 어노테이션을 사용하는 이유는 무엇일까?  
  
**코드의 가독성을 높일 수 있습니다.**  
`@Configuration`,`@GetMapping`처럼 해당 애노테이션이 붙은 클래스가 무슨 역할을 하는지 
개발자 간의 협업시 소스 코드에 대한 빠른 이해를 돕습니다.  
  
**프레임워크나 라이브러리에 기능을 확장합니다.**  
`@PostContrutor`,`@Transactional`를 소스 코드에 추가하게 되면 프레임워크는 해당 애노테이션을 
참고하여 추가 기능을 동작하게 합니다.  

**제약사항을 추가하여 컴파일 오류로 잡을 수 있습니다.**  
`@Override`,`@FunctionalInterface`처럼 메서드나 클래스에 추가하면 컴파일 시점에 
오류를 발생하게 하여 코드 안정성을 높일 수 있습니다.  
  
## 나만의 어노테이션은 어떻게 만들 수 있을까?  
```Java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME) // 어노테이션 정보를 런타임까지 유지
@Target(ElementType.METHOD)         // 어노테이션을 메소드에 적용
public @interface MyAnnotation {
    String value() default "";      // 어노테이션 속성 정의 (기본값은 빈 문자열)
}
```  
인터페이스로 작성하여 해당 애노테이션 정보와 사용할 수 있는 타입을 지정할 수 있고, 
애노테이션에 값을 받을 수 있습니다.  
  
추가로 애노테이션은 상속의 개념이 없고 메타 애노테이션이라는 개념이 있습니다.  
  
필요에 따라 이미 만들어진 애노테이션을 가져다가 추가하여 코드의 가독성을 높일 수 있습니다.  

```Java
@SpringBootTest
@Transactional
@Rollback(false)
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface NonRollbackTest {}
```  
스프링 컨테이너 환경에서 테스트를 하고, 트랜잭션후에 롤백을 하지 않는다는 애노테이션을 하나의 애노테이션으로 만들 수 있습니다.
