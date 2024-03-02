# Dockerfile 주요 명령어  
  
## 사용 목적
+ docker 이미지를 작성할 수 있는 스크립트 파일
+ 확장자가 없다.
+ 나만의 이미지를 만들 수 있고, 배포를 위해서 많이 사용한다.  
  
## 기본 문법
Dockerfile은 텍스트 파일 형식이므로, 에디터로 작성할 수 있음  
**Dockerfiel 기본문법**  
+ 기본적으로 간단한 **명령**과 **인자**로 이루어짐  
+ 명령은 통상적으로 대문자로 작성합니다(소문자도 동작가능)  
```Actionscript
명령 인수
```  

| 명령어           | 설명                                                                                                                                                                       |
|---------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| FROM **(필수)** | 베이스 이미지 지정 명령(예:FROM httpd:alpine) **필수**                                                                                                                                |
| LABEL         | 버전 정보,작성자와 같은 이미지 설명을 하기 위한 명령                                                                                                                                           |
| CMD           | docker 컨테이너가 시작할 때,실행하는 쉘 명령을 지정하는 명령,RUN은 이미지 작성시 실행하는 명령이고,CMD는 컨테이너를 시작할때 실행하는 명령 CMD ["java","exam.java"]                                                            |
| RUN           | 쉘 명령을 실행하는 명령(예:RUN["apt-get","install","nginx"]). RUN은 이미지 작성시 실행되며, 일종의 새로운 layer를 만드는 역할을 함(원하는 프로그램을 동작하기 위한 필요한 프로그램을 설치하는 작업)                                      |
| ENTRYPOINT    | docker 컨테이너가 시작할 때,실행하는 쉘 명령을 지정하는 명령,docker run 커맨드 실행시, 별도 명령어를 넣을 수 있는데, 이 때 CMD 명령은 덮어씌어짐. ENTRYPOINT는 docker run 명령과 함께 전달된 명령어가 있어도, 덮어 씌워지지 않고 ENTRYPOINT 인수로 전달됨 |
| EXPOSE        | docker 컨테이너가 외부 호스트 PC에 오픈할 포트 지정                                                                                                                                        |
| ENV           | docker 컨테이너 내부에서 사용할 환경 변수 지정 (mysql:ENV MYSQL_DATABASE=dev_test)                                                                                                        |
| WORKDIR       | docker 컨테이너에서 작업 디렉토리 설정                                                                                                                                                 |
| COPY          | 파일 또는 디렉토리를 docker 컨테이너에서 복사,ADD와 달리 URL은 지정할 수 없으며,압축 파일을 자동으로 풀어주지 않습니다                                                                                                |
| ARG           | dockerfile 내에서 필요한 변수 설정. 스크립트 내에서 사용하는 변수를 지정하는 역할                                                                                                                      |
| ADD           | COPY 명령이 명시적이므로 COPY를 사용하는 것을 권장, 압축된 파일이 있으면 압축을 해제하고 COPY하는 방향으로 변경,동일한 이름의파일 또는 디렉토리가 docker 이미지에 있을때 덮어씌우지 않음                                                        |
| HEALTHCHECK   | 컨테이너의 시작 시 건강 상태 확인.                                                                                                                                                     |
| MAINTAINER    | 이미지의 작성자 지정.                                                                                                                                                             |
| ONBUILD       | 생성한 이미지를 기반으로, 새로운 이미지 생성시 실행하는 명령을 지정                                                                                                                                   |
| SHELL         | 이미지의 기본 쉘 설정. CMD 등으로 대체 가능 (예:["/bin/bash","-c"])                                                                                                                       |
| STOPSIGNAL    | 컨테이너 종료 시스템 호출 시그널 지정.                                                                                                                                                   |
| USER          | 사용자 및 그룹 ID 설정.                                                                                                                                                          |
| VOLUME        | 볼륨 마운트 생성.                                                                                                                                                               |
  
## 주석
```Actionscript
# 도커 파일 주석은 #을 쓰면, 해당 라인이 주석이 됨
```  
  
## 도커파일 이미지로 만들기
```Actionscript
docker build [OPTIONS] PATH | URL | -
docker image build [OPTIONS] PATH | URL | -
```  

### --tag
이미지의 이름을 설정하는 명령어, 이미지:`[태그]`에서 태그 이름을 지정하는 것이 아님.  
  
