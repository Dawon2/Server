# Grafana 알람설정

## 구축 목적
: Grafana로 상황에 따른 트리거를 설정하여 알람 받기

### Port : 25, 587

## 알람설정 구축

**> Grafana 환경 전부 구축 후 진행**

**> Gmail의 경우 계정설정의 보안탭에서 보안수준이 낮은 앱의 액세스 설정을 "액세스 허용" 해주어야 함.**

### 서버 설정
```
# yum -y install postfix
# yum -y install epel-release
# yum -y install ssmtp
( smtp 테스트용 서비스 ssmtp 설치 )

# vi /etc/ssmtp/ssmtp.conf
------------------------------------------------------------------
     9 root=chku153@xenosolution.co.kr
     14 mailhub=smtp.gmail.com:587
     27 Hostname=localhost
     32 AuthUser=chku153@xenosolution.co.kr
     33 AuthPass=<패스워드 입력>
     34 UseTLS=YES
     35 UseSTARTTLS=YES
------------------------------------------------------------------
( 메일을 받을 계정과 패스워드 , smtp 서버 등 입력 )

# echo "test" | ssmtp chku153@xenosolution.co.kr
( 테스트 메일 보내기 )
* 오류 발생시 설정파일과 방화벽 확인


# vi /etc/grafana/grafana.ini
------------------------------------------------------------------
    599 [smtp]
    600 enabled = true
    601 host = smtp.gmail.com:587
    602 user = chku153@xenosolution.co.kr
    603 # If the password contains # or ; you have to wrap it with triple quotes. Ex """#password;"""
    604 password = <패스워드 입력>
    605 ;cert_file =
    606 ;key_file =
    607 skip_verify = true
    608 from_address = chku153@xenosolution.co.kr
    609 from_name = Grafana
------------------------------------------------------------------
```

**> Grafana 홈페이지 접속**

### Alert channel 생성
- Alerting -> Notification channels -> New Channel
     - Name : Gmail Test
     - Type : Email
     - Address : chku153@xenosolution.co.kr
     - Optional Email settings : single email 체크
     - Notification settings : Default , Include image 체크

**- Test 후 메일 잘 오는지 확인 후 Save**

### Alert 생성 및 연결
1. 대시보드로 들어가서 Panel 클릭 후 Edit ( Variables가 포함된 Panel은 Alert 설정 불가 ! )
2. Alert 클릭
Name : Test alert
Evaluate every : 10s  ,  For : 0  ( 10초동안 계속(0초) 컨디션 체크 )
Conditions
- WHEN : last() ,  OF : query(A, 10s, now)  ,  IS ABOVE : 80  ( A쿼리에 값중 10초전부터 지금까지, 마지막 값이 80이상이면 Alert 발동 )

If no data or all values are null : NO Data ( No data와 모든 Value null에 대해서 알람 OFF )
If execution error or timeout : Alerting ( 실행에러와 타임아웃 알람 ON )

Send to : Gmail Test ( 생성한 Alert Channel )
Message : CPU usage is over 80% !! ( 메일에 포함하고싶은 메세지 입력 )

-> Test rule 클릭 후 정상적으로 메일오는지 확인

***
## ★ 정상적으로 작동된다면 Zabbix - Grafana 연동 완료 ! ★
***

## 참조
- https://ksr930.tistory.com/177
- https://danawalab.github.io/common/2021/01/26/Common-Grafana-Alert.html
