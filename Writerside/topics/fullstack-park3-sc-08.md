# 도커 기타 명령어  

## 도커 이미지 히스토리 보기  
공식 명령어
```Actionscript
docker image history [OPTIONS] IMAGE
```  
도커 파일 내용
```Actionscript
FROM ubuntu:18.04
LABEL maintainer="dasdfasd"

RUN apt-get update
RUN apt-get install -y apache2

COPY ./2021_DEV_HTML /var/www/html

ENTRYPOINT ["/usr/sbin/apache2ctl","-D","FOREGROUND"]
```
도커 파일을 이미지로 만들고 `history`보기
```Actionscript
docker image history myweb-ubuntu

IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
f1d3bbdd3d36   3 hours ago    ENTRYPOINT ["/usr/sbin/apache2ctl" "-D" "FOR…   0B        buildkit.dockerfile.v0
<missing>      3 hours ago    COPY ./2021_DEV_HTML /var/www/html # buildkit   13.1MB    buildkit.dockerfile.v0
<missing>      17 hours ago   RUN /bin/sh -c apt-get install -y apache2 # …   96.5MB    buildkit.dockerfile.v0
<missing>      17 hours ago   RUN /bin/sh -c apt-get update # buildkit        45.7MB    buildkit.dockerfile.v0
<missing>      17 hours ago   LABEL maintainer=dasdfasd                       0B        buildkit.dockerfile.v0
<missing>      9 months ago   /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B        
<missing>      9 months ago   /bin/sh -c #(nop) ADD file:3c74e7e08cbf9a876…   63.2MB    
<missing>      9 months ago   /bin/sh -c #(nop)  LABEL org.opencontainers.…   0B        
<missing>      9 months ago   /bin/sh -c #(nop)  LABEL org.opencontainers.…   0B        
<missing>      9 months ago   /bin/sh -c #(nop)  ARG LAUNCHPAD_BUILD_ARCH     0B        
<missing>      9 months ago   /bin/sh -c #(nop)  ARG RELEASE                  0B    
```  
역순으로 이미지 레이어가 어떻게 쌓였는지 확인 할 수 있습니다.  
  

## 도커 컨테이너와 로컬 파일 시스템간 복사  

**로컬 파일을 컨테이너에 복사하는 방법**
```Actionscript
docker cp ./some_file CONTAINER:/work
```  
  
**컨테이너내 파일/폴더를 로컬에 복사하는 방법**
```Actionscript
docker cp CONTAINER:/var/logs/ /tmp/app_logs
```  
  
**컨테이너에서 표준 출력으로 파일을 복사합니다.**  
```Actionscript
docker cp apache:/var/log/apache2/access.log - | tar x -O | grep GET > access-apache.txt
```  
` - ` 명령은 파일의 출력을 표준 출력인 콘솔로 내보냅니다.
```Actionscript
docker cp apache:/var/log/apache2/access.log - 

access.log0000640000000000000040000001573014570775622011235 
0ustar0000000000000000121.169.183.19 - - 
[03/Mar/2024:04:17:51 +0000] "GET / HTTP/1.1" 304 182 "-" 
"Mozilla/5.0 (Windows NT 10.0; Win64; x64) 
AppleWebKit/537.36 (KHTML, like Gecko) 
Chrome/122.0.0.0 Safari/537.36"
```  
결과가 아카이브된 파일(`tar`)로 출력이 되기 때문에 `| tar x -O ` 파이프라인으로 아카이브를 풀고 표준출력으로 다시 출력하는 명령어 입니다.

## 도커 commit
컨테이너 안에서 어떤 작업을 했다면 컨테이너가 종료나 재시작을 하면 사라지는데
그 안의 변경된 작업을 레이어에 추가해서 새로운 이미지로 만들어주는걸 도커 커밋이라 합니다.  
```Actionscript
docker container commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
docker commit
```  
```Actionscript
docker commit -m "파일추가" apache newviweb:add
docker images
REPOSITORY                     TAG       IMAGE ID       CREATED          SIZE
# tag가 새로 추가된것을 확인할 수 있습니다.
newviweb                       add       0951e5203360   18 seconds ago   268MB
```   
  
## 도커 diff  
컨테이너 파일 시스템의 파일이나 디렉토리에 대한 변경 사항을 검사합니다.
```Actionscript
docker container diff CONTAINER
docker diff
```  
| Symbol | Description         |
|--------|---------------------|
| A      | 파일 또는 디렉토리가 추가되었습니다 |
| D      | 파일 또는 디렉토리가 삭제되었습니다 |
| C      | 파일 또는 디렉토리가 변경되었습니다 |
  
```Actionscript
docker diff apache

A /new_file.txt # 아까 추가한 파일이 있어서 A로 표시되었습니다.
C /var
C /var/log
C /var/log/apache2
C /var/log/apache2/access.log
C /var/log/apache2/error.log
C /root
C /root/.viminfo
C /root/.bash_history
C /run
C /run/apache2
C /run/apache2/apache2.pid
```  
  
