# 이미지를 다루는 다양한 옵션  
   
[공식 도커 CLI Reference](https://docs.docker.com/reference/)   

모든 도커 명령은 CLI로 키보드로 직접 입력하는 형태로 수행하며, 명령 형식은 크게 다음과 같습니다.
```Actionscript
docker 명령 옵션 선택자(이미지ID/컨테이너등)
```  
+ docker는 image와 container의 명령어가 각각 존재합니다.

```Actionscript
docker [image,container] 
```  
최근에는 해당 키워드를 붙여주는 경향이 있습니다.( 구분하기 쉽게 하기 위해서 )
  
도커 Desktop에 제공되는 GUI를 활용해도 되지만, CLI로 로그인 할 수 도 있습니다.  
```Actionscript
docker login [OPTIONS] [SERVER]
```


## 이미지 검색
```Actionscript
docker search --filter is-official=true --filter stars=200 ubuntu

NAME                DESCRIPTION                                     STARS     OFFICIAL
ubuntu              Ubuntu is a Debian-based Linux operating sys…   16916     [OK]
websphere-liberty   WebSphere Liberty multi-architecture images …   298       [OK]
```  

+ docker 이미지는 크게 `이미지명:[태그]`로 이루어질 수 있습니다.  
+ 태그는 보통 버전 정보를 넣는 경우가 많습니다.  
+ 이미지명에 태그를 넣지 않으면, 최신 버전의 이미지를 의미하며, 최신 버전 이미지에는 가장 최신이라는 의미로 `tag:latest`가 붙습니다.  
  + 따라서 이미지명에 태그가 안붙으면 받은 이미지의 태그명은 `:latest`가 됩니다.
  + 그리고 별도로 `:latest`라고 저장하는 경우는 최신 업로드는 다른 태그로 지정할 때 사용합니다.  
  
## 이미지 다운로드  
```Actionscript
docker image pull ubuntu:20.04

20.04: Pulling from library/ubuntu
8ee087424735: Pull complete 
Digest: sha256:bb1c41682308d7040f74d103022816d41c50d7b0c89e9d706a74b4e548636e54
Status: Downloaded newer image for ubuntu:20.04
docker.io/library/ubuntu:20.04
```

## 이미지 목록 확인
```Actionscript
docker image ls [OPTIONS] [REPOSITORY[:TAG]]
docker images 
```
특정 이미지를 확인하고 싶을 경우
```Actionscript
docker image ls ubuntu
```
이미지 ID만 확인할 수 있습니다.
```Actionscript
docker images -q
```
전체 이미지 중에서 태그가 없는 이미지를 삭제한다고 한다면
```Actionscript
docker rmi $(docker images -f "dangling=true" -q)
```  
  
## 이미지 삭제하기
```Actionscript
docker image ls
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
ubuntu        20.04     18ca3f4297e7   5 weeks ago     72.8MB
hello-world   latest    d2c94e258dcb   10 months ago   13.3kB
```  

1. 이미지명과 태그로 삭제하기
    ```Actionscript
    docker image rmi ubuntu:20.04 
    ```
2. 이미지 아이디로 삭제하기
    ```Actionscript
    docker image rmi 18ca 
    ```  
  
git의 커밋 코드처럼 중복되는 것만 없다면 id를 모두 입력할 필요가 없습니다.  

```Actionscript
yousd179@instance-20240224-071346:~$ docker image ls tomcat
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
tomcat       9.0.86    9f993f8252da   10 days ago   457MB
tomcat       latest    549891b60da9   10 days ago   455MB  

yousd179@instance-20240224-071346:~$ docker image ls tomcat -q
9f993f8252da
549891b60da9  

yousd179@instance-20240224-071346:~$ docker rmi $(docker image ls tomcat -q)  
Untagged: tomcat:9.0.86
Untagged: tomcat:latest
```
