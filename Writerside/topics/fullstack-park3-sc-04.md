# 참고: 도커를 위한 리눅스 사용법 요약

본 강의는 도커 강좌이므로, 도커 작업을 위해 필요한 핵심 기능을 익히는데 목적  
  
## 리눅스와 파일
  
### 모든 것은 파일이라는 철학을 따름  
하드웨어같은 것도 파일이라고 처리합니다.  
파일이라는건 읽고 쓰고 할수 있다는 의미로 모든 하드웨어를 파일처럼 다룹니다.  

모든 인터렉션은 파일을 읽고, 쓰는 것처럼 이루어져있음  
마우스,키보드와 같은 모든 디바이스 관련된 기술도 파일과 같이 다루어짐  
  
### 파일 네임 스페이스
A 드라이브 (`A:/`),C 드라이브 (`C:/User/kms`) (x)  
전역 네임스페이스 사용  
+ `/media/floofy/dave.jpg`  
  
리눅스의 경우 `/`을 루트 디렉토리라고 합니다. 
그래서 루트 디렉토리부터 이렇게 디렉토리/디렉토리/파일  
  
전체파일이 `/` 루트 리렉토리 하위에 있게 됩니다 `vue #root 처럼`  
  
## 쉘 종류   
`shell`: 사용자와 컴퓨터 하드웨어 또는 운영체제간 인터페스  

운영체제는 하드웨어를 제어하는 프로그램입니다. 컴퓨터에서 제일 중요한 프로그램으로 
사용자에게 명령을 받아야하는데 그 인터페이스가 없습니다.  
  
그래서 그 인터페이스를 별도로 프로그램으로 만들어 운영체제에서 제공합니다.  
  
그것을 바로 `shell`이라고 합니다. 윈도우에서는 `powerShell`이 있습니다.  

마우스로 클릭으로 입력을 받아 명령을 처리할 수 있는데 이걸 GUI라고 합니다.  
이것도 일종의 shell 이라고 할수 있습니다.  
  
터미널에서도 키보드로 입력받는 것을 `CLI`라고 하고 `CLI shell`은 이름이 있습니다.  
여러가지 `cli shell`프로그램이 있습니다.  
  
**리눅스의 대표적인 shell**  
1. Bourne-Again Shell(`Bash`): GNU 프로젝트의 일환으로 개발됨, 리눅스 거의 디폴트
2. Bourne Shell(`sh`)
3. C Shell(`csh`)  
4. Korn Shell(`ksh`): 유닉스에서 가장 많이 사용됨  
  
**리눅스 기본 명령어 정리**  
+ 리눅스 명령어는 결국 쉘이 제공하는 명령어
+ 리눅스 기본 쉘이 bash 이름로, bash에서 제공하는 기본 명령어를 배우는 것.  
  
리눅스나 유닉스 계열은 컴퓨터 한대를 놓고 여러명이 로그인을 해서 
동시에 작업하는 것에 특화된 운영체제 입니다.  
  
다중사용 운영체제라고 할 수 있습니다.  
  
그렇기 때문에 로그인을 해서 들어가면 자신의 아이디가 있습니다.  
```Bash
whoami # 현재 로그인한 사용자 이름
```  
root: 슈퍼관리자 ID가 있습니다.  
다중 사용자가 사용하는 시스템으로 root 아이디는 모든 작업이 경고없이 처리되기 때문에 
관리자 계정이 아니라 일반 아이디로 관리자 권한을 요청해서 사용합니다.  
  
```Bash
sudo apt-get update
```  
우리는 리눅스에 다양한 패키지가 있고 현재는 우분투 패키지를 사용합니다.  
우분투 패지키에서 업데이트가 되면 아카이브에 저장이되고, 위 명령어처럼 업데이트 된 내용을 패치받으면 
최신 정보를 받아올 수 있습니다.  
  
업데이트를 하는데 관리자 명령인 `sudo`가 필요한 이유는 
시스템 전체에 영향을 주는 명령이기 때문에 관리자 권한이 필요하기 때문입니다.  
   
현재위치를 확인한다.
```Bash
pwd
```   
모든 파일 목록을 확인할 수 있습니다.
```Bash
ls -al

drwxr-xr-x 4 yousd179 yousd179 4096 Feb 24 07:42 .
drwxr-xr-x 4 root     root     4096 Feb 24 07:21 ..
```  
+ ` yousd179 . `는 현재 디렉토리를 의미합니다.
+ ` root .. `는 상위 디렉토리를 의미합니다.  
   
  
### 리눅스와 권한
+ 운영체제는 사용자/리소스 권한을 관리
+ 리눅스는 사용자/그룹으로 권한을 관리
+ root는 슈퍼관리자
+ 파일마다 소유자,소유자 그룹 - 모든 사용자에 대해 읽고,쓰고,실행하는 권한을 관리합니다.  
  + 권한을 세가지로 나누어 관리합니다.
