# DNS 서버 구축

## 구축 목적
: 클라이언트가 도메인으로 접속했을때 그에 맞는 IP를 찾아서 전달해주기 위해

### Port 번호 : 53

***

## DNS 서버 구축
```
# yum -y install bind-*
# rpm -qa | grep bind*

# vi /etc/named.conf
----------------------------------------------
13         listen-on port 53 { any; };      # 53번 Port로 접근할수 있는 IP 설정
14         listen-on-v6 port 53 { ::1; };   #  〃 IPv6 설정
15         directory       "/var/named";    # 실제로 서비스 할 DNS의 zone파일 디렉토리 설정
21         allow-query     { any; };        # 쿼리 질의를 받는것을 허용하는 IP 대역 설정
33         recursion no;                    # 반복 질의 요청응답에 대한 설정, 보안설정 ( 사설 DNS로 사용하는 경우가 아니면 yes )
----------------------------------------------

# vi /etc/named.rfc1912.zones
( DNS zone 파일을 등록하는 설정 파일 )
----------------------------------------------
zone "study.com" IN {                       # 정방향 설정
        type master;
        file "study.com.zone";
        allow-update { none; };
};

zone "1.16.172.in-addr.arpa" IN {           # 역방향 설정
        type master;
        file "study.com.rev";
        allow-update { none; };
};
----------------------------------------------
( 맨 마지막줄에 해당 내용 추가 -> 도메인명과 zone,rev 파일 확인 )

# cd /var/named/
( zone 파일이 있는 디렉토리로 경로 이동 )

# cp named.localhost ./study.com.zone
# cp named.localhost ./study.com.rev
( 기존 파일을 복사하여 zone파일과 rev파일 생성 )

# chown .named study.com.*
( zone파일과 rev파일에 소유자 그룹 변경 -> named )

# vi study.com.zone
-------------------------------------------------------------
$TTL 1D
@       IN SOA  study.com.              root(
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        IN      NS      www.study.com.
        IN      A       172.16.1.8
www     IN      A       172.16.1.8
dw      IN      CNAME   www.study.com.
-------------------------------------------------------------
( 메인 도메인, 네임서버, A 레코드, CNAME 등을 작성해준다. )
Ex ) www와 메인 도메인(study.com)을 합친 www.study.com. 에 매핑된 IP는 172.16.1.8 으로 설정한다. - A 레코드
     * www.study.com -> 172.16.1.8
     
     dw와 메인 도메인(study.com)을 합친 dw.study.com. 에 매핑된 도메인은 www.study.com. 으로 설정한다. - CNAME
     * dw.study.com -> www.study.com -> 172.16.1.8


# vi study.com.rev
-------------------------------------------------------------
$TTL 1D
@       IN SOA  study.com.              root(
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        IN      NS      www.study.com.
8       IN      PTR     study.com.
8       IN      PTR     www.study.com.
-------------------------------------------------------------
( 메인 도메인, 네임서버, PTR 등을 작성해준다. )
EX ) 172.16.1.8 에 매핑된 도메인은 www.study.com. 으로 설정한다. - PTR
     * 172.16.1.8 -> www.study.com
     
# named-checkconf /etc/named.conf
( named.conf 파일에 대한 설정 정상여부 확인 )

# named-checkzone study.com /var/named/study.com.zone
# named-checkzone 1.16.172.in-addr.arpa /var/named/study.com.rev
( 정방향 zone파일과 역방향 rev파일에 대한 정상여부 확인 )

# vi /etc/resolv.conf
nameserver 172.16.1.8
nameserver 169.254.169.53
nameserver 169.254.169.54
( 자신의 DNS 서버를 첫번째로 들어오게 설정 )

# systemctl start named
# systemctl enable named
```

**Config 파일들의 자세한 설정 내용들은 아래 참조 사이트에서 확인**

***

## TEST
- DNS를 통한 도메인 조회
```
# nslookup
--------------------------------------------------
> server
Default server: 172.16.1.8
Address: 172.16.1.8#53
Default server: 169.254.169.53
Address: 169.254.169.53#53
Default server: 169.254.169.54
Address: 169.254.169.54#53
--------------------------------------------------
> www.study.com
Server:         172.16.1.8
Address:        172.16.1.8#53

Name:   www.study.com
Address: 172.16.1.8
--------------------------------------------------
> dw.study.com
Server:         172.16.1.8
Address:        172.16.1.8#53

dw.study.com    canonical name = www.study.com.
Name:   www.study.com
Address: 172.16.1.8
--------------------------------------------------
> 172.16.1.8
8.1.16.172.in-addr.arpa name = study.com.
8.1.16.172.in-addr.arpa name = www.study.com.
--------------------------------------------------

( 위와 같이 서버, 도메인 네임, CNAME, 역방향 등 모두 잘 적용 되었는지 확인 )
```

***
**★ 정상적으로 작동된다면 DNS 서버 구축 완료 ! ★**
***

## 참조
- https://nirsa.tistory.com/104
- https://it-serial.tistory.com/entry/Linux-CentOS-7-DNS-%EC%84%9C%EB%B2%84-%EA%B5%AC%EC%B6%95-%EB%8F%84%EB%A9%94%EC%9D%B8-%EC%84%A4%EC%A0%95
