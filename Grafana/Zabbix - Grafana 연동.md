# Zabbix - Grafana 연동

## 구축 목적
: 기존 Zabbix 보다 더 가시성 높은 환경으로 모니터링 하기 위하여


### Port 번호 : 3000
### 서버 IP : 172.17.120.241


## Grafana 연동

**먼저 Zabbix Server 구축 후 진행( Zabbix Server 구축 아래 링크 참고 )**

링크 : [Zabbix 서버 구축!][link]

[link]: https://github.com/Dawon2/Server-Practice/blob/main/Zabbix%20%EC%84%9C%EB%B2%84/Zabbix%20Server%20%EA%B5%AC%EC%B6%95.md

### 서버 설정
```
# yum -y install wget
# wget https://dl.grafana.com/enterprise/release/grafana-enterprise-8.4.1-1.x86_64.rpm
( Grafana 패키지 설치 - 버전은 홈페이지 참고해서 설치 )

# yum -y install grafana-enterprise-8.4.1-1.x86_64.rpm

# systemctl start grafana-server
# systemctl enable grafana-server

# grafana-cli plugins install alexanderzobnin-zabbix-app
( Grafana에 Zabbix 플러그인 패키지가 없기 때문에 수동 설치 )
# systemctl restart grafana-server

```

**-> 아래부터는 Grafana 홈페이지 접속 후 진행**

```http://172.17.120.241:3000/```
( 서버 IP에 Grafana 포트 3000번 입력 )

- 초기 ID / PW
: admin / admin

1. 자빅스 연동
- 플러그인
  - Configuration(설정) - Plugins - Zabbix 검색 ( 아까 설치한 플러그인 패키지 ) - 클릭 후 Enable

- 데이터소스 추가
  - Configuration(설정) - Data Sources - Add data source - zabbix
    - HTTP
      URL : ```http://172.17.120.241/zabbix/api_jsonrpc.php```

    - Zabbix API details
      Username : Admin
      Password : zabbix
      ( Zabbix 홈페이지 접속 초기 ID / PW )
    - Save & test

2. Grafana 대시보드 추가
https://grafana.com/grafana/dashboards/?search=zabbix
( 그라파나 대시보드 사이트에서 원하는 대시보드 url 복사 ! )

- 대시보드 생성
  - Create(생성) - Import - 복사한 대시보드 url 입력 - Load - Data source Zabbix 선택 - Import

  -> 대시보드 추가 완료!!

### 추가
- 대시보드에 전부 No data가 뜨면 mariadb 확인 및 재시작 , 그라파나 설정 확인
  /var/log/grafana/grafana.log 에서 log 확인

- 대시보드에 특정 부분만 No data가 뜨면
  -> 대시보드 특정 해당 부분 제목 클릭 - Explore - Item 부분이 Zabbix 최근데이터의 이름과 동일한지 확인 - 수정하면 해결

- 추가적으로 Agent 연동은 동일하게 진행하고 그라파나 재시작!

***
**★정상적으로 작동된다면 Zabbix - Grafana 연동 완료 ! ★**
***

## 참조
- https://honglab.tistory.com/72
- https://gsk121.tistory.com/427
- https://cloudest.tistory.com/25
- https://cloudest.oopy.io/posting/004

- 그라파나 공식 데모페이지 : https://play.grafana.org/
