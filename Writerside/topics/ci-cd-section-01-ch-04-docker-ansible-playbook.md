# ci-cd-section-01-ch-04-docker-ansible-playbook

Ansible 명령어를 통해 인벤토리에 저장된 서버에 동일한 작업을 커맨드로 실행할 수 있습니다.  
  
## AnsiblePlaybook 이란 ?
사용자가 원했던 내용을 미리 작업해놓은 스크립트 파일입니다.  
여러 작업을 해야한다면 여러번 커맨드를 실행해야합니다.
  
![image_58.png](image_58.png)  
  
```Console
# Ansible 플레이북 예시: 웹 서버 설정

# 이름: 웹 서버 구성
# 호스트: web_servers 그룹
# 역할: Apache 웹 서버 설치 및 구성

- name: Configure web servers # 엔서블 플레이북 이름
  hosts: web_servers # ip 이름, 적용시킬 그룹명
  become: yes
  
  tasks: # 해당 hosts에 어떤 작업을 진행할 지에 대한 설명입니다.
    - name: Install Apache web server 
      yum:
        name: httpd
        state: present
      tags:
        - install
        - apache

    - name: Add custom configuration to Apache # 작업 이름을 작성
      blockinfile: # 실질적인 작업
      # 특정한 블럭을 만들어서 어떤 내용을 추가할 때 사용합니다.
        path: /etc/httpd/conf/httpd.conf # 어떤 파일에 내용을 추가할지 작성
        block: | # 파이프라인을 꼭 입력해야합니다!
          # Custom configuration block
          # 아파치 설정파일을 열어서 아래 내용을 추가한 것을 확인할 수 있습니다.
          # blockinfile을 추가하면 이미지 처럼 블럭이 추가됩니다.
          Listen 8080
          <VirtualHost *:8080>
              ServerAdmin webmaster@example.com
              DocumentRoot /var/www/html
              ErrorLog logs/example.com-error_log
              CustomLog logs/example.com-access_log common
          </VirtualHost>
        create: yes
      tags:
        - apache
```  

## 작성하기
```Console
# pwd /root
[root@51144f7a4712 ~]# vi first-playbook.yml

[root@51144f7a4712 ~]# [root@51144f7a4712 ~]# cat first-playbook.yml
---
- name: Add an ansible hosts
  hosts: localhost
  tasks:
    - name: Add an ansible hosts
      blockinfile:
        path: /etc/ansible/hosts
        block: |
          [mygroup]
          172.17.0.5
```  

## 플레이북 실행하기
```Console
[root@51144f7a4712 ~]# [root@51144f7a4712 ~]# ansible-playbook first-playbook.yml

PLAY [Add an ansible hosts] *************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************
ok: [localhost]

TASK [Add an ansible hosts] *************************************************************************************************************************************************************
changed: [localhost]

PLAY RECAP ******************************************************************************************************************************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```  
이렇게 미리 작업할 내용을 스크립트로 작성하고 한번에 실행할 수 있습니다.  
  
## 확인하기
```Console
[root@51144f7a4712 ~]# [root@51144f7a4712 ~]# cat /etc/ansible/hosts
[devops]
172.17.0.3
172.17.0.4
# BEGIN ANSIBLE MANAGED BLOCK
[mygroup]
172.17.0.5
# END ANSIBLE MANAGED BLOCK
```  
Ansible은 멱등성으로 여러번 실행해도 동일한 결과가 발생합니다. 
만약 여기서 hosts 파일을 약간 수정한다면 어떻게 될까요?  
```Console
[root@51144f7a4712 ~]# cat /etc/ansible/hosts
[devops]
172.17.0.3
172.17.0.4
# BEGIN ANSIBLE MANAGED BLOCK
[mygroup]
172.17.0.5
# BEGIN ANSIBLE MANAGED BLOCK
[mygroup]
172.17.0.5
# END ANSIBLE MANAGED BLOCK
[mygroup]
172.17.0.5
# 2END ANSIBLE MANAGED BLOCK
```  
추가하려는 텍스트가 동일하다면 위치에 상관없이 멱등성이 보장됩니다. 
약간의 텍스트가 수정되면 멱등성은 당연히 보장되지 않습니다.  
  
## 플레이북 정리  
![image_59.png](image_59.png)  
  
ansible 커맨드 명령어로 ansible hosts 파일에 작성된 그룹이나 전체 서버에 동일한 작업을 할 수 있습니다.  
이제 작성할 커맨드 명령어를 yaml 파일에 작성하여 playbook으로 등록하여 사용하면 
다양한 작업을 미리 작성된 스크립트를 동해 명령할 수 있습니다.

## 예제-파일 복사
```Console
- name: Ansible Copy Example Local to Remote
  hosts: devops
  tasks: 
    - name: copying file with playbook
      copy: 
        src: ~/sample.txt # 전송하려는 파일 주소
        dest: /tmp # 타켓 위치
        owner: root 
        mode: 0644
```  
`~`: 틸드는 현재 디렉토리, 홈 디렉토리를 뜻합니다.  

owner
: "owner"는 파일 또는 디렉토리의 소유자의 이름이며, 이는 chown 명령에 전달되는 것과 같은 형식으로 사용됩니다.  

mode
: 0644는 소유자에게는 읽기와 쓰기 권한만 부여하고, 그 외의 사용자에게는 읽기만 부여하는 퍼미션입니다.  
  
## 예제-톰캣설치 및 압축해제
```Console
- name: Download Tomcat9 from tomcat.apache.org
  hosts: all
  tasks:
   - name: Create a Directory /opt/tomcat9
     file:
       path: /opt/tomcat9
       state: directory
       mode: 0755
   - name: Download Tomcat using get_url
     get_url:
       url: https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.85/bin/apache-tomcat-9.0.85.tar.gz
       dest: /opt/tomcat9
       mode: 0755
       checksum: sha512:https://downloads.apache.org/tomcat/tomcat-9/v9.0.85/bin/apache-tomcat-9.0.85.tar.gz.sha512
   - name: arachive tomcat
     ansible.builtin.unarchive:
       src: /opt/tomcat9/apache-tomcat-9.0.85.tar.gz
       dest: /opt/tomcat9
```  

## 젠킨스에서 Ansible 사용하기

