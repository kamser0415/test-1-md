# 조건문
  
## 학습목적
조건문은 내가 원하는 조건에 일치하는 경우 특정 코드를 추가 수행하길 원할 때 사용한다.  
  
```Java
if(boolean 을 반환하는 조건식) {
    // 추가로 실행해야하는 코드
}
```  
자바 코드는 개행에 상관없이 세미 콜론이 나오면 그때 한줄로 인식한다.
```Java
public static void main(String[] args) {
    if (true)




            
        System.out.println("IfTest.main");
}
```  
이거도 실행이된다.  
  
특정 조건일때에는 여러 조건도 사용할 수 있습니다.  
  
논리식으로 `A && B || C` 최종 결과에 따라 달라질 수도 있습니다.  
하지만 이렇게 작성하면 가독성이 나빠지므로 먼저 실행되어야할 비교를 소괄호로 표현합니다.  
`A && ( B || C )`로 이렇게 우선순위를 표시하는게 가독성과 유지보수가 좋습니다.  
  
그리고 하나의 변수에 다양한 값에 따라 다른 결과를 실행하고 싶은 경우가 있습니다.  
+ `나이가 20세 이하`는 청소년
+ `나이가 20세 초과 34세 이하`는 청년
+ `나이가 34세 초과 65세 미만`은 중장년
+ `나이가 65세 이상`은 노년  
  
이렇게 추가 로직을 수행하고 싶을 때에는 `if else`를 사용할 수 있습니다.  
`if else`는 논리 표현식에서 `&&`와 똑같습니다.  
  
예를 들면 
```Java
if(age <= 20) {
    // 청소년
} else if (age <= 34){
    // 청년
} else if (age < 65) {
    // 중장년
} else {
    // 노년
}
```   
이 코드를 논리식으로 표현하면  
`중장년`을 표현해보면 `age > 20 && age > 34 && age < 65` 으로 나타나게 되므로 
코드를 이해하기 위해서는 else if만 읽는게 아니라 앞뒤 코드까지 해석해야하는 단점이 있습니다.  

`if-else`는 `if`문으로만 작성하면 불필요한 조건에 대한 실행 
코드의 효휼성이 높아집니다. 이미 앞에서 검증하고 실패한 값이기 때문이다.
 
+ if 조건식 문장은 `{}`을 통해서 스코프를 제한하여 지역변수를 사용할 수 있습니다.
  
`|| &&` 차이  
1. || 는 조건식에 true가 나오면 종료가 됩니다.  
   + 조건에 true가 나올게 많다면 먼저 실행하도록 배치한다.
2. && 는 조건식에 false가 나오면 종료가 됩니다.  
   + 조건에 false가 나올수 있는게 많다면 먼저 실행하도록 한다.  
  
## switch - case  
하나의 변수가 여러 조건에 따라 다르게 동작할때 사용한다.  
```Java
switch(grade) {
    case "A" : XXX
        break;
    case "B" : yyy
        break;
    default: zzz
}  
```  
  
그런데 default는 중간에 들어오면 원하지 않게 동작합니다.  
이유는 switch는 조건에 해당하는 case가 있을 경우 그 이하 작성된 case는 조건에 일치하지 않아도 
추가 코드가 실행됩니다. default가 중간에 있고 break가 없다면 아래와 같은 일이 발생합니다.
```Java
public void name(String name) {
    switch (name) {
        case "A":
            System.out.println("name = " + name);
            break;
        case "B":
            System.out.println("name2 = " + name);
            break;
        default:
            System.out.println("default = " + name);
        case "V":
            System.out.println("nameV = " + name);
            break;
    }
}
```  
```Java
name("VC");
default = VC
nameV = VC
```  
case "V"에 일치하지 않아도 적용되는 것을 볼 수 있습니다.  
 
사용할 수 있는 건 `ENUM,Character,Byte,Short,Integer,String`외 `long`을 제외한 기본 자료형만 사용할 수 있습니다.  
  

