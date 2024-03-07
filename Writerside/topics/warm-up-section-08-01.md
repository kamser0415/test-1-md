# 생애 최초 배포하기
  
## 목표  
1. 클라우드 컴퓨터에 접속하여 리눅스 명령어를 다뤄본다.  
2. 개발한 서버의 배포를 위해 환경 세팅을 리눅스에서 진행하고, 실제 배포를 진행한다.
3. foreground와 background의 차이를 이해하고 background 서버를 제어한다.
4. 도메인 이름을 사용해 사용자가 IP대신 이름으로 접속할 수 있도록 한다.  
  
## 리눅스 기본 명령어
+ mkdir
+ ls
+ cd: change directory
+ pwd: print working directory
+ rmdir: 비어있는 폴더(디렉토리)를 제거하는 명령어

```Shell
total 5108
-rw-r--r-- 1 you         you    1336 Mar  3 04:20 000-default.conf
drwxr-xr-x 5 you         you    4096 Mar  2 09:20 2021_DEV_HTML
```  
### drwxr-xr-x 의미
**d** : 폴더를 의미합니다.  
d**rwx**r-xr-x  
+ r: 폴더를 읽을 수 있는 권한
+ w: 쓸 수 있는 권한 
+ x: 실행할 수 있는 권한  
  
자세히 보면 `rwx`가 3번 반복되는 것을 확인할 수 있습니다.   
폴더 소유자의 권한 **/** 폴더 소유그룹의 권한 **/** 아무나 접근했을 때의 권한  
  
폴더 소유자와 폴더 소유 그릅이란 계정은 소유자고 묶어서 관리하면 폴더 소유그룹이라고 합니다.  
  
### drwxr-xr-x 5 숫자의 의미
폴더에 걸려있는 바로가기 개수를 의미합니다.  

you         you      4096   Mar  2 09:20   2021_DEV_HTML  
폴더 소유자 | 소유그룹 |  크기  | 수정일       | 이름  
    

## 배포를 위해 필요한 프로그램
1. git - 코드를 가져오기 위해 필요
2. java 설치 - 우리가 작성한 코드를 실행할 수 있는 JVM이 필요
3. mysql - 데이터베이스의 역할을 수행함.  
  
## 사양이 낮기 때문에 Swap 설정 추가하기  
램이 부족하면 디스크를 사용하여 한다.

```Shell
# swap 메모리를 할당한다 ( 128M * 16 = 2GB )
20240224-071346:~/my-library$ sudo dd if=/dev/zero of=/swapfile bs=128M count=16
16+0 records in
16+0 records out
2147483648 bytes (2.1 GB, 2.0 GiB) copied, 20.1688 s, 106 MB/s

# 스왑 파일에 대한 권한 업데이트
20240224-071346:~/my-library$ sudo chmod 600 /swapfile

# Swap 영역 설정
20240224-071346:~/my-library$ sudo mkswap /swapfile
Setting up swapspace version 1, size = 2 GiB (2147479552 bytes)
no label, UUID=a8ed7cfa-9023-4af7-9c6e-c420f6502831

# swap 파일을 사용할 수 있도록 한다.
20240224-071346:~/my-library$ sudo swapon /swapfile

# swap 성공 확인
20240224-071346:~/my-library$ sudo swapon -s
Filename                                Type            Size    Used    Priority
/swapfile                               file            2097148 1792    -2
```  


## foreground vs background

