# WEB-WAS 연동
* WEB : apache , WAS : tomcat
* WEB WAS 각각 서버 생성하여 apache , tomcat 설치 후 진행
* 8009 Port Open 필요

***

## [ 구축 목적 ]
1. WEB 서버로 들어오는 트래픽에 대하여 정적 페이지 + 동적 페이지 모두 응답하기 위하여
2. Tomcat 만으로도 정적+동적 응답이 가능하지만 효율적인 부하분산을 통한 사용을 위하여

***
## [ WEB / WAS 연동 ]


### [ 1. WEB 서버 설정 ]
```
# wget https://mirror.navercorp.com/apache/tomcat/tomcat-connectors/jk/tomcat-connectors-1.2.49-src.tar.gz
( mod_jk 다운로드 )

# tar xvfz tomcat-connectors-1.2.49-src.tar.gz -C /usr/local/src
# cd /usr/local/src/tomcat-connectors-1.2.49-src/
# cd native
# ./buildconf.sh
( apr , apr-util 등의 라이브러리를 빌드 )
# ./configure --with-apxs=/bin/apxs
# make && make install

# ll /usr/local/apache2/modules/mod_jk.so
( 설치 확인 )

# vi /usr/local/apache2/conf/httpd.conf
---
LoadModule version_module modules/mod_version.so
LoadModule jk_module modules/mod_jk.so

<IfModule jk_module>
 JkWorkersFile /usr/local/apache2/conf/workers.properties
 JkLogFile /usr/local/apache2/logs/mod_jk.log
 JkLogLevel info
 JkLogStampFormat "[%a %b %d %H:%M:%S %Y]"
 JkShmFile /usr/local/apache2/logs/mod_jk.shm
 JKMount /* worker1
</IfModule>
---
( mod_jk 관련 설정 추가 )

# vi /usr/local/apache2/conf/workers.properties
---
# 노드 이름
worker.list=worker1

# 톰캣의 AJP 포트 정보 (어떤 AJP 요청을 어떤 포트로 수락하는지)
worker.worker1.type=ajp13
worker.worker1.port=8009

# 톰캣 호스트 IP (톰캣 인스턴스가 실행중인 IP 주소)
worker.worker1.host=172.16.10.9
---
( 연동할 Tomcat 정보가 담긴 파일 생성 )

# systemctl restart httpd
# systemctl status httpd
# ps -ef | grep httpd
```

***

### [ 2. WAS 서버 설정 ]
```
# vi /opt/apache-tomcat-8.5.96/conf/server.xml
---
    <Connector protocol="AJP/1.3"
               address="0.0.0.0"
               port="8009"
               redirectPort="8443"
               maxParameterCount="1000"
               secretRequired="false"
               />
---
( AJP 포트 허용 - 주석 해제 필요 )

# systemctl restart tomcat
# systemctl status tomcat
# ps -ef | grep tomcat
# netstat -plunt | grep 8009
( 8009 Port 올라왔는지 확인 )
```


### [ 3. 웹 접속 확인 ]
> 인터넷에 WEB 서버 아이피 입력하여 정상 접속 확인
( http://[WEB SERVER IP] )

***
**★ 정상적으로 작동된다면 WEB / WAS 연동 완료 ! ★**
***

## 참조
- https://velog.io/@sherlockid8/aphache-tomcat-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0
- https://seul96.tistory.com/347