```Bash
권한           소유자    소유 그룹  크기 수정시간        이름
drwxr-xr-x 4 yousd179 yousd179 4096 Feb 24 07:42 .
drwxr-xr-x 4 root     root     4096 Feb 24 07:21 ..
-rw------- 1 yousd179 yousd179   16 Feb 24 07:42 .bash_history
-rw-r--r-- 1 yousd179 yousd179  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 yousd179 yousd179 3771 Feb 25  2020 .bashrc
drwx------ 2 yousd179 yousd179 4096 Feb 24 07:41 .cache
-rw-r--r-- 1 yousd179 yousd179  807 Feb 25  2020 .profile
drwx------ 2 yousd179 yousd179 4096 Feb 24 07:34 .ssh
```  
맨 앞에있는 알파벳 `d`는 폴더를 의미합니다.  
그외에 다양한 알파벳이 맨 앞자리에 붙습니다.  
  
  
## 리다이렉션과 파이프  
유닉스 계열에서 어떤 명령은 항상 3가지의 기본적인 스트림을 가지고 있다고 합니다.  
  
command로 실행되는 프로세스는 세 가지 스트림을 가지고 있음  
+ 표준 입력 스트림(Standard Input Strean) - `stdin`  
+ 표준 출력 스트림(Standard Output Strean) - `stdout`  
+ 오류 출력 스트림(Standard Error Strean) - `stderr`  
  
**모든 스트림은 일반적인 plain text로 console에 출력하도록 되어 있음**  
  
예를 들어 `ls -al` 명령어를 입력했습니다.
```Bash
ls -al

drwxr-xr-x 4 yousd179 yousd179 4096 Feb 24 07:42 .
drwxr-xr-x 4 root     root     4096 Feb 24 07:21 ..
   :
```  
리눅스의 기본적인 출력 스트림은 화면에 출력되도록 기능이 만들어졌습니다.  
```Bash
drwxr-xr-x 4 yousd179 yousd179 4096 Feb 24 07:42 .
drwxr-xr-x 4 root     root     4096 Feb 24 07:21 ..
   :  
```  
이 부분 입니다.   
  
그리고 입력 스트림은 무엇이냐면, `-al` 이 해당하게 됩니다.  
`-al`이라는 옵션이 `ls` 명령의 입력으로 들어가게 됩니다.  
  
에러는 기본적으로 출력 스트림과 동일하게 화면에 출력하도록 기능이 만들어져있습니다.  
  
그러면 각 스트림을 다룰 수 있는 방법들이 기본적으로 설정되어있습니다.  
  
이 방법을 바꿔주는 기능이 **_리다이렉션과 파이프_** 입니다.  
  
스트림은 물줄기로 표현하는데 
출력 스트림은 기본적으로 화면에 출력되는 흐름인데 
그 사이에 구멍을 `파이프`를 통해서 다른 곳에 출력되도록 하는 것을 
`리다이렉션과 파이프`라고 생각하면 됩니다.  
  
### 리다이렉션(redirection)  
표준 스트림 흐름을 바꿀 수 있습니다.  
+ `>`,`<` 을 사용합니다.  
+ 주로 명령어 표준 출력을 화면이 아닌 파일에 쓸 때 사용합니다.  
  
```Bash
yousd179@instance-20240224-071346:~$ ls -al > test.txt

yousd179@instance-20240224-071346:~$ ls
test.txt

yousd179@instance-20240224-071346:~$ cat test.txt
total 32
drwxr-xr-x 4 yousd179 yousd179 4096 Feb 24 10:04 .
drwxr-xr-x 4 root     root     4096 Feb 24 07:21 ..
    : (이하생략)
```  
  
`>`,`<`은 덮어쓰기 입니다. 기존에 파일에 내용이 있으면 모두 없애고 새 출력 스트림 데이터가 들어갑니다.  
  
+ `<<`,`>>`는 기존 데이터에 추가합니다.  
```Bash
ls -al >> test.txt 
```  
  
**도커를 실행하고나서 특정 결과를 파일에 저장할 때 사용합니다**  
  
### 파이프(pipe)  
  
