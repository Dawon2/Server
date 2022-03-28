# Zabbix Agent 구축 - CentOS7

### IP
```
Zabbix Server : 172.17.124.244
Zabbix Agent - Centos : 172.17.124.243
Zabbix Agent - Window : 172.17.124.198
```
***

## Agent 구축
```
# cat /etc/*-release | uniq
( 버전 확인 )

# rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
# yum -y install zabbix-agent
# yum -y install net-tools

# vi /etc/zabbix/zabbix_agentd.conf

13 PidFile=/var/run/zabbix/zabbix_agentd.pid    [ 기본 설정 ]
32 LogFile=/var/log/zabbix/zabbix_agentd.log   [ 기본 설정 ]
117 Server=192.168.232.128        	           [ Zabbix Server의 IP 또는 호스트 이름 ]
158 #ServerActive=127.0.0.1		           [ 주석 처리 ]
169 Hostname=192.168.232.129                   [ Agent 설치 서버의 IP 또는 호스트 이름 ]

# systemctl start zabbix-agent
# systemctl enable zabbix-agent
```
***
## TEST
**1. 포트 잘 올라와있나 확인**
```
# netstat -plunt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      952/sshd            
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1055/master         
tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN      1351/zabbix_agentd  
tcp6       0      0 :::22                   :::*                    LISTEN      952/sshd            
tcp6       0      0 ::1:25                  :::*                    LISTEN      1055/master         
tcp6       0      0 :::10050                :::*                    LISTEN      1351/zabbix_agentd  
```
**2. 프로세스 확인 - listener까지 나오면 정상**
```
# ps -ef | grep zabbix
zabbix     1351      1  0  21 ?      00:00:00 /usr/sbin/zabbix_agentd -c /etc/zabbix/zabbix_agentd.conf
zabbix     1352   1351  0  21 ?      00:00:06 /usr/sbin/zabbix_agentd: collector [idle 1 sec]
zabbix     1353   1351  0  21 ?      00:00:01 /usr/sbin/zabbix_agentd: listener #1 [waiting for connection]
zabbix     1354   1351  0  21 ?      00:00:01 /usr/sbin/zabbix_agentd: listener #2 [waiting for connection]
zabbix     1355   1351  0  21 ?      00:00:01 /usr/sbin/zabbix_agentd: listener #3 [waiting for connection]
root       3419   1070  0 02:50 pts/0    00:00:00 grep --color=auto zabbix
```

- 정상 확인 후 Zabbix Web 설정

**1. 호스트 작성**
호스트명 : Linux Server - Agent
그룹 : Linux servers
Interfaces : 172.17.124.243 ( Agent 서버 IP )
IP 주소 : 포트 10050
템플릿 - Link new templates : Template OS Linux by Zabbix agent
추가

**2. 호스트에서 ZBX 초록불 확인**

***
**★ 정상적으로 작동된다면 Zabbix Agent Centos 구축 완료 ! ★**
***

## 참고
- https://cloudest.oopy.io/posting/002
- https://foxydog.tistory.com/16
