# Zabbix 호스트 자동 등록

## 구축 목적
: Zabbix Agent가 새로 생길때마다 일일히 등록하기 힘들기 때문에 자동 호스트를 등록하기 위하여

## 자동 등록 구축

[ Zabbix 웹 ]

설정 -> 액션 -> 왼 상단 Autoregistration actions

- 액션 작성
이름 : Linux auto ADD
조건 : 호스트 메타데이터 , 포함 , Linux

- 오퍼레이션
종류 : 호스트 추가, 호스트그룹 추가, 템플릿과 링크 작성, 호스트 활성화 등
```
------------------------------
호스트 추가
호스트그룹을 추가: Linux servers
템플릿과 링크: Template OS Linux by Zabbix agent active
호스트를 활성화
------------------------------
```
* 액션에서 이러이러한 조건에 해당되면 오퍼레이션 설정이 발동되는 구조 *
ex ) 호스트의 메타데이터에 Linux가 포함되면 호스트 , 그룹 , 템플릿 , 활성화 등이 자동으로 실행


[ Zabbix Agent ]

* 앞서 내용과 똑같이 설치, 있다면 conf 내용 수정 후 리스타트


```
# rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
# yum clean all
# yum –y install zabbix-agent

# vi /etc/zabbix/zabbix-agentd.conf

Ex)
Server = 172.17.124.244 
ServerActive = 172.17.124.244
Hostname = Zabbix agent
HostMetadata = Linux_agent_1

* Hostname 주석처리하고 HostnameItem 도 사용가능 ( #HostnameItem=system.hostname 서버의 hostname을 따와서 등록 )
```

## TEST
1.
정상적으로 다 설정한 후 Zabbix 웹에서 호스트 삭제 후 
```
# systemctl restart zabbix-agent

정상적으로 호스트 자동 등록되는지 확인
```

***
**★ 정상적으로 작동된다면 Agent 자동 등록 설정 완료 ! ★**
***