```Bash
ls -al | grep bash

-rw------- 1 yousd179 yousd179   16 Feb 24 07:42 .bash_history
-rw-r--r-- 1 yousd179 yousd179  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 yousd179 yousd179 3771 Feb 25  2020 .bashrc
```  
  
`ls -al`의 결과가 다시 입력 스트림이 되어서 `grep bash`로 해당 라인중에서 
`bash`가 들어간 라인을 출력해주는 명령어가 됩니다.  
   
```Bash
yousd179@instance-20240224-071346:~$ ps aux | grep bash
yousd179    1862  0.0  0.5  10172  5312 pts/0    Ss   08:46   0:00 -bash
yousd179    2536  0.0  0.0   8168   656 pts/0    R+   10:26   0:00 grep --color=auto bash
```  
리눅스 서버에서 실행되는 프로그램 중에서 `ps aux`을 출력한 결과물 중에서 `bash`가 들어간 라인을 출력합니다.  
  

### 참고: grep 명령어
grep : 검색 명령어  
```Bash
- grep [-option] [pattern] [file or directory name]
```  
```Bash
< option >  
-i : 영문의 대소문자를 구별하지 않는다.  
-v : pattern을 포함하지 않는 라인을 출력한다.  
-n : 검색 결과의 각 행의 선두에 행 번호를 넣는다 (first line is 1).
-l : 파일명만 출력한다.
-c : 패턴과 일치하는 라인의 개수만 출력한다.
-r : 하위 디렉토리까지 검색한다.
```  
  
## 프로세스 관리  
  
### 프로세스 vs 바이너리  
+ 코드 이미지 또는 바이너리 실행파일 
+ 실행 중인 프로그램: 프로세스  
  + 가상 메모리 및 물리 저장소 정보
  + 시스템 리소스 관련 정보
  + 스케줄링 단위  

컴파일러 언어(Compiled Language)라고 합니다. 컴파일러 언어는 코드를 사전에 컴파일하여 기계어로 번역한 후 실행 가능한 파일을 생성합니다.  
그래서 c언어를 윈도우 환경에 맞게 컴파일을 하고 빌드를 하면 exe 파일이 되는데 내부에는 `0101111`처럼 해당 운영체제가 이해할 수 있는 기계어로 되어있습니다.  
  
이 `exe`파일을 실행하면 운영체제가 프로세스라는 것으로 바꿔서 실행하게 됩니다.  
바이너리에 있는 일정 코드 부분을 메모리(`RAM`)에 넣고 그 다음에 운영체제가 이 메모리에 있는 코드를 
운영체제가 정의한 포멧과 단위에 따라서 실행하면서 해당 응용 프로그램이 실행하게 됩니다.  
  
### 리눅스는 다양한 프로세스 실행 환경  
리눅스는 기본적으로 다양한 프로세스가 실행됩니다.  
유닉스 철학은 여러 프로그램이 서로 유기적으로 각자의 일을 수행하면서 전체 시스템이 동작하도록 하는 모델입니다.  

이 철학의 핵심은 작고 잘 정의된 프로그램들이 각각 특정한 일을 수행하면서, 이러한 프로그램들을 결합하여 전체 시스템을 구축하는 것입니다. 이런 방식으로 시스템을 구성함으로써, 각 프로그램은 명확하고 단순하며 잘 정의된 역할을 수행하게 됩니다.  
  
```Bash
ls -al | grep -c bash
```  
  
이 명령어를 보더라도 ls 명령과 grep이 유기적으로 동작하여 파일 리스트 중에서 특정 파일을 찾는 기능이 됩니다.  
  
각각의 기능을 만들고 유기적으로 연결하여 원하는 기능으로 사용할 수 있도록 하는 철학이 있습니다.  
  
### foreground process / background process
+ foreground process; 쉘에서 해당 프로세스 실행을 명령한 후, 해당 프로세스 수행 종료까지 사용자가 다른 입력을 하지 못하는 프로세스
+ background process; 사용자 입력과 상관없이 실행되는 프로세스 
  + 쉘(shell)에서 해당 프로세스 실행시, 맨 뒤에 & 를 붙여줌
  + 사용 예:  
    ```Bash
    find / -name '*.py' > list.txt &
    [1] 2568
    ```  
    `[1]`은 작업 번호 (job number), 57은 `pid(process id)`를 의미합니다.  
    ctrl + c를 누르면 해당 프로세스가 종료됩니다.

  
