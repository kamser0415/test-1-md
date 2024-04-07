# Context-switching

+ CPU는 프로세스에 있는 여러 Thread들을 돌아가면서 실행하는 스케줄링을 사용함
+ 프로세스나 쓰레드가 중단되었다가 다시 실행 될때 필요한 정보를 Context라고 한다.
  + 프로세스의 Context 중 PCB가 있고 쓰레드는 TCB가 있습니다.
  1. 현재 실행중인 Context를 잠시 중단 및 저장합니다
  2. 새로운 컨택스트를 로딩및 프로세스나 쓰레드를 실행하는 것을 컨택스트 스위칭이라고 합니다.  
  3. 해당 과정중 중요한 것은 CPU Cache를 초기화(`flush`) 합니다.
  4. 다른 쓰레드나 프로세스의 코드를 실행하기때문에 기존 캐시를 날리고 새로운 캐시를 업로드하는 과정이 필요합니다.
  5. 따라서 컨택스트 스위칭이 발생하면 성능이 떨어집니다.  
    

## 발생 이유
1. Sleep:  
   + 프로세스나 쓰레드가 `sleep` 상태로 전환되면 현재 실행중인 프로세스나 쓰레드의 상태를 저장하고, 다음에 실행될 프로세스나 스레드의 상태를 복원하게 됩니다.
2. Lock:
   + 잠금 상태인 임계 영역에 들어갈 때 발생합니다. 해당 쓰레드나 프로세스는 대기 상태가 되어 상태를 저장하고, 잠금을 획득한 쓰레드나 프로세스의 상태를 복원하게 됩니다.
3. I/O:
   + 네트워크 입출력,파일 입출력,콘솔 입출력이 발생하면 해당 작업이 완료될때까지 쓰레드와 프로세스는 대기상태가 되므로 발생합니다.  
4. 시스템 API 호출: 
   + 시스템 호출이 처리될 때까지 대기하므로 발생합니다.