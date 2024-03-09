# 웹 서버 이해와 도커로 웹 서부 구축하기  
  
각각의 서비스를 도커 컨테이너로 실행하고, 그 전체를 도커 컴포즈로 구성해서 실행하기 위해서는 
기본적으로 웹 서버에 대해서 상세하기 이해할 필요가 있습니다.  
  
## Apache MPM

MPM은 다중처리 모듈(Multi Processing Module)의 약자로, 클라이언트로 받은 요청을 어떤 방식으로 처리할 것인지에 대한 모듈을 말합니다.  
  
아파치에서 가장 대중적으로 사용하는 MPM은 크게 3가지가 있습니다.  
+ Prefork
+ Worker
+ Event
  
### Prefork  
![image_133.png](image_133.png)  
  
하나의 요청에 대해서 **_하나의 자식 프로세스가 하나의 스레드를 사용해서 처리하는 방식_**  

+ 미리 복수의 프로세스를 생성하여 클라이언트의 요청에 대비하는 멀티프로세스 방식이다.
+ 하나의 요청에 대해서 하나의 자식 프로세스가 하나의 스레드를 사용해서 처리하는 방식이다. 동시에 여러개의 요청이 들어올 경우, 미리 생성되어 있는 자식 프로세스에서 각 요청을 처리한다.
+ 각 프로세스들의 자원이 독립적이기 때문에(프로세스 복제시 메모리 영역까지 복제), 다른 요청이 들어오거나 프로세스 하나에 오류가 발생해도 다른 요청에 영향이 가지 않는다.
+ 하지만 프로세스가 많아질 경우, 메모리가 많이 소비된다.
                       
### Worker
![image_136.png](image_136.png)  

+ 멀티쓰레드와 멀티프로세스의 하이브리드형 박식으로, 각 요청을 프로세스의 스레드로 받아 처리한다.
+ 연결마다 같은 메모리 공간을 공유하기 때문에 경합이 발생할 수 있고 이에 따른 주의가 필요하다.
+ 프로세스 별 스레드 개수 제한까지 요청을 받으며, 일정 개수가 넘어갈 때는 프로세스를 생성하여 처리한다.
+ prefork에 비해 메모리 등 리소스 사용량이 비교적 적다.
+ 하지만 프로세스 오류가 발생할 때 프로세스 내 스레드까지 죽어버리므로, 여러 연결이 동시에 끊어질 수 있다.
+ 통신량이 많은 대규모 환경에 적합하다. 
  
`root@44297c0c4e1e:/usr/local/apache2/conf/original/extra# vi httpd-mpm.conf 에서 확인가능`  
  
```Actionscript
httpd -V
Server version: Apache/2.4.58 (Unix)
Server built:   Feb 13 2024 01:53:49
Server's Module Magic Number: 20120211:129
Server loaded:  APR 1.7.2, APR-UTIL 1.6.3, PCRE 8.39 2016-06-14
Compiled using: APR 1.7.2, APR-UTIL 1.6.3, PCRE 8.39 2016-06-14
Architecture:   64-bit
Server MPM:     event
  threaded:     yes (fixed thread count)
    forked:     yes (variable process count)
```  
명령어로 확인해보면 최신 아파치는 `event` MPM을 사용한다는 것을 확인할 수 있습니다.  
  
### Event  
+ Worker 방식을 기반으로 멀티쓰레드와 멀티프로세스로 동작한다. 클라이언트로부터 데이터를 기다리도록 유지되는 단점을 보완하기 위해 각 프로세스에 대한 전용 리스너 스레드를 사용하여 Listening 소켓, Keep Alive 상태에 있는 모든 소켓, 처리기 및 프로토콜 필터가 작업을 수행한 소켓 및 나머지 유일한 소켓을 처리한다.
+ 기존에는 클라이언트의 연결이 완전히 끝나지 않는 한 하나의 프로세스(또는 스레드)를 계속 연결하고 있었다.(리소스 부하) 따라서 대량 접속이 발생하는 경우 효율이 굉장히 떨어지는 이슈가 있었다.
+ Event Driven 지원 (한개 또는 고정된 프로세스만 생성하고 그 프로세스의 내부에서 비동기 방식으로 효율적으로 작업을 처리하는 방식으로, 속도가 가장 빠르다.)
+ 클라이언트 요청을 받는 스레드를 따로 두어, 분산된 처리를 한다.  
  
