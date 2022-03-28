# HAproxy 서버 구축

## 구축 목적
: 2개의 WEB서버의 트래픽을 효율적으로 분산하기 위하여 ( 로드밸런싱 )

### IP
VIP : 172.17.124.240 - Keepalived
HA 1 : 172.17.124.241
HA 2 : 172.17.124.242
WEB 1 : 172.17.124.243
WEB 2 : 172.17.124.244

## 완전 기본서버일때만 설정

### 기본 네트워크 설정
```
# vi /etc/sysconfig/network-scripts/ifcfg-eth0

ex)
TYPE=Ethernet
BOOTPROTO=static
NAME=eth0
DEVICE=eth0
ONBOOT=yes
IPADDR=172.17.124.241
GATEWAY=172.17.124.1
NETMASK=255.255.255.0

# vi /etc/resolv.conf
nameserver 8.8.8.8
```

### 기본 시간 설정
```
# yum -y install ntp
# vi /etc/ntp.conf
21 server time.google.com iburst
( 나머지 server 설정 삭제 - google에 ntp서버에서 시간정보 받아오기 )

# systemctl restart ntpd
# systemctl status ntpd
# systemctl enable ntpd
# ntpq -p ( 잘 들어갔는지 확인 )
# date ( 확인 )
```

## HAProxy 구축
```
# yum -y install gcc openssl openssl-devel pcre-static pcre-devel systemd-devel
# yum -y install wget

# mkdir /HAproxy
# cd /HAproxy

# wget http://www.haproxy.org/download/2.5/src/haproxy-2.5.1.tar.gz
( haproxy 공식 홈페이지에서 버전 확인후 링크 주소 복사 )
# tar xvfs haproxy-2.5.1.tar.gz

# cd haproxy-2.5.1
# make TARGET=linux-glibc USE_OPENSSL=1 USE_PCRE=1 USE_ZLIB=1 USE_SYSTEMD=1
# make install

# curl "http://git.haproxy.org/?p=haproxy-2.3.git;a=blob_plain;f=contrib/systemd/haproxy.service.in" -o /etc/systemd/system/haproxy.service
# vi /etc/systemd/system/haproxy.service
ExecStartPre=/usr/local/sbin/haproxy -f $CONFIG -c -q $EXTRAOPTS
ExecStart=/usr/local/sbin/haproxy -Ws -f $CONFIG -p $PIDFILE $EXTRAOPTS
ExecReload=/usr/local/sbin/haproxy -f $CONFIG -c -q $EXTRAOPTS
( 해당 부분 수정 )

# mkdir /etc/haproxy
# mkdir /etc/haproxy/certs
# mkdir /etc/haproxy/errors
# mkdir /var/log/haproxy
# cd ./examples/errorfiles/
# cp ./*.http /etc/haproxy/errors

# useradd -c "HAproxy Daemon User" -s /sbin/nologin haproxy
# tail -1 /etc/passwd

# vi /etc/rsyslog.d/haproxy.conf
--------------------------------------------------------
$ModLoad imudp
$UDPServerAddress 127.0.0.1
$UDPServerRUN 514

local0.* /var/log/haproxy/haproxy_traffic.log

# vi /etc/logrotate.d/haproxy
/var/log/haproxy/haproxy_traffic.log {
        daily
        rotate 30
        create 0600 root root
        compress
        notifempty
        missingok
        postrotate
                /bin/systemctl restart rsyslog.service > /dev/null 2> /dev/null || true
        endscript
}
--------------------------------------------------------

# vi /etc/haproxy/haproxy.cfg
--------------------------------------------------------
global
	daemon
	maxconn 4000
	user haproxy
	group haproxy
	log 127.0.0.1:514 local0

defaults
	mode http
	option redispatch
	retries 3
	log global
	option httplog
	option dontlognull
	option dontlog-normal
	option http-server-close
	option forwardfor

maxconn 3000
	timeout connect 10s
	timeout http-request 10s
	timeout http-keep-alive 10s
	timeout client 1m
	timeout server 1m
	timeout queue 1m

	errorfile 400 /etc/haproxy/errors/400.http
	errorfile 403 /etc/haproxy/errors/403.http
	errorfile 408 /etc/haproxy/errors/408.http
	errorfile 500 /etc/haproxy/errors/500.http
	errorfile 502 /etc/haproxy/errors/502.http
	errorfile 503 /etc/haproxy/errors/503.http
	errorfile 504 /etc/haproxy/errors/504.http

frontend proxy
	bind *:80
	default_backend WEB_SRV_list

backend WEB_SRV_list
	balance roundrobin
	option httpchk HEAD /
	http-request set-header X-Forwarded-Port %[dst_port]
	#cookie SRVID insert indirect nocache maxlife 10m
	#server WEB_01 172.17.124.243:80 cookie WEB_01 check inter 3000 fall 5 rise 3
	#server WEB_02 172.17.124.244:80 cookie WEB_02 check inter 3000 fall 5 rise 3
	server WEB_01 172.17.124.243:80 check inter 3000 fall 5 rise 3
	server WEB_02 172.17.124.244:80 check inter 3000 fall 5 rise 3

listen stats
	bind *:9000
	stats enable
	stats realm Haproxy Stats Page
	stats uri /
	stats auth admin:haproxy1
--------------------------------------------------------

# haproxy -f /etc/haproxy/haproxy.cfg -c
( 잘 설정되었는지 확인하는 명령어, yum haproxy 설치해야 사용 가능 )

# systemctl start haproxy
# systemctl enable haproxy
```

## TEST
1.
WEB에서 HAproxy 주소를 입력하였을때 정상적으로 로드밸런싱 되는지 확인

***
**★ 정상적으로 작동된다면 HAproxy 구축 완료 ! ★**
***

## 참고
설정파일 찾는곳 : https://cbonte.github.io/haproxy-dconv/2.3/configuration.html#3.1
	          https://www.haproxy.com/blog/the-four-essential-sections-of-an-haproxy-configuration/
