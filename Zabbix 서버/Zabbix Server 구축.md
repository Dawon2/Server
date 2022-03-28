# Zabbix Server 구축

## 구축 목적
: Agent서버의 네트워크 하드웨어를 감시, 추적하여 장애발생을 신속하게 알리기 위한 '모니터링' 환경 구축

### IP
Zabbix Server : 172.17.124.244
Zabbix Agent - Centos : 172.17.124.243
Zabbix Agent - Window : 172.17.124.198

## Zabbix의 동작 방식
1. Passive 방식 ( Default )
: Server -> Agent 로 데이터를 요청하여 응답받는 방식 ( 수집 )
- 목적지가 Agent 이므로 10050 포트 사용
- 수동적 Agent ( 데이터를 달라하면 줌 )

2. Active 방식
: Agent -> Server 로 데이터를 전달해주는 방식 ( 전송 )
- 목적지가 Server 이므로 10051 포트 사용
- Agent 설정에서 ServerActive의 IP를 지정하여 데이터를 전송하는 방식
- 능동적 Agent ( 알아서 데이터를 보냄 )

***

## Server 구축
```
# hostnamectl set-hostname server

# mv /etc/localtime /etc/localtime.ori
# ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime
( TimeZone 설정 )

# vi /etc/selinux/config
SELINUX=disabled

# yum -y install net-tools
# yum -y install httpd mariadb mariadb-devel mariadb-server

# rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
# yum clean all
# yum -y install zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf zabbix-agent

# yum -y install centos-release-scl
# vi /etc/yum.repos.d/zabbix.repo
------------------------------
[zabbix-frontend]
...
enabled=1
...
------------------------------
# yum -y install zabbix-web-mysql-scl zabbix-apache-conf-scl
( 프론트엔드 설정 )

# systemctl start httpd mariadb
# systemctl enable httpd mariadb
# systemctl stop firewalld
# systemctl disable firewalld

# mysql -u root -p
> create database zabbix character set utf8 collate utf8_bin;
> grant all privileges on zabbix.* to zabbix@localhost identified by 'zabbix';
> flush privileges;
> quit;
( DB 설정 )

# zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
( Zabbix 서버 호스트에서 초기 스키마 및 데이터를 가져옵니다 )

# vi /etc/zabbix/zabbix_server.conf
38 LogFile=/var/log/zabbix/zabbix_server.log
72 PidFile=/var/run/zabbix/zabbix_server.pid
91 DBHost=localhost
100 DBName=zabbix
116 DBUser=zabbix
124 DBPassword=zabbix

# vi /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf
php_value[date.timezone] = Asia/Seoul
( 주석 해제 및 아시아 서울로 변경 )

# systemctl start zabbix-server zabbix-agent rh-php72-php-fpm
# systemctl enable zabbix-server zabbix-agent rh-php72-php-fpm
```

- **오류 발생시 vi /var/log/zabbix/zabbix_server.log 에서 zabbix log 확인**

- **재부팅 한번 해주어야 server 포트가 정상적으로 올라옴**

***

## TEST
1. Zabbix server(10051), agent(10050), mysql(3306), httpd(80) 다 정상적으로 포트 올라와있는지 확인
```
# netstat -plunt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      846/sshd            
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1187/master         
tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN      860/zabbix_agentd   
tcp        0      0 0.0.0.0:10051           0.0.0.0:*               LISTEN      12691/zabbix_server 
tcp        0      0 127.0.0.1:9000          0.0.0.0:*               LISTEN      850/php-fpm: master 
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      8428/mysqld         
tcp6       0      0 :::80                   :::*                    LISTEN      8204/httpd          
tcp6       0      0 :::22                   :::*                    LISTEN      846/sshd            
tcp6       0      0 ::1:25                  :::*                    LISTEN      1187/master         
tcp6       0      0 :::10050                :::*                    LISTEN      860/zabbix_agentd   
tcp6       0      0 :::10051                :::*                    LISTEN      12691/zabbix_server 
```

2. 홈페이지에 172.17.124.244/zabbix ( Zabbix Server IP ) 입력 후 정상 접속 확인

- 정상접속 확인 후 Zabbix WEB 설정

1. Next step

2. 전부 OK 확인 후 Next step

3. Database type : MySQL
   Database host : localhost
   Database port : 0
   Database name : zabbix
   User : zabbix
   password : zabbix

4. Zabbix server name : Zabbix monitoring오후 6:04 2022-02-15

5. 초기 로그인 값
Username : Admin
Password : zabbix

6. Administration -> Users -> Admin
Language 언어설정 및 Theme 스킨 테마 설정

7. 모니터링 -> 호스트
Zabbix server의 상태에 초록불이 들어오면 서버 환경 구성 완료

***
**★ 정상적으로 작동된다면 Keepalive 구축 완료 ! ★**
***

## 참고
- https://cloudest.oopy.io/posting/003
- https://www.zabbix.com/download?zabbix=5.0&os_distribution=centos&os_version=7&db=mysql&ws=apache
- https://foxydog.tistory.com/15