### -f  
도커 파일명이 디폴트(`Dockerfile`)가 아닌 경우 지정합니다.
```Actionscript
docker build --tag create-image-name -f Dockerfile-custom ./
```  
  
마지막 `./`은 현재 디렉토리를 나타내며 ` . `도 가능하지만 명시적인 작성을 위해 `./`을 권장  
  
### 태그 추가하기
```Actionscript
docker build --tag image-name:add-tag -f Dockerfile-custom ./

docker image ls
NAME        TAG
image-name  add-tag
```  
  

## 정리  

### 기본 예제
```Actionscript
# 베이스 이미지 설정
FROM httpd:alpine

# 작성자를 지정할 수 있음
LABEL maintainer="kamser0415@fun.com"

# 파일 복사
COPY ./2021_DEV_HTML /usr/local/apache2/htdocs

# 덮어씌우지 못하기 때문에 반드시 실행되는 명령
ENTRYPOINT ["/bin/echo","hello"]
```  
### 환경설정
```Actionscript
FROM mysql:8.0.36

ENV MYSQL_ROOT_PASSWORD=1234
ENV MYSQL_DATABASE=libary
```  

### 필요한 프로그램 설치
```Actionscript
FROM ubuntu:18.04
LABEL maintainer="ksksk"

# 필요한 프로그램 설치
RUN apt-get update

RUN apt-get install -y apache2 apt-utils

# 포트 개방
EXPOSE 80
COPY ./2021_DEV_HTML /var/www/html/

ENTRYPOINT ["/usr/sbin/apache2ctl","-D","FOREGROUND"]
```  
#### 작성방법
```Actionscript
RUN apt-get install -y apache2 apt-utils
```
+ 스페이스바를 구분자로 여러 프로그램을 설치할 수 있습니다.
+ `apt-utils`는 이미 만들어진 도커 프로그램 안에 파일이 있다면 캐시된 것을 사용합니다.

(도커 이미지 만드는 최고의 방법)[https://docs.docker.com/develop/develop-images/instructions/#apt-get] 
### 기본 작업 폴더 위치 지정
```Actionscript
FROM httpd:alpine

WORKDIR /usr/local/apache2/htdocs

CMD ["/bin/cat","index.html"]
```  
작업폴더를 지정하여 전달될 팡리의 위치를 지정할 수 있습니다.
  
## 주의사항

### docker stop 과 docker kill 차이
docker stop은 즉시 컨테이너를 종료하는게 아니라 내부에 동작하는 종료 생명주기까지 기다리고 종료합니다ㅣ.
하지만 docker kill은 즉시 종료합니다.

### docker logs  
만약 아파치를 도커 컨테이너로 사용하고 있다면  
이 로그(아파치컨테이너)는 아파치 웹서버에서 접속 기록이나 요청 기록을 기록해 놓기 위해서
특정 로그라는 파일에 저장하는 기록입니다.

도커 logs 컨테이너 표준 출력으로 콘솔창에 출력된다고 할 수 있습니다.
원래는 아파치 웹 서버 로그를 특정 파일에 저장하는 것이 일반적인데
도커 이미지용 httpd는 이것을 그냥 다 표준 출력으로 콘솔에 출력합니다.
그래서 도커 logs를 이용하면 이런 기록이나 쭉 한번에 볼 수 있다

### CMD를 덮어 씌우는 방법
도커 파일 스크립트
```Actionscript
FROM httpd:alpine

LABEL maintainer="kamser0415@fun.com"

COPY ./2021_DEV_HTML /usr/local/apache2/htdocs

ENTRYPOINT ["/bin/echo","hello"]
```
```Actionscript
docker run -dit -p 9999:80 --name httpdweb2 myweb:error /bin/sh -c httpd-foreground
```
```Actionscript
"Cmd": [
        "/bin/sh",
        "-c",
        "httpd-foreground"
    ],
"Entrypoint": [
    "/bin/echo",
    "hello"
],
```  
`CMD`에 명령어와 인수가 없지만 도커를 실행할 때 도커 이미지 뒤에 실행할 명령어를 전달 할 수 있습니다.  
```Actionscript
docker logs httpdweb2
hello /bin/sh -c httpd-foreground
```  
구조를 확인해보면
`[ENTRYPOINT] + [CMD]`로 ENTRYPOINT가 있다면 CMD는 뒤에 인수로 전달됩니다.