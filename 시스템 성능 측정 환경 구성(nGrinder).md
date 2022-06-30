# 시스템 성능 측정 환경 구성(nGrinder)

## 구축 목적
: 실제 서비스에 투입 되기 전 서버의 자세한 성능을 테스트하기 위하여


## nGrinder 구조

![image](./images/%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%B1%EB%8A%A5%20%EC%B8%A1%EC%A0%95%20%ED%99%98%EA%B2%BD%20%EA%B5%AC%EC%84%B1(nGrinder)/1.PNG)


### 주요 구성 요소
* Controller
> 유저, 스크립트, 에이전트를 관리하고, 테스트 분석, 통계를 해주는 모듈

> 성능 테스터가 스크립트를 작성하고 테스트 실행을 구성 할 수 있도록 하는 웹 애플리케이션


* Agent
> 실제 스크립트를 실행시켜 서버에 부하를 주는 모듈

> 컨트롤러의 명령을 받아 실행하고, 스크립트를 수행하는 워커(Worker) 개념으로 타깃(Target)이 되는 기계에 프로세스와 스레드를 실행 시켜 부하를 발생시키는 역할

+ 직접 Groovy, Jython 언어를 사용해 테스트 스크립트를 짤 수 있다.
+ Groovy는 Java와 거의 완벽 호환된다
***

## nGrinder 설치 (CentOS 7)

< 방화벽 작업 >

에이전트
* 모두 => 컨트롤러 : 16001
* 모두 => 컨트롤러 : 12000~12000+ (동시 테스트 허용 횟수)

컨트롤러

* 모두 => 모니터 : 13243
* 컨트롤러 => 공용 사용자 : 톰캣 구성에 따라 다르지만, 기본적으로 8080으로 설정된다.

***
```
** 모든 서버에 Oracle JDK 1.6 이상 또는 OpenJDK 1.7 이상이 필요하다.

< Open JDK 설치 - 모든 서버 >
# yum -y install java-1.8.0-openjdk
# yum -y install java-1.8.0-openjdk-devel

# readlink -f /usr/bin/java
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.332.b09-1.el7_9.x86_64/jre/bin/java
( 환경변수 등록을 위한 경로 확인 )

# vi /etc/profile
--------------------------------------------------------------------------------
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.332.b09-1.el7_9.x86_64/jre/bin/java
PATH=$PATH:$JAVA_HOME/bin
CLASSPATH=$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar

export JAVA_HOME PATH CLASSPATH
--------------------------------------------------------------------------------
( 맨 아랫줄에 추가 )

[ OpenJDK 설치 테스트 ]

1.
# echo $JAVA_HOME
# echo $PATH
# echo $CLASSPATH

2. 
# vi HelloWorld.java
-----------------------------------------------
public class HelloWorld{
   public static void main(String[] args){
        System.out.println("Hello World!!");
   }
}
-----------------------------------------------

# javac HelloWorld.java
# java -cp . HelloWorld
Hello World!!
```

***

```
< nGrinder 컨트롤러 설치 >
* https://github.com/naver/ngrinder/releases/
  -> 버전 확인 후 최신버전 다운로드


# wget https://github.com/naver/ngrinder/releases/download/ngrinder-3.5.5-p1-20210531/ngrinder-controller-3.5.5-p1.war

# nohup java -jar ngrinder-controller.war --port=8300 >> log_ngrinder.nohup&
( 백그라운드로 nGrinder 컨트롤러 실행 ! )

```
* 컨트롤러 서버 IP:8300 입력하여 nGrinder 브라우저 접속
  
  -> 초기 계정 - ID : admin , PW : admin ( 한국어 접속 가능 )

![image](./images/%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%B1%EB%8A%A5%20%EC%B8%A1%EC%A0%95%20%ED%99%98%EA%B2%BD%20%EA%B5%AC%EC%84%B1(nGrinder)/2.PNG)

< 초기 설정 >
1. 사용자 정보 - 비밀번호 변경 에서 초기 패스워드 변경
2. 사용자 관리 - admin을 제외한 나머지 계정 삭제

![image](./images/%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%B1%EB%8A%A5%20%EC%B8%A1%EC%A0%95%20%ED%99%98%EA%B2%BD%20%EA%B5%AC%EC%84%B1(nGrinder)/3.PNG)


***

```
< nGrinder 에이전트 설치 >

* 에이전트 설치파일 다운로드

< 방법 >
1. nGrinder 컨트롤러 웹에 들어가서 메뉴에 에이전트 다운로드 후 FTP로 서버에 업로드
2. wget으로 해당 에이전트 다운로드 링크주소 복사하여 서버에 다운로드
   ex ) wget http://[controller_ip]/agent/download/ngrinder-agent-3.5.5-[controller_ip].tar


# tar -xvf ngrinder-agent-3.5.5-[controller_ip].tar

# cd ngrinder-agent
# ./run_agent_bg.sh
( 에이전트 프로그램 실행 - 백그라운드 용 프로그램으로 실행 함 )

* 실행 후 컨트롤러 웹에 에이전트 관리로 들어가서 에이전트가 정상적으로 올라왔는지 확인 ( 승인됨 확인 )
```
![image](./images/%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%B1%EB%8A%A5%20%EC%B8%A1%EC%A0%95%20%ED%99%98%EA%B2%BD%20%EA%B5%AC%EC%84%B1(nGrinder)/4.PNG)

***

< TEST >
* nGrinder는 JYM에서 실행되는 Python 인 Jython 언어 사용
* 여기서는 기본으로 스크립트를 만들어서 사용하는 방식만 설명 !

 
1. 테스트 Groovy 스크립트 생성
   - 스크립트 - 만들기 클릭하여 스크립트 생성

![image](./images/%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%B1%EB%8A%A5%20%EC%B8%A1%EC%A0%95%20%ED%99%98%EA%B2%BD%20%EA%B5%AC%EC%84%B1(nGrinder)/5.PNG)
- google.com 의 GET 요청 테스트


2. 성능 테스트
   - 테스트 생성
  ![image](./images/%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%B1%EB%8A%A5%20%EC%B8%A1%EC%A0%95%20%ED%99%98%EA%B2%BD%20%EA%B5%AC%EC%84%B1(nGrinder)/6.PNG)
  - 에이전트 수 , 사용자 수 , 스크립트 , 테스트기간 등 필요한 설정을 추가하여 생성

3. 테스트 결과 확인
   - 테스트 완료 후 상세 보고서 클릭
    ![image](./images/%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%B1%EB%8A%A5%20%EC%B8%A1%EC%A0%95%20%ED%99%98%EA%B2%BD%20%EA%B5%AC%EC%84%B1(nGrinder)/7.PNG)

***
**★ 정상적으로 작동된다면 nGrinder 설치 완료 ! ★**
***

### 참조
- https://seankim.life/2021/11/18/ngrinder%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%84%B1%EB%8A%A5-%EC%B8%A1%EC%A0%95/
- https://gogoonbuntu.tistory.com/83
- OpenJDK 설치 : https://bamdule.tistory.com/57