## 프로세스 상태 확인 -ps 명령어  
+ 사용법 : ps `[option(s)]`  
+ option(s)  
```Bash
:~$ ps
    PID TTY          TIME CMD
   1862 pts/0    00:00:00 bash
   2575 pts/0    00:00:00 ps
```  
단순한 `ps`는 해당 사용자가 직접 실행한 프로세스를 보여줍니다.  
```Bash
1. -e: 시스템 전체 프로세스를 보여줍니다.
2. -f: 전체 포맷으로 프로세스 정보를 보여줍니다.
3. -l: 긴 포맷으로 프로세스 정보를 보여줍니다.
4. -u user: 특정 사용자가 실행한 프로세스만 표시합니다.
5. -p pid: 특정 프로세스 ID에 해당하는 프로세스 정보만 표시합니다.
6. -a: 다른 사용자의 프로세스도 표시합니다.
7. -x: 데몬과 사용자가 같은 이름을 사용하는 프로세스도 표시합니다.
8. -o format: 특정 포맷으로 출력을 지정합니다.
9. --forest: 프로세스의 부모-자식 관계를 트리 형태로 보여줍니다.
```  

```Bash
ps aux ( 모든 프로세스, 데몬 프로세스 표시)
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  1.3 103884 12892 ?        Ss   07:19   0:02 /sbin/init
root           2  0.0  0.0      0     0 ?        S    07:19   0:00 [kthreadd]
  :
```  
  
## 프로세스 중지시키기
+ `kill` 명령어  
+ 사용법
    ```Bash
    1. kill % 작업번호(job) number
    2. kill 프로세스 ID(pid)
    3. 작업 강제 종료 옵션 -9
    kill -9 57
    강제로 죽일때 -9 옵션을 사용합니다.
    ```  
  
## 하드링크와 소프트링크   
웹 서버 같은 프로그램에 설정을 해야할때 사용합니다.

**cp 명령: 파일 복사**  
+ 1MB 사이즈를 가지고 있는 A파일을 B파일로 복사
```Bash
yousd179@instance-20240224-071346:~$ cp list.txt listcp.txt
yousd179@instance-20240224-071346:~$ ls -al
-rw-rw-r-- 1 yousd179 yousd179 1723753 Feb 24 11:07 list.txt
-rw-rw-r-- 1 yousd179 yousd179 1723753 Feb 24 11:11 listcp.txt
```  

**하드 링크: In A B**  
+ A와 B는 동일한 10MB 파일을 가리킴
+ 즉 동일한 파일을 가진 이름을 하나 더 만드는 것일 뿐
+ 전체 파일 용량은 달라지지 않음
+ 만약 A를 지운다고해도 B파일은 남아있음
```Bash
ln list.txt list_in.txt
```  
  
**소프트(심볼릭) 링크** : `ln -s A B`  
+ Window OS의 바로가기와 동일 
+ `ls -al`하면, 소프트 링크 확인 가능  
```Bash
yousd179@instance-20240224-071346:~$ ln -s list.txt list_s.txt
yousd179@instance-20240224-071346:~$ ls -al 

-rw-rw-r-- 2 yousd179 yousd179 1723753 Feb 24 11:07 list.txt
lrwxrwxrwx 1 yousd179 yousd179       8 Feb 24 11:20 list_s.txt -> list.txt
```
+ rm A로 A를 삭제하면 B는 해당 파일 접근 불가  

소프트 링크를 사용하는 방식:  
웹 서버 설정 등에서 이렇게 소프트 링크를 사용하여 원본 파일은 다른 디렉토리에 있고 
해당 소프트 링크를 가진 어떤 파일명을 특정 폴더에 넣어서 웹서버를 추가로 설정하는 경우도 꽤 있습니다.  

소프트 링크를 사용하여 웹 서버를 설정하면, 실제 파일이 위치한 디렉토리 구조를 변경하거나 파일을 업데이트해도 웹 서버 설정을 다시 작성할 필요가 없으며, 유연하게 관리할 수 있습니다.
  
## 우분투 패키지 관리  
+ 다양한 배포판 중 하나
+ 데비안 배포판을 기반으로 캐노니컬 사가 우분투 배포판 개발
  + 데비안 배포판은 apt 프로그램을 이용해서 소프트웨어 설치 및 업데이트를 간편하게 한 패키지
+ 우분투 의미: 남아프리카 부족 언어로 `너가 있으니 나도 있다`는 의미
  + 우분투 데스크탑 배보판 (X 윈도우 기반, GUI 환경 기본 제공)과 우분투 서버 배포판, 두가지 기본 배포판을 제공
  + 지원 기간이 짧은 일반 버전과 지원 기간이 장기(5년)인 LTS(Long Term Support) 버전으로 나눠서 발표  
    
