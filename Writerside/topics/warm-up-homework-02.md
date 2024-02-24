# 과제 2일차
  
## 문제 1
두 수를 입력하면, 다음과 같은 결과가 나오는 GET API를 만든다.
path: `/api/v1/calc`  
query: `num1,num2`  
  
응답 형식
```JSON
{
  "add": "덧셈결과",
  "minus": "뺄셈결과",
  "multiply": "곱셈결과"
}
```  
  
```Java
package com.group.libraryapp.controller.homework;

import org.springframework.util.Assert;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HomeworkController {

    @GetMapping("/api/v1/calc")
    public HomeworkResponse getCalculatorResult(
            @RequestParam("num1") Integer num1,
            @RequestParam("num2") Integer num2) {

        Calculator calculator = new Calculator(num1, num2);
        return new HomeworkResponse(calculator.add(), calculator.minus(), calculator.multiply());
    }

    public static class HomeworkResponse {
        private Integer add;
        private Integer minus;
        private Integer multiply;

        public HomeworkResponse(Integer add, Integer minus, Integer multiply) {
            this.add = add;
            this.minus = minus;
            this.multiply = multiply;
        }

        public Integer getAdd() {
            return add;
        }

        public Integer getMinus() {
            return minus;
        }

        public Integer getMultiply() {
            return multiply;
        }
    }

    public static class Calculator {

        private final Integer num1;
        private final Integer num2;

        public Calculator(Integer num1, Integer num2) {
            Assert.notNull(num1, "num1은 null이 허용되지 않습니다.");
            Assert.notNull(num2, "num2은 null이 허용되지 않습니다.");
            this.num1 = num1;
            this.num2 = num2;
        }

        public Integer add() {
            return num1 + num2;
        }

        public Integer minus() {
            return num1 - num2;
        }

        public Integer multiply() {
            return num1 * num2;
        }
    }
}
```   

요청
```HTTP
GET http://localhost:8080/api/v1/calc?num1=10&num2=5
```    

응답 
```HTTP
HTTP/1.1 200 
Content-Type: application/json

{
  "add": 15,
  "minus": 5,
  "multiply": 50
}
```    

문제 1번을 풀다보니 코드를 잘못 작성했다는 걸 느꼈습니다.  
내부 필드로 저장하는것보다 메서드에서 인수로 받아 연산 결과를 출력하는게 
더 좋을 거라 생각이 됩니다.
  
## 문제 2
날짜를 입력하면, 몇 요일인지 알려주는 `GET API`를 만들어보자
path와 쿼리 파라미터는 임의로 만들어도 된다.  

```Java
@GetMapping("/api/v1/day-of-the-week")
public DateResponse getDate(@RequestParam("date") String queryDate) {
    String text = LocalDateUtil.getDayOfTheWeek(queryDate);
    return new DateResponse(text);
}

public static class LocalDateUtil {
    public static String getDayOfTheWeek(String queryDate) {
        LocalDate date = LocalDate.parse(queryDate, DateTimeFormatter.ISO_LOCAL_DATE);
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("ccc", new Locale("US","KR"));
        return date.format(formatter);
    }
}


public static class DateResponse {
    private final String dayOfTheWeek;

    public DateResponse(String dayOfTheWeek) {
        this.dayOfTheWeek = dayOfTheWeek;
    }

    public String getDayOfTheWeek() {
        return dayOfTheWeek;
    }
}
```

요청
```HTTP
GET http://localhost:8080/api/v1/day-of-the-week?date=2024-02-20
```  

응답  
```HTTP
HTTP/1.1 200 
Content-Type: application/json

{
  "dayOfTheWeek": "Tue"
}
```  
  
## 문제 3  
여러 수를 받아 총 합을 반환하는 POST API를 만들기   
```HTTP
{
    "numbers"= [1,2,3,4,5]
}
```  

코드
```Java
@PostMapping("/api/v1/calc")
public Integer getCalcSumAll(@RequestBody CalculatorSumRequest request) {
    Calculator calculator = new Calculator();
    return calculator.add(request.getNumbers());
}
public Integer add(List<Integer> numbers) {
    if (numbers.isEmpty()) {
        return null;
    }
    return numbers.stream().mapToInt(it->it).sum();
}
```
  
요청
```HTTP
POST http://localhost:8080/api/v1/calc
Content-Type: application/json

{
  "numbers": [1,2,3]
}
```  
응답
```HTTP
6
```
  
요청
```HTTP
POST http://localhost:8080/api/v1/calc
Content-Type: application/json

{
  "numbers": []
}
```   
  
응답
```HTTP

```