remote 223.130.162.247 1194
# Client가 접속할 서버의 IP와 포트번호 지정

client
# 현재 설정파일이 Client용 임을 지정

remote-cert-tls server
# 접속할 OpenVPN서버가 tls통신을 사용함을 지정

dev tun0
# OpenVPN이 사용하는 네트워크 디바이스 명을 지정

proto tcp
# OpenVPN이 사용하는 기반 프로토콜을 지정

cipher AES-256-GCM
auth SHA512
# 패킷 암호화 방식 지정 및 패킷 인증 알고리즘

resolv-retry infinite
# 호스트를 찾지 못했을 때 재 시도 시간 옵션

nobind
# 소켓 바인드 기능(서버 역할)을 하지 않는다고 지정

persist-key
# 재연결 시에도 동일한 키를 그대로 사용한다고 지정

persist-tun
# 재연결 시에도 동일한 디바이스를 그대로 사용한다고 지정

float
# 클라이언트가 고정된 IP를 사용하지 않는다고 지정

 
ca "C:\\Program Files\\OpenVPN\\keys\\ca.crt"
# CA 인증서 파일명 지정

cert "C:\\Program Files\\OpenVPN\\keys\\dw.crt"
# 클라이언트 인증서 파일명 지정

key "C:\\Program Files\\OpenVPN\\keys\\dw.key"
# 클라이언트 키 파일명 지정
