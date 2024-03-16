# 인증서 자동 갱신  
  
자동으로 인증서를 갱신하는 기법으로 각자 환경에 맞게 테스트한 후 일부분만 수정하면 됩니다.  
  
리눅스 스케줄러 종류
1. cron
2. at
3. systemd.timer
  
  
   
cron 과 at 사용 방법 설명 :

systemd.timer 공식문서 : [공식문서](https://www.freedesktop.org/software/systemd/man/latest/systemd.timer.html),[예제](https://umount.net/migrating-cron-jobs-to-systemd-timer/)  
   
```Actionscript
crontab -e

PATH=/usr/local/bin
* * * * * docker-compose -f /home/xxx/docker-compose.yml restart certbot >> 
/home/xxx/cron.log 2>&1
```  
  
+ PATH: crontab은 독립적으로 실행되므로, 환경변수를 사용하지 않음
+ MAILTO : [설정방법](https://www.lesstif.com/system-admin/cron-59343125.html)  
  + 작업이 실패했을 경우에만 메일을 전송하도록 설정할 수 있습니다.
  
  
  