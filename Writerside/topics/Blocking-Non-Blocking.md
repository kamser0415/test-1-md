# Blocking-Non-Blocking

## Blocking  
+ 함수를 실행하고 모든 코드가 완료된 후 리턴되면 Blocking 입니다.


## Non-Blocking
+ 실행한 함수의 코드의 완료되지 않고 리턴되면 Non-Blocking  
+ Non-Blocking 함수를 실행하고 완료됨을 어떻게 알 수 있을까?
+ Polling & Event


## Polling  
```Java
where(true){
    if(isFinish == true){
        Break;
    }
    sleep(1000);
}
```  
주기적으로 요청이 들어온지 확인합니다.  
sleep을 입력하여 현재 쓰레드에 할당된 CPU를 잠시 넘깁니다.  
  
  
## @Event
```Java
setTimeout(callback, 1000);

function callBack(){
    ...
}
```  
Event로 완료됨을 보고할 수 있습니다.  
  
콜백 함수에게 실행할 메서드를 넘기면 실행이 됩니다.  
콜백을 무리해서 사용하여 콜백지옥이 되는 경우가 있어서 아래와 같은 방식을 사용합니다.  
  
## async & await  
```Java
main(){
    String result = await getName();
    System.out.println(result);
}

async getName(){
    //...
}
```
호출한 함수 main은 await가 없다면 응답을 기다리지 않고 아래 메서드를 실행하지만, 
await를 실행하면 result에 결과값이 담기기 전까지 대기합니다.  
  
