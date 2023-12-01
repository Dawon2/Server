# Tomcat 컴파일 설치

* JAVA(openjdk)가 있는지 확인 후 없으면 아래 JAVA 설치부터 진행
> java -version 명령어로 확인

### [ 0. JAVA 설치 ]
```
* java 버전을 업그레이드 할때도 동일

wget https://download.java.net/java/GA/jdk21.0.1/415e3f918a1f4062a0074a2794853d0d/12/GPL/openjdk-21.0.1_linux-x64_bin.tar.gz
( 해당 사이트에서 openjdk 최신버전 확인 후 다운로드 - Priv 서버일 시 SFTP 사용 )

# tar xvfz openjdk-21.0.1_linux-x64_bin.tar.gz -C /usr/local/src
( 원하는 경로 지정하여 압축 해제 )

# cd /usr/local/src/
# mv jdk-21.0.1/ jdk-21

# vi /etc/profile
---
JAVA_HOME=/usr/local/src/jdk-21
PATH=$PATH:$JAVA_HOME/bin
CLASSPATH=.:JAVA_HOME/lib/tools.jar

export JAVA_HOME PATH CLASSPATH
---
# source /etc/profile
( 설정 적용 )

# ll /usr/bin/java -> /etc/alternatives/java
# ll /etc/alternatives/java -> /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.352.b08-2.el8_6.x86_64/jre/bin/java
( 업그레이드 시 - 위와 같이 기존 java 링크로 등록되어 있어, 링크 삭제 후 재 등록 필요 )

# cd /etc/alternatives/java
* 업그레이드 시 - # rm java

# ln -s /usr/local/src/jdk-21/bin/java java
# ll /etc/alternatives/java -> /usr/local/src/jdk-21/bin/java
( 링크 설정 )

# java -version
( 신규설치 or 업그레이드 된 java 버전 확인 )

```

***

### [ 1. Tomcat 컴파일 설치 ]
```
# wget https://dlcdn.apache.org/tomcat/tomcat-8/v8.5.96/bin/apache-tomcat-8.5.96.tar.gz
( 해당 사이트에서 tomcat 최신버전 확인 후 다운로드 - Priv 서버일 시 SFTP 사용 )

# tar xvfz apache-tomcat-8.5.96.tar.gz -C /opt/
( 경로 지정 후 압축 해제 )

# /opt/apache-tomcat-8.5.96/startup.sh
( tomcat 실행 )
# /opt/apache-tomcat-8.5.96/shutdown.sh
( tomcat 셧다운 )

# useradd tomcat
# chown -R /opt/apache-tomcat-8.5.96

# cd /etc/systemd/system
# vi tomcat.service
---
[UNIT]
Description=tomcat
After=syslog.target network.target

[Service]
Type=forking

Environment=/opt/apache-tomcat-8.5.96
ExecStart=/opt/apache-tomcat-8.5.96/bin/startup.sh
ExecStop=/opt/apache-tomcat-8.5.96/bin/shutdown.sh

User=tomcat
Group=tomcat

[Install]
WantedBy=multi-user.target
---

# su - tomcat

# systemctl start tomcat
# systemctl status tomcat
# ps -ef | grep tomcat
# netstat -plunt
( 프로세스 잘 떴는지 , Port 잘 올라왔는지 확인 )
```

### [ 2. 웹 접속 확인 ]
> 인터넷에 아이피 입력하여 정상 접속 확인
( http://[IP]:8080 )

***
**★ 정상적으로 작동된다면 Tomcat 설치 완료 ! ★**
***

## 참조
- https://velog.io/@es_seong/Tomcat-CentOS-7-%ED%86%B0%EC%BA%A3-%EC%9E%AC%EC%84%A4%EC%B9%98%EB%B2%84%EC%A0%84%EC%97%85
- https://veneas.tistory.com/entry/Linux-CentOS7-%EC%95%84%ED%8C%8C%EC%B9%98-%ED%86%B0%EC%BA%A3apache-tomcat-%EC%84%A4%EC%B9%98