# Zabbix Server LINE 연동

## 구축 목적
: Zabbix Server로 부터 모니터링 중 문제되는 현상이 발생했을때 LINE 으로 메세지를 트리거 해주기 위해

## LINE 연동 구축
```
# vi /usr/lib/zabbix/alertscripts/zbxln.sh
( alertscripts 밑에 새로운 쉘 스크립트 생성 )
-----------------------------------------------------
#!/bin/bash
# LINE Notify Token - Media > "Send to".
TOKEN="$1"
# {ALERT.SUBJECT}
subject="$2"
# {ALERT.MESSAGE}
message="$3"
# Line Notify notice message.
notice="
${subject}
${message}
"
curl -X POST -H "Authorization: Bearer ${TOKEN}" -F "message=${notice}" 
https://notify-api.line.me/api/notify
-----------------------------------------------------

# chmod +x zbxln.sh
( 실행 권한 추가 )

# vi /etc/zabbix/zabbix_server.conf
-----------------------------------------------------
523 AlertScriptsPath=/usr/lib/zabbix/alertscripts
( 파일 경로와 일치하는지 확인 )
-----------------------------------------------------


+
-----------------------------------------------------
그룹이 없다면 -> 그룹 생성 및 토큰 발급
-----------------------------------------------------
1. 라인 접속 및 그룹 만들기

2. Line Notify를 멤버로 추가 ( 메세지 송신자 )

3. https://notify-bot.line.me/en/ 들어가서 로그인 후 마이페이지 -> 하단 Generate Token -> 토큰 네임 기입 + 앞서 생성한 그룹 클릭

4. 토큰 생성 완료 ( 따로 복사해서 저장 )
-----------------------------------------------------
```

### Zabbix 홈페이지 설정

1.
- xml 파일을 통해 작성
관리 -> 미디어타입

가져오기로 zbx_Line_mediatypes_5.0.xml ( 미디어 타입 설정 파일 ) 가져오기
> [Zabbix Line 연동 파일!][link]

[link]: https://github.com/Dawon2/Server-Practice/tree/main/Zabbix%20%EC%84%9C%EB%B2%84/Zabbix%20LINE%20%EC%97%B0%EB%8F%99%20%ED%8C%8C%EC%9D%BC


- 문제 발생시 수동 작성

	- 연락 방법
```
이름 : Line Notify_shell
종류 : 스크립트
스크립트 이름 : zbxln.sh ( 전에 만든 스크립트 파일명과 동일 )
스크립트 파라미터 : (ALERT_SENDTO)
		 (ALERT_SUBJECT)
		 (ALERT_MESSAGE)
```

	- Message templates
장애 , Problem recovery , Problem update , 디스커버리 , Autoregistration 추가

- 2. 액션 작성
설정 -> 액션 -> 왼쪽 위 Trigger actions - 액션작성

	- 액션
이름 : Line Action
( 조건 X , 모든 액션에 대해서 )

	- 오퍼레이션
오퍼레이션 : 사용자에게 메세지를 송신 : Admin
( 액션 발동 시 어디로 어떤 행동을 취할건지 선택 )

복구시, 갱신시 = 필요시에 추가

- 3. 토큰 연결
관리 -> 유저

	- Admin 클릭 - 추가

	- 연락방법
종류 : Line Notify_shell
수신처 : Line 토큰 입력


## TEST

1.
관리 -> 미디어타입 

Line Notify_shell 테스트 클릭해서 생성 한 그룹으로 메세지 잘 오는지 확인

2.
Zabbix Server에서
```
# cd /usr/lib/zabbix/alertscripts/

# ./zbxln.sh DQN5cYVVNpm4DqbgLKabtWhz6wvuYQav93LM3wHNY3p $SUBJECT $MESSAGE
  ( ./zbxln.sh  라인 토큰값, 제목, 메세지 )
```
-> 그룹으로 메세지 잘 오는지 확인

***
**★ 정상적으로 작동된다면 LINE 연동 완료 ! ★**



