# Zabbix Agent 구축 - Window 수동 설치

### IP
Zabbix Server : 172.17.124.244
Zabbix Agent - Centos : 172.17.124.243
Zabbix Agent - Window : 172.17.124.198

***

## Agent 구축

- 1 먼저 설치파일 다운로드 진행 !!
> https://www.zabbix.com/download_agents?version=5.4&release=5.4.10&os=Windows&os_version=Any&hardware=amd64&encryption=OpenSSL&packaging=MSI&show_legacy=0


2. C:\zabbix 에 폴더 이름변경 후 안에 압축 해제

3. C:\zabbix\conf 경로에서 zabbix_agentd.conf 파일을 워드패드로 실행시켜서 내용 수정
```
-----------------------------------------------------------------------------------------------
LogFile=c:\zabbix\zabbix_agentd.log [디폴트는 C:\ 되어 있으므로 설치된 경로로 변경하는 것을 권고]

Server=172.17.124.244 [ Zabbix Server IP 입력(수정) ]

#ServerActive=127.0.0.1 [ 현재는 사용할 일이 없으므로 주석 처리 ]

Hostname=172.17.124.243 [ Zabbix Agent IP 입력(수정) ]

HostnameItem [ 주석 처리 ]
-----------------------------------------------------------------------------------------------
```

4. cmd 실행

C:\Users\Administrator > cd c:\zabbix\bin
( 경로 이동 )

c:\zabbix\bin > zabbix_agentd.exe -i -c c:\zabbix\conf\zabbix_agentd.conf
( zabbix agent 파일로 설치를 진행하고 config 파일을 넣어준다 )

5. 서비스 설정

윈도우 + R -> services.msc -> Zabbix Agent 서비스 시작

6. 방화벽 설정

윈도우 + R -> firewall.cpl -> 고급 설정 -> 인바운드 규칙 + 새 규칙 ->
포트 선택 -> TCP + 특정 로컬 포트 10050 -> 연결 허용 -> 도메인, 개인, 공용 체크 -> 이름 - Zabbix Agent 포트 허용 + 설명 - 포트 : 10050

7. Zabbix WEB 설정

- Zabbix 서버 홈페이지로 이동

  - 설정 - 호스트그룹 - 호스트 그룹 작성
그룹이름 : Windows Server

  - 설정 - 호스트
호스트명 : Windows Server - Agent
그룹 : Windows Server
Interfaces : 172.17.124.243 ( Agent 서버 주소 )
IP주소 : 포트 10050
템플릿 - Link new templates : Template OS Windows by Zabbix agent
추가

***

## TEST
모두 등록 완료 후 호스트에서 Zabbix Agent 연결 확인
( ZBX 초록불 )

***
**★ 정상적으로 작동된다면 Zabbix Agent 윈도우 구축 완료 ! ★**
***

## 참조
- https://foxydog.tistory.com/18
