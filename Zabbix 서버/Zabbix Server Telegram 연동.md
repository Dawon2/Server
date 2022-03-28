# Zabbix Server Telegram 연동

## 구축 목적
: Zabbix Server로 부터 모니터링 중 문제되는 현상이 발생했을때 Telegram 으로 메세지를 트리거 해주기 위해

## Telegram 연동

**먼저 Telegram Bot 생성**
```
------------------------------------------------------
1. 검색창에 BotFather 입력해서 시작
/newbot -> Dwzbx -> Dwzbx_bot
( 새로운 봇 생성 , 봇 이름 , 봇 사용자 이름 - 끝에 bot )

2. Dwzbx 채팅방 들어가서 /start 입력
------------------------------------------------------
```
```
# yum -y install git
# git clone https://github.com/ableev/Zabbix-in-Telegram.git

# cd Zabbix-in-Telegram/
# cp zbxtg_settings.example.py zbxtg_settings.py

# vi zabbix_settings.py
-----------------------------------------
3 tg_key = "5276296542:AAGvHERGRs2VOo-WMUNYIxfH6C20KL9ndwc"
  ( telegram api 키 입력 )

6 zbx_tg_tmp_dir = "/zbxtg/" + zbx_tg_prefix

16 zbx_api_user = "Admin"
17 zbx_api_pass = "zabbix"

25 zbx_basic_auth_user = "Dwzbx_bot"
( 생성한 봇 사용자 명 입력 )
26 zbx_basic_auth_pass = "zabbix"

47 zbx_db_host = "localhost"
48 zbx_db_database = "zabbix"
49 zbx_db_user = "zabbix"
50 zbx_db_password = "zabbix"
-----------------------------------------

# mv Zabbix-in-Telegram/* /usr/lib/zabbix/alertscripts/


# yum -y install epel-release python-pip

# vi /etc/yum/repos.d/epel.repo
-----------------------------------------
3 Baseurl 주석  풀기
4 metalink 주석
-----------------------------------------

# pip install requests
```

### Zabbix 웹 설정

1.
관리 -> 유저 -> 텔레그램 선택

- 연락방법
```
종류 : 스크립트
스크립트 이름 : zbxtg.py
스크립트 파라미터 : (ALTER SENDTO)
		 (ALTER SUBJECT)
		 (ALTER MESSAGE)
```

2.
```
관리 -> 유저

admin 클릭 ( 앞서 선택한 계정 )
```
- 연락방법
```
추가
종류 : Telegram
수신처 : api 키
추가
```

## TEST

1.
```
# cd /usr/lib/zabbix/alertscripts

# python zbxtg.py [“Telegram ID”] [“Title”] [“Context”]
ex ) python zbxtg.py @dwdwdw456 hi hihihihi
( Telegram ID는 Telegram에서 setting -> Edit Profile 에서 확인 , 없으면 생성 )

텔레그램 그룹방에 메세지 잘 오는지 확인
```

2.
메세지 보냈을때 uid 잘 올라오는지 확인
```
# cd /var/tmp/zbxtg
# cat uids.txt
( Telegram 채팅방에 정상적으로 올라오면 같이 올라옴 )
```
3.
```
관리 -> 미디어타입

telegram 테스트
수신처 api 키 입력해서 테스트
```

***
**★ 정상적으로 작동된다면 Telegram 연동 완료 ! ★**
***
