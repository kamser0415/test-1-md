# 웹서버(WAS)  

## 외장서버
![image_178.png](image_178.png)  
  
+ 스프링 부트 빌드를 `war` 형식으로 빌드
+ 서버에 설치된 `Tomcat` 내부에 빌드 파일을 배포
+ `Tomcat`이 `war`파일을 압축해제하여 사용
  
**WAR** 파일 내부에는 애플리케이션의 소스 코드, 리소스, 설정 파일등이 포함되어있습니다.  

## 내장서버  
![image_179.png](image_179.png)  
  
+ 서버에 실행할 `jar` 파일을 실행하면 된다.
+ `was`설치나 연동하는 과정이 필요가 없다.    

[내장 톰캣과 외장 톰캣의 차이](https://buly.kr/FsG7lja)
[임베디드 톰캣이란](https://www.theserverside.com/definition/embedded-Tomcat)  
  
## 비교  
독립형(외장) 서버를 사용하면 장점은 `host`에 따라 다른 루트 컨택스트를 설정하여 하나의 `WAS`로 여러 애플리케이션을 운영할 수 있습니다.
여러 애플리케이션의 서버 환경이나 구성을 `server.xml` 파일 하나로 관리할 수 있습니다. 또한 톰캣 서버 하나로 가동되기 때문에 톰캣 프로세스에 대한 리소스를 절약할 수 있습니다.
  
임베디드 톰캣 서버를 사용하면 장점은 애플리케이션 마다 각 다른 서버 설정을 스프링 부트 설정으로 간단하게 할 수 있습니다.  
또한 개별 톰캣으로 동작하기 때문에 리소스가 독립되어 애플리케이션의 성능을 보장할 수 있습니다.  
그리고 서버에 문제가 발생할 경우 독립되어 다른 개별 서비스 어플리케이션은 영향을 받지 않습니다.  
다만 애플리케이션 마다 톰캣을 실행하기 때문에 리소스가 독립형으로 실행될 때보다 더 필요합니다.  

## 스프링 부트 서버 설정
[서버 propertis](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#appendix.application-properties.server)