## Nginx  
> Event Driven + 비동기 처리  
  
![image_138.png](image_138.png)  

`요청 > Request Queue 에 저장 (Event Loop) -> 처리(비동기)`  
  
[Nginx와 Apache](https://velog.io/@moonyoung/Nginx와-Apache)  
[Nginx와 Backend 지연 처리 분석](https://blog.naver.com/bumsukoh/222179640856)  
[피케이의 Nginx](https://youtu.be/6FAwAXXj5N0?feature=shared)  
[피케이의 Nginx 정리 파일](https://sihyung92.oopy.io/server/nginx_feat_apache)  
[Nginx 비교분석](https://codenme.tistory.com/82)
  
Nginx는 간단하게 설명하면 쿠버네티스와 구조가 유사합니다.  
마스터 프로세스와 워커 프로세스로 구성되어있습니다. 
마스터 프로스세는 설정 파일을 동적으로 읽어 워커 프로세스를 관리합니다. 
워커 프로세스는 실제로 웹 요청을 처리하는 프로세스로 `Apache`와 다르게 모든 작업을 `event`로 여기고 처리합니다.  
작업 큐에 이벤트를 넣어 처리를 하며 오래 걸리는 디스크 I/O 작업은 별도의 쓰레드 풀에 넣어서 작업을 위임합니다.  
  
성능을 중시하는 Nginx는 아파치와 다르게 개발자가 모듈을 추가하기 어려우며 워커 프로세스도 CPU의 코어 수와 동일하게 실행하여 
컨택스트 스위칭을 최적화했습니다.   
  
정리하면 Nginx는 대용량 트래픽에 적합하고, Apache는 OS 안정성,다양한 모듈 지원,디렉토리별 추가 구성이 가능하기 때문에 
필요한걸 선택해서 사용하면 되지만, 성능 차이는 크게 나지 않는다고 합니다.  
  
아파치와 Nginx는 목적이 다른 웹서버이기 때문에 목적에 맞게 사용하는 방법을 아는 것도 중요하다고 생각합니다.  
  
## 기본 사용법  
```Actionscript
docker num -dit -p 80:8080 --name myos ubuntu:20.04
```  
기본 베이스로 우분투 리눅스:20.04 버전을 설치합니다.  
이제 우분투로 들어가서 Nginx를 설치합니다.  
```Actionscript
docker exec -it myos /bin/bash
apt-get update
```  
**apt-get update**는 운영체제에서 사용 가능한 패키지들과 그 버전에 대한 정보를 업데이트하는 명령어입니다. 설치되어 있는 패키지를 최신으로 업데이트하는 것이 아닌 설치 가능한 리스트를 업데이트하는 것  
  
```Actionscript
apt-cache policy nginx
```  
Ubuntu 및 Debian Linux에서 패키지의 설치 상태와 후보 버전 정보를 표시하는 데 사용됩니다. 이 명령은 다음과 같은 정보를 제공합니다:

+ Installed: 현재 시스템에 설치된 패키지 버전.
+ Candidate: 사용 가능한 패키지 버전.
+ Version table: 사용 가능한 패키지 버전과 해당 버전의 소스 저장소 정보.
예를 들어, 아래와 같은 출력이 나타날 수 있습니다:
```Actionscript
nginx:
  Installed: (none)
  Candidate: 1.18.0-0ubuntu1.4
  Version table:
     1.18.0-0ubuntu1.4 500
        500 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages
        500 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages
     1.17.10-0ubuntu1 500
        500 http://archive.ubuntu.com/ubuntu focal/main amd64 Packages 
```  
버전 테이블에서 원하는 버전의 `Nginx`를 설치합니다.  
```Actionscript
apt-get install nginx=1.18.0-0ubuntu1.4
```  
그러면 이렇게 웹 서버의 시간을 설정하는 CLI가 나옵니다.
```Actionscript
Please select the geographic area in which you live. Subsequent configuration questions will narrow this down by presenting a list of cities, representing the time zones in which they
are located.

  1. Africa  2. America  3. Antarctica  4. Australia  5. Arctic  6. Asia  7. Atlantic  8. Europe  9. Indian  10. Pacific  11. SystemV  12. US  13. Etc
Geographic area: 6

Please select the city or region corresponding to your time zone.

  1. Aden      10. Bahrain     19. Chongqing  28. Harbin       37. Jerusalem    46. Kuala_Lumpur  55. Novokuznetsk  64. Qyzylorda      73. Taipei         82. Ulaanbaatar
  2. Almaty    11. Baku        20. Colombo    29. Hebron       38. Kabul        47. Kuching       56. Novosibirsk   65. Rangoon        74. Tashkent       83. Urumqi
  3. Amman     12. Bangkok     21. Damascus   30. Ho_Chi_Minh  39. Kamchatka    48. Kuwait        57. Omsk          66. Riyadh         75. Tbilisi        84. Ust-Nera
  4. Anadyr    13. Barnaul     22. Dhaka      31. Hong_Kong    40. Karachi      49. Macau         58. Oral          67. Sakhalin       76. Tehran         85. Vientiane
  5. Aqtau     14. Beirut      23. Dili       32. Hovd         41. Kashgar      50. Magadan       59. Phnom_Penh    68. Samarkand      77. Tel_Aviv       86. Vladivostok
  6. Aqtobe    15. Bishkek     24. Dubai      33. Irkutsk      42. Kathmandu    51. Makassar      60. Pontianak     69. Seoul          78. Thimphu        87. Yakutsk
  7. Ashgabat  16. Brunei      25. Dushanbe   34. Istanbul     43. Khandyga     52. Manila        61. Pyongyang     70. Shanghai       79. Tokyo          88. Yangon
  8. Atyrau    17. Chita       26. Famagusta  35. Jakarta      44. Kolkata      53. Muscat        62. Qatar         71. Singapore      80. Tomsk          89. Yekaterinburg
  9. Baghdad   18. Choibalsan  27. Gaza       36. Jayapura     45. Krasnoyarsk  54. Nicosia       63. Qostanay      72. Srednekolymsk  81. Ujung_Pandang  90. Yerevan
Time zone: 69
```  
6번 69번을 눌러서 서울로 선택하면 됩니다.  
  
```Actionscript
user www-data; 
    # 웹 서버를 돌리는 아이디, 웹서버 권한이 있어서 설정이 가능하다.
worker_processes auto; 
    # 워커 프로세스 개수 설정
pid /run/nginx.pid; 
    # 시스템과 nginx의 프로세스와의 연결고리와 관련된 설정
include /etc/nginx/modules-enabled/*.conf; 
    # 설정이 하나의 파일에 있는게 아니라 여러가지 플러그인들이 있을 수 있고
    # 플러그인과 관련된 설정이 있기 때문에 여러 파일에 나누어있을 수 있습니다.
    # 그 파일들을 정규식으로 모두 포함시키는 부분입니다. 
    # /modules-enabled/ 모듈 파일을 모두 적용

events {
        worker_connections 768; # 동시에 이벤트를 몇개 지원할지 지정
        # multi_accept on;
}
```
http 블럭에서 설정할 수 있는 모든 [설정 공식사이트](https://nginx.org/en/docs/http/ngx_http_core_module.html)  
  
```Actionscript
http {
        ##
        # Virtual Host Configs
        ##
        ## 1번
        include /etc/nginx/conf.d/*.conf;
        ## 2번
        include /etc/nginx/sites-enabled/*;
}
```  
http 블록에서 설정할 수 있는 것을 1번,2번 폴더에 별도로 각각 기능별이나 사이트 별로 설정파일로 만들고 해당 폴더에 넣으면 
모두 읽어서 적용할 수 있습니다.  
  
`/etc/nginx/sites-enabled`로 이동하고 `default`파일을 확인할 수 있습니다.  
  
```Actionscript
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
        }
```  
이렇게 기본 웹 서버에 대한 기본 설정을 확인할 수 있습니다.  
  
```Actionscript
root@7e77a81cfbc1:/etc/nginx/sites-enabled# ls -al
total 8
drwxr-xr-x 2 root root 4096 Mar  9 21:25 .
drwxr-xr-x 8 root root 4096 Mar  9 21:39 ..
lrwxrwxrwx 1 root root   34 Mar  9 21:25 default -> /etc/nginx/sites-available/default
```  
심볼릭 링크가 되어있는데 다시 구조를 확인해보면
```Actionscript
root@7e77a81cfbc1:/etc/nginx# ls
conf.d  
sites-enabled/default ->  sites-available/default 
```  
`Nginx` 웹서버가 참고하는 `http`블럭내 참조하는 설정 디렉토리는 3가지 입니다.  
Nginx에서 sites-available 폴더와 sites-enabled 폴더를 사용하는 이유는 **가상 호스팅(Virtual Hosting)** 을 효율적으로 관리하기 위함입니다.

+ **sites-available**: 이 폴더는 비활성화된 사이트들의 설정 파일들을 저장하는 곳입니다. 즉, 아직 활성화되지 않은 가상 호스팅 설정 파일들이 위치합니다.
+ **sites-enabled**: 이 폴더는 활성화된 사이트들의 설정 파일들을 저장하는 곳입니다. 실제로 웹 서버에서 사용 중인 가상 호스팅 설정 파일들이 이 폴더에 있습니다.
    
이렇게 나누어 관리하는 이유는 다음과 같습니다:

+ 모듈화와 유연성: `sites-available` 폴더에는 모든 설정 파일을 미리 준비해두고, 필요한 사이트만 심볼릭 링크로 활성화할 수 있습니다. 이렇게 하면 설정 파일을 모듈화하여 관리할 수 있고, 필요한 사이트를 쉽게 활성화/비활성화할 수 있습니다.
+ 가독성과 정리: `sites-enabled` 폴더에는 현재 활성화된 사이트들만 있으므로, 설정 파일을 보다 간결하게 관리할 수 있습니다. 또한 가상 호스팅 설정 파일들이 분리되어 있어 가독성이 좋아집니다.
이렇게 나누어진 폴더 구조는 웹 서버 설정을 더욱 체계적으로 관리하고, 가상 호스팅을 효율적으로 구성할 수 있도록 도와줍니다.  
  
출처 : [왜 나누었을까?](https://serverfault.com/questions/527630/difference-in-sites-available-vs-sites-enabled-vs-conf-d-directories-nginx)  
  
## default 파일 server 설정  
```Actionscript
server {
        # listen 80 default_server;
        listen 8080 default_server;
        listen [::]:8080 default_server;
}
```
8080 포트를 사용하도록 변경합니다.

```Actionscript
root /var/www/html;

# Add index.php to the list if you are using PHP
index index.html index.htm index.nginx-debian.html;

server_name _;

location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        try_files $uri $uri/ =404;
}
```  
`server_name_`은 요청을 받을 도메인이름을 설정합니다.  
예를 들어:  
```Actionscript
server_name kokopadock.shop www.kokopadock.shop;
```  
만약 서버네임이 없다면 이렇게 디폴드인 `server_name _;`으로 설정해도 됩니다.  
  
+ `root` : 기본적인 ip:port를 입력했을때 찾는 기본 디렉토리 위치
+ `index`
  + index.html index.htm index.nginx-debian.html 파일명을 순차로 찾아서 반환한다는 의미입니다.  
  
### location  
```Actionscript
12.125.125.245/blog
```  
위 주소로 입력할 경우 `/` 이후부터가  `location /`에 해당하게 됩니다.  
먼저 `try_files`에서 `$url`은 `blog`가 되기때문에 해당 파일(`blog`)을 찾는다.  
없다면 `$url/` `blog/`폴더를 찾는다.  
없다면 `=404` 404 NOT-FOUND를 반환한다는 의미입니다.  
    
**Nginx 다시 시작**
```Actionscript
service nginx restart
 * Restarting nginx nginx
```    
   
```Actionscript
location /blog {
        # /var/www/index.html file-read
        root /var/www;
}
location /flask {

        root /var/www;
}
```
접속 URL이 `www.kokopadock.shop/blog`로 들어온다면 `/var/www/blog/`안에 index 파일을 찾습니다.  
  
이렇게 `location` 설정으로 해당하는 서비스를 연결해줄 수 있습니다.  
  
## nginx reverse proxy  
  
**Proxy Server 란**  
+ 클라이언트가 자신을 통해, 다른 네트워크 서비스에 접속할 수 있게 하는 서버를 말한다.  

**포워드 프록시 (Forward Proxy)**  
+ 클라이언트가 인터넷에 접근할 때, 직접 웹 서버에 요청하는 것이 아니라 포워드 프록시 서버를 통해 요청합니다.
+ 포워드 프록시는 클라이언트의 요청을 받아서 해당 웹 서버로 전달해주는 역할을 합니다.
+ 주로 보안과 캐싱을 위해 사용됩니다.
+ 예시: 회사 내부에서 직원들이 인터넷을 사용할 때, 포워드 프록시를 통해 외부 웹 서버에 접근합니다.  
  

**리버스 프록시 (Reverse Proxy)**  
+ 클라이언트가 웹 서비스에 접근할 때, 직접 웹 서버에 요청하는 것이 아니라 리버스 프록시 서버를 통해 요청합니다.
+ 리버스 프록시는 클라이언트의 요청을 받아서 내부 서버 (보통 WAS)에서 데이터를 받은 후 클라이언트에게 전달합니다.
+ 주로 보안, 로드 밸런싱, 성능 최적화를 위해 사용됩니다.
+ 예시: 웹 서비스를 제공하는 서버를 내부망에 두고, 리버스 프록시를 통해 외부에서 요청이 들어오면 내부 서버로 연결합니다.  
  
## Reverse Proxy 설정 익히기  
### 포트로 구분하기  
프록시 서버에서 외부에 포트 2개 (`8080`,`8081`)을 열고, 각각 포트에 들어온 요청을 내부 컨테이너로 돌아가는
`아파치`,`Nginx`에 전달하여 결과를 리버스 프록시가 받아 클라이언트에게 전달하는 방식입니다.  

```Actionscript
version: "3"

services:
    nginxproxy:
        image: nginx:1.18.0
        ports:
            - "8080:8080"
            - "8081:8081"
        restart: always
        volumes:
            - "./nginx/nginx.conf:/etc/nginx/nginx.conf"

    nginx:
        depends_on: # 우선순위
            - nginxproxy
        image: nginx:1.18.0
        restart: always

    apache:
        depends_on: # 우선순위
            - nginxproxy
        image: httpd:2.4.46
        restart: always
```  
도커 컴포즈를 통해서 하나의 명령어로 3개의 컨테이너 서버를 동작했습니다.   
+ 리버스 프록시 역할 컨테이너 : `nginxproxy`  
  + 내부 아파치 웹 서버 컨테이너 : `apache 2.4.46`
  + 내부 Nginx 웹 서버 컨케이너 : `nginx 1.18.0`  
  
![image_139.png](image_139.png)  
  
클라이언트가 요청을 보낸 포트에 따라 내부 서버에 전달해야하기 때문에 `Nginx`설정 파일을 `Volumes`를 통해서 설정했습니다.  
  
```Actionscript
error_log  /var/log/nginx/error.log warn;
```  
+ `error_log`: 에러 로그가 생겼을 때 에러 로그를 어디에 저장할지 위치를 지정합니다.  
  
```Actionscript
http {
  include /etc/nginx/mime.types;
  default_type aplication/octet-stream;
}  
```  
**include /etc/nginx/mime.types;**  
+ 이 옵션은 **MIME 타입 (Multipurpose Internet Mail Extensions)** 을 정의하는 파일인 **mime.types**를 포함하도록 설정합니다.
+ MIME 타입은 파일 확장자와 해당 파일의 컨텐츠 유형을 매핑하는 역할을 합니다. 예를 들어, `.html` 파일은 `text/html MIME 타입`으로 인식됩니다.
+ 이 설정은 Nginx가 정확한 MIME 타입을 지정하여 클라이언트에게 전달할 수 있도록 도와줍니다.  
  
> 예를들어,  
> /blog/name.json, /blog/name.txt 등 뒤에 확장자가 무엇을 의미하는지 `Nginx`가 알면 그에 따라서 
> 더 적절한 동작을 할 수 있게 데이터 타입을 지정하는 구문입니다.  
>   
> 스프링 부트에서도 있던 기능인데 지금은 안쓰는걸로 알고 있습니다.
  
**default_type application/octet-stream;**    
+ 이 옵션은 기본 MIME 타입을 설정합니다.
+ `application/octet-stream`은 이진 파일을 나타내는 MIME 타입입니다. 이 설정은 클라이언트에게 해당 파일이 이진 데이터로 처리되어야 함을 알려줍니다.
+ 예를 들어, 브라우저는 이진 파일을 다운로드하도록 유도할 것입니다.
+ 이 설정은 Nginx에서 정확한 MIME 타입을 지정하고, 클라이언트에게 적절한 파일 처리 방법을 알려주는 데 사용됩니다.  
  
> 만약 /blog/name.aaa 처럼 위에 작성된 type이 아닌 경우 표준 인코딩 파일로 이해하겠다는 디폴트 타입입니다.  
     
#### log_format
```Actionscript
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
```  
**log_format**  
로그 파일에 로그를 작성할때 어떤 포멧으로 작성할 것인지에 대한 설정값입니다. 접속 기록을 저장합니다.  
  

#### sendfile 옵션    
[Nginx의 sendfile 이해](https://www.ateamsystems.com/tech-blog/understanding-nginxs-sendfile-parameter-and-its-implications-with-nfs/)
```Actionscript
sendfile on;
```

**sendfile이 off인 경우:**  
1. 클라이언트 요청이 들어오면 웹 서버는 커널로 시스템 콜을 통해 디스크에 있는 파일 데이터를 읽어옵니다.
2. 그런 다음 웹 서버는 이 데이터를 유저 스페이스라고 불리는 가상화 공간에 복사합니다.
3. 마지막으로 웹 서버는 유저 스페이스에서 읽은 데이터를 클라이언트에게 전달합니다.  
   

**sendfile이 on인 경우:**  
1. 클라이언트 요청이 들어오면 웹 서버는 커널로 시스템 콜을 통해 디스크에 있는 파일 데이터를 읽어옵니다.
2. 그러나 이번에는 유저 스페이스에 복사하지 않고, 커널 스페이스에서 직접 클라이언트에게 데이터를 전달합니다.
3. 이렇게 하면 복사 과정이 생략되어 더 효율적으로 파일을 전송할 수 있습니다.  
  
> 정리하면, user 영역 buffer가 아닌 커널 파일 buffer를 사용한다는 의미  
>   
  
#### keepalive_timeout  
연결된 상태를 유지하는 시간  
  
### nginx.conf 설정 (매우중요)  
```Actionscript
upstream docker-nginx {
    server nginx:80; 
}

upstream docker-apache {
    server apache:80;
}
```  
### upstream이란  
[nginx upstream 최적화](https://brunch.co.kr/@alden/11)  
`prox_pass` 지시자를 통해 nginx가 받은 리쿼스트를 넘겨 줄 서버들을 정의하는 지시자가 `upstream`입니다.    

![image_140.png](image_140.png)  
  
이렇게 클라이언트에서 받은 요청을 내부 서버에 전달하는 것을 설정하는 부분입니다.   
```Actionscript
upstream backend {
  server backend1.example.com:9000
}
```  
현재 방식으로 작성하는 건 권장하지 않는다고 합니다.  
`nginx`와 `paly`를 연결하는 내부 통신에도 리쿼스트마다 세션을 만들고 `TCP handshake`가 일어나기 때문입니다.  

![image_141.png](image_141.png)  
  
간단하게 이야기하면 클라이언트와 프록시 서버는 `keepalive`가 있어서 불필요한 `TCP handshake`가 발생하지 않지만, 
프록시 서버와 백엔드 서버는 `keepalive`설정이 없어서 성능저하가 발생합니다.  
  
따라서 아래와 같이 코드를 수정하는 것이 좋습니다.  
```Actionscript
proxy_http_version 1.1; # proxy 통신시 HTTP/1.1로 통신함을 명시
proxy_set_header Connection "";

upstream backend {
  server backend1.example.com:9000
  keepalive 100; # keepalive로 유지시키는 최대 커넥션 개수
} 
```  
읽어야할 keepalive 글  
[tcp keepalive와 nginx keepalive](https://brunch.co.kr/@alden/9)    
[KeepAlive 가 뭐지?](https://velog.io/@msung99/ㄴㄴㄴ)    
[nginx_keepalive_공식문서](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive)  
  
### 서버 설정
```Actionscript
server {
    listen 8080;

    location / {
        proxy_pass         http://docker-nginx;
        proxy_redirect     off;
        proxy_set_header   Host $host;
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Host $server_name;
    }
}

server {
    # 포트로 포워딩
    listen 8081;

    location / {
        # 어디로이동할지에 대한 위치를 지정한 것
        proxy_pass         http://docker-apache;
        proxy_redirect     off;
        proxy_set_header   Host $host;
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Host $server_name;
    }
}
```  
`server` 설정은 `default` 파일에서 작성했습니다.  
`nginx.conf`파일에는 맨 끝에 include라는 옵션이 있어서 특정 폴더 `site-available`이나 `conf` 폴더에 있는 저장된 파일이나 설정을 
import하게 되어있는데 그렇게 하지 않고 하나의 파일에 저장했습니다.  
  
include 할 경우 설정이 겹칠 수 있기 때문입니다.  
```
include /etc/nginx/conf.d/*.conf;
```  
이 부분을 삭제한것입니다.  
  
[Nginx 구조와 파일 소개](https://medium.com/@jina-dev/nginx-기본설정-fa06e7ef612d)  
  
`proxy_set`을 설정하는 이유는 내부 서버사이에 http 통신을 하면, 실제 외부 클라이언트에 대한 정보가 누락되므로 이상 동작을 할수 있습니다.  
+ proxy_rediect: 서버 응답 헤더의 주소 변경
+ Host $host: Host 헤더가 없으면,server_name;
+ X-Real-IP : 클라이언트 IP 주소
+ X-Forwarded-For(XFF) : 클라이언트 IP 주소를 식별하기 위한 설정으로, 클라이언트 IP 부터 중간 서버(여러 프록시) IP들을 리스트로 작성해서 전달합니다.  
  + 위 설정이 없으면, 모든 http 요청은 reversed proxy가 한 것으로 기록되므로, 클라이언트 IP 기록을 위해 필요합니다.  
+ X-Forwarded-For : 프록시 서버가 아닌 실제 클라이언트의 호스트 이름을 기록함
+ X-Forwarded-Host : 클라이언트와 처음 만난 `reversed proxy` 접속시 사용한 프로토콜 설정(https)
  + 즉 클라이언트와 프록시는 `https`로 통신했는데 프록시 서버와 내부 서버는 `http`로 통신한다면 다시 클라이언트에게 전달할 때는 
    `https`로 전달해야하기 때문에 설정해야합니다.
  
[엔진엑스 프록시 모듈 설정](https://12bme.tistory.com/367)