# NTP 서버 구축

## 구축 목적
: 여러 시스템들간 서버 시간을 동기화해주기 위하여

**Port 번호 : 123번**

## NTP 서버 구축

### 서버 설정
```
# yum -y install ntp

# vi /etc/ntp.conf
21 server time.google.com iburst
( 시간을 구글 ntp서버에서 주는 시간으로 동기화시키기 )

# systemctl start ntpd
# systemctl enable ntpd
```

## TEST

### 1. 시간 잘 설정되었는지 확인
```
# date
Fri Feb 18 16:49:46 KST 2022
```

### 2. ntp 설정이 정상적으로 작동되는지 확인
```
# ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
*time1.google.co .GOOG.           1 u  429  512  377   55.289   -0.088   0.113

```

***
**★ 정상적으로 작동된다면 ntp 서버 구축 완료 ! ★**
***
