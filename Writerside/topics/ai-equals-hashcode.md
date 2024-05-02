# equals와 hashcode 메소드의 관계와 각각의 역할
  
## 1차
컴퓨터는 두 데이터를 비교할 때 물리적으로 비교하는 동일성 방식을 사용합니다.
하지만 자바 언어는 객체지향 언어로 객체를 사용하며 객체가 의미적으로 같은지 논리적으로 비교하는 동등성을 사용합니다.  
 
동일성을 비교하는 방법은 == 이며, 동등성을 비교하는 방법은 equals와 hashcode가 있습니다.  
  
equals와 hashcode는 Object 클래스가 가지고 있는 메소드로 모든 클래스는 두 메소드를 가지고 있습니다. 
클래스를 구현할때 두 메소드를 구현해야하는 이유는 논리적으로 비교할때 사용하기 때문입니다.  
  
equals는 객체의 필드를 비교하여 동일한 값을 가지고 있다면 같은 객체로 판단하는 방식이고,
hashcode는 객체의 필드를 해싱하여 숫자 타입의 자료형으로 변환하여 비교하는 방법입니다.  
  
hashcode를 사용하는 이유는 자바 자료구조인 컬렉션에서 데이터를 저장,삭제,조회를 할때 equals 방법보다 int로 비교하는
hashcode가 더 빠르기 때문입니다.  
  
그러면 hashcode만 구현해도 될지 않을까 싶지만 자바는 해쉬를 int로 표현하고, 운영체제와 CPU가 64비트로 데이터를 전달하기 때문에
형변환이 발생하여 데이터 손실이 발생합니다. 다른 객체가 같은 해쉬값을 가지게 되어 해쉬 충돌이 발생하게 되면 성능저하가 발생되고 
오류가 발생하지 않도록 자바에서는 equals를 통해 같은 객체인지 비교하는 방식을 사용합니다.  
  
따라서 hashcode와 euqals는 같은 객체일 경우 같은 값이 반환되어야 자료구조에서 데이터를 저장,삭제,조회할때 오류가 발생하지 않습니다.  
  
## gpt

1. 물리적인 동일성과 논리적인 동등성: 컴퓨터는 두 데이터를 물리적인 측면에서 비교하는 동일성을 사용하지만, 자바는 객체지향 언어로 객체의 의미적 동등성을 확인하기 위해 논리적인 동등성을 사용합니다.
2. 동일성과 동등성 비교 방법: 동일성을 비교하는 방법은 == 연산자를 사용하고, 동등성을 비교하는 방법은 equals()와 hashCode() 메소드를 사용합니다.
3. equals와 hashCode의 역할: equals는 객체의 필드를 비교하여 논리적으로 같은지 확인하는 역할을 하고, hashCode는 객체의 필드를 해싱하여 숫자로 변환하여 빠르게 비교하는 역할을 합니다.
4. equals와 hashCode의 필요성: 자바에서 모든 클래스는 Object 클래스를 상속하며, Object 클래스에는 equals와 hashCode가 정의되어 있습니다. 이를 재정의하여 논리적 동등성을 확인하고, 컬렉션 등 자료구조에서 올바른 동작을 보장하기 위해 두 메소드를 구현해야 합니다.
5. hashcode와 equals의 관계: hashCode만 구현하여 논리적 동등성을 보장할 수 있을 것 같지만, 해시 충돌이 발생할 수 있으며, 이를 방지하기 위해 equals로 객체의 논리적 동등성을 다시 확인합니다. 이러한 이유로 equals와 hashCode는 같은 객체에 대해 같은 결과를 반환해야 합니다.