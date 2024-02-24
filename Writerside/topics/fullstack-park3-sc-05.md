# 리눅스용 도커 설치
  
## 도커 엔진 설치 
[공식 홈페이지 참고](https://docs.docker.com/engine/install/ubuntu/)

1. 도커 과거 버전 삭제
2. apt 리포지토리를 사용하기
    ```Bash
    # Add Docker's official GPG key:
    sudo apt-get update
    sudo apt-get install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
    
    # Add the repository to Apt sources:
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    ```
    - `sudo apt-get update`: 시스템 패키지 관리자인 apt의 패키지 목록을 업데이트합니다.
    - `sudo apt-get install ca-certificates curl`: 필요한 패키지(ca-certificates와 curl)를 설치합니다. ca-certificates는 인증서 관련 도구이고, curl은 데이터 전송 도구입니다.
    
    - `sudo install -m 0755 -d /etc/apt/keyrings`: `/etc/apt/keyrings` 디렉토리를 생성하고, 해당 디렉토리에 대한 권한을 설정합니다. 여기서 `-m 0755`는 디렉토리의 퍼미션을 나타냅니다.
    
    - `sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc`: Docker의 공식 GPG 키를 다운로드하여 `/etc/apt/keyrings/docker.asc` 파일에 저장합니다.
    
    - `sudo chmod a+r /etc/apt/keyrings/docker.asc`: Docker의 GPG 키를 읽기 가능하게 권한을 설정합니다.
    
   - ```bash
     echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
     ```: Docker의 Apt 저장소를 `/etc/apt/sources.list.d/docker.list` 파일에 추가합니다. 이 저장소는 `https://download.docker.com/linux/ubuntu` 주소를 사용하며, 시스템의 아키텍처와 Ubuntu의 버전 코드명을 자동으로 설정합니다.
    
   - `sudo apt-get update`: Docker 저장소가 추가된 후에 다시 패키지 목록을 업데이트합니다.  

3. apt install로 도커를 설치합니다.  
4. 도커 설치를 확인하기 위해 `hello-world`를 실행해봅니다.  
  
## sudo 명령 없이 docker 명령어 사용하기 설정
1. 현 사용자(`GCP는 이메일 아이디`)를 docker group에 포함합니다.  
    ```Bash
    sudo usermod -aG docker ${USER} 
    ```
2. 터미널 끊고, 재접속 합니다 (재로그인으로 적용)
3. 현 ID가 docker gruop에 포함되어있는지 확인합니다.
    ```Bash
    id -nG 
    ```
4. 이제 sudo없이 실행되는지 확인합니다.
> 만약 sudo 없이 docker 명령이 안되면, 시스템 재부팅 후 재실행을 합니다.  
> `sudo systemctl reboot`  
>   
  
## docker-compose 설치  
[공식 홈페이지 참조](https://docs.docker.com/compose/install/standalone/)  
  
1. 다운로드 명령어 입력시 `sudo`를 추가합니다.
2. 리눅스에서 프로그램이 실행되려면 실행 권한이 필요합니다. 실행 권한을 설정합니다.
   ```bash
    # `sudo`: 명령어를 관리자 권한으로 실행합니다.
    # `chmod`: 파일의 권한을 변경하는 명령어입니다.
    # `+x`: 실행 권한을 추가하는 옵션입니다.
    # `/usr/local/bin/docker-compose`: 실행 가능한 파일로 만들고자 하는 파일의 경로입니다.
    
    sudo chmod +x /usr/local/bin/docker-compose
    ```
   도커 컴포즈가 설치된 위치에 실행권한(`x`) 을 변경합니다.
3. `docker-compose --version`으로 확인합니다.