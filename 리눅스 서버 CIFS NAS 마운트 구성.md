# 리눅스 서버 CIFS NAS 마운트 구성
* CentOS 7.9 기준
  
* 서버 IP 예시

>  NAS 서버 : 10.1.1.1

>  NAS를 마운트 할 서버 : 10.123.123.123

***

## 구성 목적
: 윈도우서버와 리눅스서버 간 동일한 NAS 사용이 필요할때 CIFS 프로토콜의 NAS를 사용해야 함

***

### [ 마운트 작업 전 방화벽 작업 진행 ]
* NAS 적용 서버 -> NAS
> 출발지 : 10.123.123.123 ( NAS 적용 서버 )

> 목적지 : 10.1.1.1 ( NAS )

> Port : TCP / 445 (SMB)

***

### [ CIFS NAS 마운트 구성 ]
```
< 1. cifs-utils 패키지 설치 >

- yum 설치 가능 -
# yum -y install cifs-utils

- yum 설치 불가능 -
1. 동일한 구성의 타 서버에서 rpm 파일 다운로드
( 해당 rpm파일이 없는 서버여야 하며, --resolve 설정으로 의존성파일까지 받아야 함 )
# yumdownloader --downloadonly --resolve cifs-utils
# yumdownloader --downloadonly --resolve samba-common-libs-4.10.16-24.el7_9.x86_64.rpm
# yumdownloader --downloadonly --resolve samba-client-libs-4.10.16-24.el7_9.x86_64.rpm
# yumdownloader --downloadonly --resolve libwbclient-4.10.16-24.el7_9.x86_64.rpm

- 위와 같이 rpm 파일을 다운로드하면 26개의 rpm 파일이 다운로드 됨
---
avahi-libs-0.6.31-20.el7.x86_64.rpm
cifs-utils-6.2-10.el7.x86_64.rpm
cups-libs-1.6.3-51.el7.x86_64.rpm
gnutls-3.3.29-9.el7_6.x86_64.rpm
krb5-devel-1.15.1-55.el7_9.x86_64.rpm
krb5-libs-1.15.1-55.el7_9.x86_64.rpm
libkadm5-1.15.1-55.el7_9.x86_64.rpm
libldb-1.5.4-2.el7.x86_64.rpm
libtalloc-2.1.16-1.el7.x86_64.rpm
libtdb-1.3.18-1.el7.x86_64.rpm
libtevent-0.9.39-1.el7.x86_64.rpm
libwbclient-4.10.16-24.el7_9.x86_64.rpm
nettle-2.7.1-9.el7_9.x86_64.rpm
nss-3.79.0-5.el7_9.x86_64.rpm
nss-pem-1.0.3-7.el7_9.1.x86_64.rpm
nss-sysinit-3.79.0-5.el7_9.x86_64.rpm
nss-tools-3.79.0-5.el7_9.x86_64.rpm
openssl-devel-1.0.2k-26.el7_9.x86_64.rpm
openssl-libs-1.0.2k-26.el7_9.x86_64.rpm
openssl-static-1.0.2k-26.el7_9.x86_64.rpm
samba-client-libs-4.10.16-24.el7_9.x86_64.rpm
samba-common-4.10.16-24.el7_9.noarch.rpm
samba-common-libs-4.10.16-24.el7_9.x86_64.rpm
trousers-0.3.14-2.el7.x86_64.rpm
zlib-1.2.7-21.el7_9.x86_64.rpm
zlib-devel-1.2.7-21.el7_9.x86_64.rpm
---

2. NAS 설치 서버로 rpm 파일 업로드
# rpm -Uvh (패키지명)
( 해당 명령어로 전체 패키지 순차적으로 설치 )
* 의존성이 서로 물려있는 경우 두개의 패키지를 동시에 설치하면 됨
```
***
```
< 2. CIFS 설정 및 마운트 >
1. cifs 설정파일 작성
# /root/nas/account.conf
---
user=cifstest
password=test123!@#
domain=10.123.123.123
---

2. /etc/fstab 설정 적용
# vi /etc/fstab
---
//10.123.123.123/share_~~~  /mnt/nas    cifs    credentials=/root/nas/account.conf
---

3. mount 및 적용 확인
# mount /mnt/nas
# df -Th
```

***
**★ 정상적으로 적용 되었다면 리눅스 CIFS NAS 마운트 완료 ★**
***