## 도커 inspect  
Docker inspect는 Docker에 의해 제어되는 구조에 대한 상세 정보를 제공합니다.  
기본적으로, docker inspect는 결과를 JSON 배열로 렌더링합니다.
```Actionscript
docker inspect [OPTIONS] NAME|ID [NAME|ID...]

# 타입 명시가능
--type container|image|node|network|secret|service|volume|task|plugin

# 예시
docker inspect --type=volume myvolume
docker inspect network bridge
docker inspect container apache
```  
  
## 도커 --link
[도커 네트워킹 공식문서](https://docs.docker.com/network/)  
  
`--link`는 도커 컨테이너 간의 이름을 통해서 통신할 수 있는 방법입니다.  
현재는 `Deprecated` 상태입니다.  
  
### Deprecated 이유
+ 유연성 부족: --link를 사용하면 컨테이너 간의 네트워크 연결이 고정되어 있어서 유연성이 부족합니다. 컨테이너가 동적으로 생성되거나 다시 시작될 때 IP 주소가 변경되는 문제가 있습니다.
+ 보안 문제: --link를 사용하면 컨테이너 간의 통신이 보안적으로 취약할 수 있습니다. 컨테이너 간에 직접적인 링크가 설정되므로 보안상의 위험을 초래할 수 있습니다.
+ 복잡성: --link를 사용하는 것은 네트워크 관리를 복잡하게 만들 수 있습니다. 특히 여러 개의 컨테이너를 관리할 때 더욱 복잡해질 수 있습니다.  
  
### 사용방법
```Actionscript
docker run --rm -d -p 8888:8888 \
-v /home/yousd179/2021_LEARN:/home/jovyan/work \ 
--link mydb:myjupyterdb jupyter/datascience-notebook
```  
`jupyter/datascience-notebook`이미지를 사용하는 컨테이너가 `mydb` 컨테이너에 접근할때 `ip` 대신에 
link 된 이름인 `myjupyterdb`을 통해서 접근할 수 있습니다.  
  
### 다른 방식으로 연결하기
```Actionscript
# 네트워크 브릿지 생성(기본)
docker network create my-net

# mydb 컨테이너 네트워크망 my-net 사용
docker run -d --network=my-net  --name mydb mysqldb

# jupyter 컨테이너 네트워크망 my-net 사용
docker run --rm -d -p 8888:8888 
--network=my-net \ 
-v /home/yousd179/2021_LEARN:/home/jovyan/work 
jupyter/datascience-notebook
```  
#### 네트워크 드라이브 종류
| 드라이버 이름 | 설명                                                                                 |
|---------|------------------------------------------------------------------------------------|
| bridge  | 기본적으로 제공되는 Docker 네트워크 드라이버로, 동일한 호스트에서 실행되는 컨테이너 간 통신에 사용됩니다.                     |
| overlay | 여러 호스트에서 실행되는 컨테이너 간 통신을 위한 드라이버로, Docker Swarm에서 사용됩니다.                           |
| host    | 호스트의 네트워크 스택을 공유하는 드라이버로, 호스트 네트워크와 컨테이너 네트워크를 동일하게 만듭니다.                          |
| macvlan | 호스트의 물리적 네트워크 인터페이스를 통해 컨테이너에 고유한 MAC 주소를 할당하는 드라이버로, 네트워크의 외부에 직접 연결되는 것처럼 동작합니다. |
| none    | 네트워크 드라이버를 사용하지 않는 것으로, 컨테이너가 네트워크를 사용하지 않음을 나타냅니다.                                |

  
#### 네트워크 확인해보기
```Actionscript
docker inspect network my-net
[
    {
        "Name": "my-net",
        "Id": "1895b42df93e46110306f3b57df546d5ffe1023c94013ef86c6ea80852e67bd2",
        "Created": "2024-03-03T06:47:49.735746379Z",
        "Containers": {
            "547024c792cae1b558f7fa50c38df50f1bd36f3f1e2b28a18791eba46e90ce7a": {
                "Name": "mydb",
                "EndpointID": "4b4f3995357972dc270527248d9db8e8be4606f8f473d2e9986f6c77f39f20d3",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
            },
            "e2950474b18e49a5a01f87b378ad6cd9d4d67b9e00b23e224ba41c316682e549": {
                "Name": "modest_jang",
                "EndpointID": "39f52ed606c6ab38836d6ad86cfb5b9355afede7d1d23ce9a865cb195044ecc8",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
            }
        },
    }
]
```  
서로 컨테이너 이름을 통해서 접근할 수 있습니다.  
서로 다른 네트워크 망끼리는 접근할 수 없기 때문에 보안에 유리합니다.  
  
Docker에서는 아무런 `--network` 옵션을 입력하지 않으면 기본적으로 컨테이너가 기본 브릿지 네트워크(default bridge network)에 연결됩니다.  
따라서 다음과 같이 docker run 명령어를 실행하면 기본 브릿지 네트워크에 컨테이너가 연결됩니다.  
  
