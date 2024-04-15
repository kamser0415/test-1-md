# Balking 패턴  
  
[코드 출처](https://github.com/myc0058/multi-thread.git)  
  
## 정의 
+ 내가 해야될 작업이 있는지 주기적으로 확인합니다.  
+ 작업이 있으면 실행하고 없으면 넘어갑니다.  
  
먼저 멀티쓰레드 환경에서 해당 쓰레드가 잠금을 획득하지 못한 경우에 대기하지 않고 종료하거나 주기적으로 확인하는 것을 말합니다.
## 코드  
### 공유객체 
```Java
public class BalkingData {
    private String filename;
    private String content;
    private boolean changed;

    public BalkingData(String filename, String content) {
        this.filename = filename;
        this.content = content;
        this.changed = true;
    }

    public synchronized void change(String newContent) {
        content = newContent;
        changed = true;
    }

    public synchronized void save() throws IOException {
        if (!changed) {
            return; // balking
        }
        doSave();
        changed = false;
    }

    private void doSave() throws IOException {
        System.out.println(Thread.currentThread().getName() + " calls doSave, content = " + content);
    }
}
```
```Java
if (!changed) {
    return; // balking
}
```
`save()` 메서드 내용을 보면 만약 `changed` 공유 변수를 확인하고 만약 변경이 안되었다면, 바로 포기하기하는 것을 말합니다.  

```Java
public class SaverThread extends Thread {
    private BalkingData data;

    public SaverThread(String name, BalkingData data) {
        super(name);
        this.data = data;
    }

    @Override
    public void run() {
        try {
            while (true) {
                data.save();
                Thread.sleep(1000);
            }
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
        }
    }
}
public class ChangerThread extends Thread {
    private BalkingData data;

    public ChangerThread(String name, BalkingData data) {
        super(name);
        this.data = data;
    }

    @Override
    public void run() {
        for (int i = 0; true; i++) {
            data.change("No." + i);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```  
### 실행 코드
```Java
public class Main {
    public static void main(String[] args) {
        BalkingData balkingData = new BalkingData("data.txt", "(empty)");
        new SaverThread("SaverThread", balkingData).start();
        new ChangerThread("ChangerThread", balkingData).start();
    }
}    
```  

`SaverThread`는 주기적으로 데이터를 저장하려고 하며 만약 잠금을 획득하지 못한 경우에는 포기하고 다른 작업을 할 수 있습니다.
```Java
public synchronized boolean save() throws IOException {
    if (!changed) {
        return false; // balking
    }
    doSave();
    changed = false;
    return true;
}
```  
`save`작업을 실패할 경우에 횟수를 지정하여 아예 포기하거나, 메세지를 출력하는 작업을 실행할 수 있습니다.
```Java
@Override
public void run() {
    try {
        int cnt = 0
        while (true) {
            if(data.save() == false){
                cnt++;
            } else {
                cnt=0;
            }
            if(cnt == 5){
                System.out.println("데이터 저장 5회 실패했습니다");
                // 메세지 전송으로 개발자에게 전달 등..
                return;
            }
            Thread.sleep(1000);
        }
    } catch (IOException | InterruptedException e) {
        e.printStackTrace();
    }
}
```  
  
## 장점  
쓰레드가 공회전도는 동안 작업에 대한 예외상황이나 추가 작업이 가능한 패턴입니다.  
  
## GuardedSuspension과 비교  
+ GuardedSuspension
  + 작업이 없으면 클라이언트가 작업을 추가하고, `notify`하기 전까지 대기상태로 머무른다.
  + 그러다보니 쓰레드 낭비가 적다.
+ Balking 
  + 작업이 없으면 클라이언트가 작업을 추가하기 전까지 그외 작업이 가능하다.
  + 주기적으로 작업물을 확인하는 방식으로 작업 외에 다른 작업도 가능합니다.
