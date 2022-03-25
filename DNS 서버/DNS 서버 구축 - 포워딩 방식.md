# DNS 서버 구축 - 포워딩 방식

## 구축 목적
: 클라이언트가 도메인으로 접속했을때 그에 맞는 IP를 찾아서 전달해주기 위해

## DNS 구축
```
# yum -y install bind-*

# vi /etc/named.conf
12 options {
13         listen-on port 53 { any; };
( DNS 포트인 53번 포트와 접근 할 수 있는 IP 설정 - any : 모든 IP 지정 )
21         forward only;
( only : forwarders 서버들이 응답하지 않을 경우 다른 서버에게 요청을 보내지 않는다. -> 포워딩서버만 사용 )
22         forwarders { 8.8.8.8; };
( 도메인에 대한 질의를 8.8.8.8 서버로 넘긴다 )
23         allow-query     { any; };
( DNS 서버의 쿼리를 허용할 IP 대역 설정 - any : 모든 IP 지정 )

* 포워딩 설정
Forwarding을 설정하여 해당 네임 서버가 직접 도메인 정보를 찾는것이 아니라, 
클라이언트의 모든 요청을 Forwarding에 지정된 서버로 전달하여 해당 서버가 요청을 처리하게 설정한다.


# vi /etc/resolv.conf
3 nameserver 127.0.0.1
( 1번째로 자기자신 DNS 서버 IP에서 찾기 )
4 nameserver 8.8.8.8
( 1번째에서 없으면 2번째 구글 DNS서버에서 찾기 )

* /etc/resolv.conf 는 도메인정보를 어디서 찾아올건지 정해주는 곳!
  + 도메인을 찾을때는 /etc/hosts 파일을 먼저 확인하고 없으면 resolv.conf 를 찾는 순서로 진행 됨

# systemctl start named
# systemctl enable named

```
* * *
- TEST -
```
1.
DNS 포트 잘 올라와있는지 확인 

# netstat -plunt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 172.27.1.13:53          0.0.0.0:*               LISTEN      7301/named          
tcp        0      0 127.0.0.1:53            0.0.0.0:*               LISTEN      7301/named          
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      824/sshd            
tcp        0      0 127.0.0.1:953           0.0.0.0:*               LISTEN      7301/named          
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1267/master         
tcp6       0      0 ::1:53                  :::*                    LISTEN      7301/named          
tcp6       0      0 :::22                   :::*                    LISTEN      824/sshd            
tcp6       0      0 ::1:953                 :::*                    LISTEN      7301/named          
tcp6       0      0 ::1:25                  :::*                    LISTEN      1267/master         
udp        0      0 172.27.1.13:123         0.0.0.0:*                           2445/ntpd           
udp        0      0 127.0.0.1:123           0.0.0.0:*                           2445/ntpd           
udp        0      0 0.0.0.0:123             0.0.0.0:*                           2445/ntpd           
udp        0      0 0.0.0.0:34437           0.0.0.0:*                           756/dhclient        
udp        0      0 0.0.0.0:5353            0.0.0.0:*                           541/avahi-daemon: r 
udp        0      0 0.0.0.0:54211           0.0.0.0:*                           541/avahi-daemon: r 
udp        0      0 172.27.1.13:53          0.0.0.0:*                           7301/named          
udp        0      0 127.0.0.1:53            0.0.0.0:*                           7301/named          
udp        0      0 0.0.0.0:68              0.0.0.0:*                           756/dhclient        
udp6       0      0 fe80::62ff:fe03:62:123  :::*                                2445/ntpd           
udp6       0      0 ::1:123                 :::*                                2445/ntpd           
udp6       0      0 :::123                  :::*                                2445/ntpd           
udp6       0      0 :::29873                :::*                                756/dhclient        
udp6       0      0 ::1:53                  :::*                                7301/named          

2.
nslookup 명령어로 DNS레코드에 잘 들어가있는지 확인

[ 리눅스 - 서버 ]
# nslookup
> server
Default server: 127.0.0.1
Address: 127.0.0.1#53
Default server: 8.8.8.8
Address: 8.8.8.8#53
> www.google.com
Server:         127.0.0.1
Address:        127.0.0.1#53

Non-authoritative answer:
Name:   www.google.com
Address: 142.250.196.132
Name:   www.google.com
Address: 2404:6800:4004:81d::2004

[ 윈도우 - 클라이언트 ]
# nslookup
> server 211.251.236.200
기본 서버:  [211.251.236.200]
Address:  211.251.236.200

> www.google.com
서버:    [211.251.236.200]
Address:  211.251.236.200

권한 없는 응답:
이름:    www.google.com
Addresses:  2404:6800:4004:81d::2004
          172.217.175.36


3. 
클라이언트에서 DNS 서버를 211.251.236.200 으로 바꿔서 접속 테스트

ncpa.cpl -> 이더넷 -> ipv4 -> 기본 DNS 211.251.236.200

-> 구글 접속 테스트


* 안되면 클라이언트와 서버의 방화벽 설정 확인 , DNS conf 확인
```

★ 정상적으로 작동된다면 DNS 서버 포워딩방식 설정 완료 ! ★

* * *

참조 : https://nirsa.tistory.com/108?category=872350
        https://chhanz.github.io/linux/2020/11/06/configuration-dns/
        https://aidencom.tistory.com/57
