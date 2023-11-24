# Apache 컴파일 설치

### [ Apache 컴파일 설치 ]
```
* 아래 파일 설치 여부 확인 후 없으면 설치 진행
# yum install -y gcc gcc-c++ pcre-devel expat-devel

# cd /usr/local/src
( 소스파일 디렉터리 )
# wget https://dlcdn.apache.org/apr/apr-1.7.4.tar.gz
# wget https://dlcdn.apache.org/apr/apr-util-1.6.3.tar.gz
# wget https://github.com/PCRE2Project/pcre2/releases/download/pcre2-10.42/pcre2-10.42.tar.gz
# wget https://dlcdn.apache.org/httpd/httpd-2.4.58.tar.gz
( 최신 버전 확인 후 설치 )

# tar xvfz apr-1.7.4.tar.gz
# tar xvfz apr-util-1.6.3.tar.gz
# tar xvfz pcre2-10.42.tar.gz pcre2-10.42
# tar xvfz httpd-2.4.58.tar.gz
( 압축 해제 )

# cd pcre2-10.42
# ./configure --prefix=/usr/local/src/pcre2-10.42
# make && make install
# cd ..
( pcre 설치 )

# mv apr-1.7.4/ ./httpd-2.4.58/srclib/apr
# mv apr-util-1.6.3/ ./httpd-2.4.58/srclib/apr-util

# cd httpd-2.4.58
# ./configure --prefix=/usr/local/apache2 --with-included-apr --with-pcre=/usr/bin/pcre-config
# make && make install
( apache 설치 )

# vi /usr/lib/systemd/system/httpd.service
---
[Unit]
Description=Apache Service

[Service]
Type=forking
#EnvironmentFile=/usr/local/apache2/bin/envvars
PIDFile=/usr/local/apache2/logs/httpd.pid
ExecStart=/usr/local/apache2/bin/apachectl start
ExecReload=/usr/local/apache2/bin/apachectl graceful
ExecStop=/usr/local/apache2/bin/apachectl stop
KillSignal=SIGCONT
PrivateTmp=true

[Install]
WantedBy=multi-user.target
---
( systmed 에 httpd 추가 )

# systemctl daemon-reload

# vi /usr/local/apache2/httpd.conf
---
User nobody
Group nobody
---

# systemctl start httpd
# systemctl status httpd
# systemctl enable httpd
( 서비스 기동 )
```

### [ 웹 접속 확인 ]
> 인터넷에 아이피 입력하여 정상 접속 확인
( httpd.conf 파일의 DocumentRoot 경로 확인하여 index.html 생성 확인 )


***
**★ 정상적으로 작동된다면 Apache 컴파일 설치 완료 ! ★**
***

## 참조
- https://veneas.tistory.com/entry/Linux-%EC%95%84%ED%8C%8C%EC%B9%98-%EC%84%9C%EB%B2%84-%EC%BB%B4%ED%8C%8C%EC%9D%BC-%EC%84%A4%EC%B9%98-httpd-2452
- https://velog.io/@class1119/Apache-%EC%BB%B4%ED%8C%8C%EC%9D%BC-%EC%84%A4%EC%B9%98#1-gcc-%ED%8C%A8%ED%82%A4%EC%A7%80-%EC%84%A4%EC%B9%98

