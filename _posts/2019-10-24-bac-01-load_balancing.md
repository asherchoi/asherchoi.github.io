---
title: "[Backend] Load Balancing"
excerpt:
last_modified_at: 2019-10-24

categories:
  - Backend

tags:
  - load balancing  
  - high performance system
  - distributed
  - web service infra

---

![nginx-version-1.14.0](https://img.shields.io/badge/nginx-v1.14.0-blue.svg)
![nginx-version-1.14.0](https://img.shields.io/badge/nginx-v1.14.0-blue.svg)

# Load Balancing with NGINX 

## What is Load Balancing
로드 밸런싱이란 하나의 인터넷 서비스가 발생하는 트래픽이 많을 때 여러 대의 서버가 분산처리하여 서버의 로드율 증가, 부하량, 속도저하 등을 고려하여 적절히 분산 처리하여 해결해주는 서비스이다. 네트워크 상단에 로드밸런서가 존재하여 서버로 들어오는 패킷을 실제 서버로 균일하게 트래픽을 부하 분산시킨다. 만약, 실제 서빙 중 정상적으로 작동하지 않는 경우, 이를 감지해 정상적으로 작동하는 서버로 부하 분산시키기도 한다.

## Why Load Balancing?
로드 밸런싱을 사용하면 고가의 서버로 확장하지 않고 저렴한 비용으로 다수의 서버를 증설해 비용을 절감할 수 있고, 어떤 부분에서의 서버에 장애가 발생하여도 서비스 중단 없이 다른 서버로 적잘히 자동으로 분배하여 서비스가 운용가능하게 할 수 있다. 추후 사용량이 많아 서버 확장에 용이할 뿐만 아니라, 서비스 중단없이 서버 증설이 가능해 지는 효과를 얻을 수 있다.

## Scheduling Algorithm
실제 서버에 처리를 분산할 때, 모든 서버로 균등하게 분산하면 스펙이 다른 서버가 존재하는 환경에서는 그 서버에 트래픽이 치우칠 수 있다. 그래서 상황에 맞는 스케줄링 알고리즘을 선택하여 사용해야 한다. nginx기준으로 디폴트는 Round-Robin 방식이다.
 - RR(Round Robin) : Real-server를 처음부터 차례로 선택해가기
# How to ?

# NginX health checks

nginx에서 리버스 프록시 구현은 in-band 또는 passive 서버 헬스 체크를 지원한다. 만약 특정 서버에서 응답이 실패하면 nginx는 해당 서버는 다운되었다고 판단하고 다음에 오는 요청에 일정시간 해당 서버를 선택하지 않는다.

max_fails 지시어는 일련의 실패수에 도달하면 fail_timeout 동안 해당 서버를 선택하지 않는다. 기본적으로 max_fails는 1로 세팅되어 있다. 만약 0으로 설정한다면 헬스체크를 하지 않는다. fail_timeout 파라미터는 얼마동안 서버를 실패로 표시할 것인가를 의미한다. fail_timeout 인터벌이 지난 후에 nginx는 클라이언트를 요청을 해당 서버가 동작하는지 검증하기 위해 전송할 것이다. 만약 성공하면 서버는 활성화된 상태로 표시될 것이다.

# NginX proxy configuration (Reverse Proxy)
외부에서 내부 서버가 제공하는 서비스 접근 시, Proxy 서버를 먼저 거쳐서 내부 서버로 들어오는 프록시를 Reverse Proxy라고 한다. 외부 사용자는 실제 내부망에 있는 서버의 존재를 모른다. 모든 접속은 Reverse Proxy 서버에게 들어오며, Reverse Proxy는 요청에 맵핑되는 내부 서버의 정보를 알고 요청을 넘겨준다. 따라서 내부 서버의 정보를 외부로부터 숨길 수 있다.

잠깐:) Nginx의 디렉터리 의미를 간단하게 알아봅시다.
/etc/nginx: 해당 디렉터리는 Nginx를 설정하는 디렉터리입니다.모든 설정을 이 디렉터리 안에서 합니다.

/etc/nginx/nginx.conf: Ngnix의 메인 설정 파일로 Nginx의 글로벌 설정을 수정 할 수 있습니다.

/etc/nginx/sites-available: 해당 디렉터리에서 프록시 설정 및 어떻게 요청을 처리해야 할지에 대해 설정 할 수 있습니다.

/etc/nginx/sites-enabled: 해당 디렉터리는 sites-available 디렉터리에서 연결된 파일들이 존재하는 곳 입니다.이 곳에 디렉터리와 연결이 되어 있어야 nginx가 프록시 설정을 적용합니다.

/etc/nginx/snippets: sites-available 디렉터리에 있는 파일들에 공통적으로 포함될 수 있는 설정들을 정의할 수 있는 디렉터리 입니다.
	
sudo vim /etc/nginx/sites-available/default




# Reference
 - [https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)
 - [https://hojak99.tistory.com/448](https://hojak99.tistory.com/448)
 - [https://opennote46.tistory.com/215](https://opennote46.tistory.com/215)
 - [https://medium.com/sjk5766/nginx-reverse-proxy-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-e11e18fcf843](https://medium.com/sjk5766/nginx-reverse-proxy-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-e11e18fcf843)




sudo find / -name nginx.conf
nfinx config 파일을 찾음

cat /etc/nginx/nginx.conf
nginx config파일을 수정

nginx -t
config의 에러유무를 파악


위 명령어는 위에서부터 웹서버의 시작, 중지, 재시작을 나타냅니다. 서버를 시작하였다면 이제 서버가 잘 구동되는지 브라우저에서 8080포트로 접속하여 테스트를 수행합니다.

트리 이미지

sudo vi nginx.conf

http 블록
http 블록은 이후에 소개할 server, location의 루트 블록이라고 할 수 있고, 여기서 설정된 값을 하위 블록들은 상속한다. http 블록은 여러개를 사용할 수 있지만 관리상의 이슈로 한번만 사용하는 것을 권장한다. 

http, server, location 블록은 계층구조를 가지고 있다. 많은 지시어가 각각의 블록에서 동시에 사용할 수 있는데, http의 내용은 server의 기본값이 되고, server의 지시어는 location의 기본값이 된다. 그리고 하위의 블록에서 선언된 지시어는 상위의 선언을 무시하고 적용된다. 

server 블록
server 블록은 하나의 웹사이트를 선언하는데 사용된다. 가상 호스팅(Virtual Host)의 개념이다. 예를들어 하나의 서버로 http://opentutorials.org 과 http://egoing.net 을 동시에 운영하고 싶은 경우 사용할 수 있는 방법이다. 가상 호스팅에 대한 자세한 내용은 가상 호스팅 수업을 참고하자. 

location 블록
location 블록은 server 블록 안에 등장하면서 특정 URL을 처리하는 방법을 정의한다. 이를테면 http://opentutorials.org/course/1 과 http://opentutorials.org/module/1 로 접근하는 요청을 다르게 처리하고 싶을 때 사용한다. 

events 블록
이벤트 블록은 주로 네트워크의 동작방법과 관련된 설정값을 가진다. 이벤트 블록의 지시어들은 이벤트 블록에서만 사용할 수 있고, http, server, location와는 상속관계를 갖지 않는다. 이벤트 모듈 지시어에 대한 설명은 이벤트 모듈 지시어 사전을 참고한다.

설정 파일의 반영
설정 파일의 내용을 변경한 후에는 이를 NGINX에 반영해야 하는데 아래와 같이 reload 명령을 이용한다. restart를 이용해도 되지만 권장되지 않는다. 

1
sudo service nginx reload;
 sudo service nginx start
sudo service nginx stop
sudo service nginx restart


sudo netstat -ntlp
'tcp정보' 중에서 'listening' 상태인 소켓의 정보를 '10진수'로 변환하여 보여주며 실행되고 있는 '프로그램과 PID정보'를 출력한다.

curl -X POST localhost:5001/user
curl -X POST localhost:5002/user