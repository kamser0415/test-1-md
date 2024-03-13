# 클론코딩-실제 서비스 구축하기  
  
워드프로세스는 서버 내부안에 워드프레스 관련된 파일이나 프로그램이 있기 때문에 자칫 리버스 프록시에서 경로를 수정하여 요청할 경우 
이후 코드들이 `Nginx`에 해당 요청을 할 수 있기 때문에 자체로 돌아가는 워드프레스는 경로를 수정하지 않도록 합니다.  
  
워드프레스의 base 경로를 수정하는 방식을 사용하여 워드프레스와 `Nginx`가 충돌나지 않도록 합니다.  
  
## 구조
  
![image_148.png](image_148.png)  
  
## 서버 설정
```Actionscript
upstream docker-wordpress {
    server wordpress:80;
}

upstream docker-web {
    server nginx:80;
}

server {
    location /blog/ {
        #rewrite ^/blog(.*)$ $1 break;
        proxy_pass         http://docker-wordpress;
        proxy_redirect     off;
        proxy_set_header   Host $host;
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Host $server_name;
    }
    
    location / {
        proxy_pass         http://docker-web;
        proxy_redirect     off;
        proxy_set_header   Host $host;
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Host $server_name;
    }
}
```
`pwd = ./etc/nginx/nginx.conf`  
서버 설정을 확인해보면 `rewrite`를 하지 않고 그대로 `/blog`로 워드프로세스 서버에 `path`가 전달됩니다.  

워드프레스는 기본 폴더가 `/var/www/html`에 정보가 저장되기 때문에 `/blog`로 들어왔을때 워드프레스가 응답을 안하기 때문에
기본 폴더의 위치를 path 위치로 변경합니다.  
  
```Actionscript
#pwd /var/www/html
mkdir blog
mv * blog/ # 모두 이동됩니다.
```  
이때 오류가 발생하는데 blog 폴더안에 blog폴더로 이동하려고 하기 때문에 오류가 발생하는 것이기 때문에 
신경쓰지 않아도 됩니다.

## 워드프레스 설정
```Actionscript
FROM wordpress:5.7.0

#1 RUN mkdir -p /usr/src/blog
#2 RUN mkdir -p /usr/src/blog/wp-content/plugins
#3 RUN mkdir -p /usr/src/blog/wp-content/uploads
#4 RUN cp -rf /usr/src/wordpress/* /usr/src/blog
#5 RUN mv /usr/src/blog /usr/src/wordpress/
#6 RUN chown -R www-data:www-data /usr/src/wordpress
#7 RUN find /usr/src/wordpress/blog/ -type d -exec chmod 0755 {} \;
#8 RUN find /usr/src/wordpress/blog/ -type f -exec chmod 644 {} \;
```  

### chown  
리눅스는 파일마다 소유자와 그룹이 존재하기 때문에 `chown`을 통해
파일이나 디렉토리의 소유자와 그룹을 변경하는 명령어입니다.  
  
+ `-R` 명령어는 하위 디렉토리까지 포함되는 명령어입니다.  
  
정리하면 `/usr/src/wordpress` 폴더 하위까지 소유자와 그룹을 `www-data`로 변경하는 명령어입니다.  
이 명령어를 사용하는 이유가 워드프레스가 실행될때 `www-data` 계정을 사용하기 때문에 
웹 서버의 권한으로 파일을 읽고 쓰는 작업을 위해 변경합니다.  
  
### find ~ 
`find "find를 시작할 디렉토리 경로" -type d -exec`  
`-type d`: 폴더를 찾는다는 의미
`-type f`: 파일을 찾는다는 의미  
`-exec`: find로 찾은 파일이나 폴더에 실행할 명령이 뒤에 옵니다.  
  
`chmod 0755 {} /;`: `{}`는 find로 찾은 폴더나 파일을 의미합니다. `/`는 `-exec`의 명령어 종료를 의미합니다.  
  
예를 들어,  
`find . -type f -name "*.txt" -exec echo {} \;`   
위 명령어를 실행하면 `txt`확장자를 가진 모든 파일의 경로를 출력합니다.
```Actionscript
root@8a7590871725:/# find . -type f -name "*.txt" -exec echo {} \;
find: './proc/57/task/57/fdinfo': Permission denied
find: './proc/57/map_files': Permission denied
find: './proc/57/fdinfo': Permission denied
find: './proc/58/task/58/fdinfo': Permission denied
find: './proc/58/map_files': Permission denied
```  

