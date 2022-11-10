# MariaDB 바이너리 설치 (Ubuntu)


## < 설치 전 참조 >
1. 어플리케이션 설치 디렉토리와 데이터 디렉토리를 구분하여 진행
* /app , /data 디렉토리 생성 후 진행 할 예정
  
2. MariaDB 공식 홈페이지에서 tar 파일 다운로드 후 FTP로 서버에 업로드 하고 진행
* 주소 : https://mariadb.org/download/?t=mariadb&p=mariadb&r=10.11.0&os=windows&cpu=x86_64&pkg=msi&m=blendbyte

3. DB 전용 계정 및 그룹 생성

  
## < MariaDB 설치 >
```

# adduser maria

# mkdir /app
# mkdir /data

# mv mariadb-10.1.48-linux-systemd-x86_64.tar.gz /app
# tar xvfz mariadb-10.1.48-linux-systemd-x86_64.tar.gz
# mv mariadb-10.1.48-linux-systemd-x86_64.tar.gz mariadb

# ln -s /app/mariadb /usr/local/mysql

# chown -R maria.maria /app
# chown -R maria.maria /usr/local/mysql

# vi /etc/profile
---------------------------------------
export MARIADB_HOME=/usr/local/mysql
export PATH=$PATH:$MARIADB_HOME/bin:.
---------------------------------------
( 해당 내용 아래줄에 추가 )

# source /etc/profile

# vi /etc/my.cnf
( 없으면 생성 )
------------------------------------------------------------------------------
[client-server]
port=3306

# This will be passed to all MariaDB clients

# The MariaDB server
[mysqld]
# Directory where you want to put your data
basedir=/usr/local/mysql
datadir=/data
tmpdir=/tmp

# This is the prefix name to be used for all log, error and replication files
log-basename=mariadb

innodb_file_format=Barracuda
innodb_large_prefix=1
innodb_default_row_format=dynamic
------------------------------------------------------------------------------
( 설치 정보 입력 )

# $MARIADB_HOME/scripts/mysql_install_db --user=maria --defaults-file=/etc/my.cnf
( user 정보 및 설정파일 경로 추가하여 DB 설치 )

---
< 설치 중 오류해결 정보 >
1. WARNING: The host 'ettdb' could not be looked up with /usr/local/mysql/bin/resolveip.
-> /etc/hosts 파일에 호스트 정보 추가 ex) db01 172.25.1.1

2. /usr/local/mysql/bin/mysqld: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory
-> libaio1 설치 - apt -y install libaio1
---

# ll /data
( 설정한 data 디렉토리에 데이터가 생겼는지 확인 )

# cp -rfp $MARIADB_HOME/support-files/mysql.server /etc/init.d/mariadb
( 권한을 포함하여 하위 디렉토리 내용까지, 서비스 실행 프로그램을 init.d 밑에 복사 )

# update-rc.d mariadb defaults
( 부팅 시 시작되도록 서비스 추가 )
# update-rc.d mariadb start 20 2 3 4 5
( mariadb 서비스를 20 우선순위로 2,3,4,5 런레벨에서 실행 )

# vi /app/mariadb/mydqld_safe
# vi /etc/init.d/mariadb
( 해당 두개의 conf 파일에서 user를 maria로 변경 )
------------------
user='maria'
------------------

# systemctl enable mariadb
# systemctl start mariadb
* or service mariadb start

-> 정상적으로 mariadb 서비스가 올라왔는지 확인!
---
1. ps -ef | grep mariadb
-> 프로세스 확인
2. netstat -plunt
-> 3306 포트로 db 떠있는지 확인
3. mysql
-> DB 접속 정상적으로 되는지 확인
---

# visudo
maria  ALL=(ALL) NOPASSWD: /bin/systemctl start mariadb
maria  ALL=(ALL) NOPASSWD: /bin/systemctl status mariadb
maria  ALL=(ALL) NOPASSWD: /bin/systemctl stop mariadb
maria  ALL=(ALL) NOPASSWD: /bin/systemctl enable mariadb
( DB 전용계정에 해당 권한 내용 추가 )

# su - maria
# systemctl status mariadb
( DB 전용계정에서 확인 )

```

***
**★ 정상적으로 작동된다면 MariaDB 바이너리 설치 완료 ! ★**
***

## <참조>
- https://bangu4.tistory.com/133
- update-rc.d 관련 사이트 : https://hyess.tistory.com/255