배포판에 대해서 백그라운드를 이해를 해보면 원래는 이제 리눅스 운영체제는 리눅스 커널이라는 핵심 운영체제 프로그램과 함께 여기에 사용자에서 인터페이스를 받을 수 있는 사용자 명령을 받을 수 있는 `bash`가 있습니다.  
```Bash
------------
bash, 그외 기타 프로세스
------------
리눅스 커널
------------
하드웨어
```  
이 구조를 어떻게 만드냐에 따라 여러가지 배포판이 생긴다고 할 수 있습니다.  
`JDK`도 만드는 회사마다 다양한 `Java` 종류가 있는거라고 생각됩니다.  
  
## ubuntu 패키지 관리자
+ CentOS 나 Fedora 과 같은 RedHat 계열 배포판은 RPM(Red Hat Package Manager) 이라는 패키징 시스템을 사용
+ ubuntu와 같이 데비안 계열 배포판은 deb(Debian Package) 라는 패키징 시스템을 사용함
+ 패키지와 패키지 정보를 저장하고 있는 패키지 저장소라는 개념을 가지고 있음
+ 소프트웨어 패치, 추가등 정보를 관리
+ 우분투 사용자가 패키지 관리자를 통해 패키지 장소에 접근하면, 소프트 웨어 변경사항을 알려주고 업데이트, 다운로드를 지원  
  
### 패키지 관리 실무
우분투 패키지는 `apt` 명령어를 사용하여 패키지를 관리합니다.

- 우분투 패키지 인덱스 정보 업데이트:
    ```bash
    sudo apt-get update
    ```
  이 명령어를 실행하면 시스템에 설치된 패키지 목록과 각 패키지의 버전 정보 등이 포함된 패키지 인덱스가 업데이트됩니다. 이 과정은 우분투 소프트웨어 저장소에서 최신 패키지 정보를 가져와 로컬 시스템에 반영하는 것입니다.

- **소프트웨어 저장소**:
  프로그램이 저장된 위치는 우분투 소프트웨어 저장소(Repository)입니다. 우분투는 기본 저장소 외에도 다양한 공식 및 비공식 저장소를 제공합니다. `apt-get update` 명령어를 사용하여 이 저장소에서 패키지 정보를 업데이트합니다.

- **패키지 정보 관리**:
  패키지 정보는 각 패키지의 버전 및 의존성 등을 포함하고 있습니다. 따라서 패키지 정보를 업데이트하는 것은 새로운 패키지가 추가되거나 기존 패키지의 버전이 변경되었을 때 필요합니다.

- **패키지 관리자(Apt)**:
  `apt-get`은 우분투의 패키지 관리자 중 하나입니다. `apt` 명령어는 `apt-get`의 간소화된 버전으로, 더 직관적이고 사용하기 쉬운 명령어입니다. 최근 우분투 버전에서는 `apt` 명령어를 권장하고 있습니다.
  


+ 설치된 ubuntu 패키지 업데이트 (함부로 하면 안됩니다.)
```Bash
sudo apt-get upgrade 
```  
이유는 시스템이라는게 유기적으로 돌아가는데 업그레이드라는 명령어는 인덱스 정보를 기반으로 현재 우분투에 설치된 
프로그램을 최신 버전으로 업데이트합니다. 최신 버전으로 모두 업데이트 하는 것은 항상 좋은 결과를 만들어내지 않기 때문에 
주의해서 사용해야합니다.  


### apt update와 apt upgrade 비교

- **apt update**: 패키지 목록을 업데이트하여 우분투 소프트웨어 저장소에서 패키지의 최신 메타데이터를 가져오는 단계입니다. 이는 Git에서 `git fetch`와 유사한 작업으로, 원격 저장소의 최신 변경사항을 로컬에 가져오는 역할을 합니다.
- **apt upgrade**: 패키지를 업그레이드하여 최신 버전으로 설치하는 단계입니다. 이는 Git에서 `git pull`과 유사한 작업으로, 원격 저장소의 최신 변경사항을 가져와 로컬 저장소를 업데이트하는 역할을 합니다.

**패키지 설치**  
```Bash
sudo apt-get install 패키지명
```
**패키지 삭제(설정파일 제외)**  
```Bash
sudo apt-get remove 패키지명
```
**패키지 설치(설정파일 포함)**  
```Bash
sudo apt-get --purge remove 패키지명
```  
