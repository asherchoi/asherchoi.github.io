---
title: "[Backend] Stress Test"
excerpt:
last_modified_at: 2019-10-25

categories:
  - Backend

tags:
  - load balancing  
  - high performance system
  - distributed
  - web service infra

---

![nginx-version-1.14.0](https://img.shields.io/badge/nginx-v1.14.0-blue.svg)

# Load test

# Load test by using ab
AB(Apache HTTP server Benchmarking tool)는 커맨드 라인을 활용한 매우 가볍고 유용한 웹서버 벤치마킹 도구 이다.
간단한 REST API나 정적 컨텐츠에 대한 성능 테스트 시에 빠르고 간편하게 벤치마킹 정보를 얻어올 수 있다.
이번에 성능개선 작업을 진행하면서 수정된 API에 대해서 개발환경에는 ngrinder를 활용하고 운영환경에서는 서버에 뭔가 설치하고 성능테스틀 돌려보기 힘든 환경이라 AB를 활용해서 테스트를 실시해 보기로 했다.

# Usage

# Benchmark
## Benchmark 1. concurrency 100 request number 50000
``` console 
choi@airi:~/myproject$ ab -k -c 100 -n 50000 localhost:80/
```
``` console
Concurrency Level:      100
Time taken for tests:   8.978 seconds
Complete requests:      50000
Failed requests:        0
Keep-Alive requests:    49547
Total transferred:      8897735 bytes
HTML transferred:       1200000 bytes
Requests per second:    5569.02 [#/sec] (mean)
Time per request:       17.956 [ms] (mean)
Time per request:       0.180 [ms] (mean, across all concurrent requests)
Transfer rate:          967.81 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.1      0       2
Processing:     0   18  17.5      6      41
Waiting:        0   18  17.5      6      41
Total:          0   18  17.5      7      41

Percentage of the requests served within a certain time (ms)
  50%      7
  66%     35
  75%     35
  80%     35
  90%     36
  95%     36
  98%     36
  99%     36
 100%     41 (longest request)
```

## Benchmark 2. concurrency 200 request number 50000
``` console 
choi@airi:~/myproject$ ab -k -c 200 -n 50000 localhost:80/
```
``` console
Concurrency Level:      200
Time taken for tests:   27.372 seconds
Complete requests:      50000
Failed requests:        0
Keep-Alive requests:    49584
Total transferred:      8897920 bytes
HTML transferred:       1200000 bytes
Requests per second:    1826.66 [#/sec] (mean)
Time per request:       109.489 [ms] (mean)
Time per request:       0.547 [ms] (mean, across all concurrent requests)
Transfer rate:          317.45 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.2      0       4
Processing:     0   76 853.7     12   27343
Waiting:        0   76 853.7     12   27343
Total:          0   76 853.7     14   27343

Percentage of the requests served within a certain time (ms)
  50%     14
  66%     46
  75%     47
  80%     47
  90%     47
  95%     47
  98%     50
  99%    640
 100%  27343 (longest request)
```

## Benchmark 3. concurrency 300 request number 50000
``` console 
choi@airi:~/myproject$ ab -k -c 300 -n 50000 localhost:80/
```
``` console
Concurrency Level:      300
Time taken for tests:   27.495 seconds
Complete requests:      50000
Failed requests:        11592
   (Connect: 0, Receive: 0, Length: 11592, Exceptions: 0)
Non-2xx responses:      11592
Keep-Alive requests:    49542
Total transferred:      10474222 bytes
HTML transferred:       2741736 bytes
Requests per second:    1818.54 [#/sec] (mean)
Time per request:       164.967 [ms] (mean)
Time per request:       0.550 [ms] (mean, across all concurrent requests)
Transfer rate:          372.03 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.3      0       5
Processing:     0  116 1341.2      1   27472
Waiting:        0  116 1341.2      1   27472
Total:          0  116 1341.3      1   27476

Percentage of the requests served within a certain time (ms)
  50%      1
  66%      3
  75%     46
  80%     46
  90%     47
  95%     47
  98%    637
  99%    669
 100%  27476 (longest request)

```
## Benchmark 4. concurrency 400 request number 50000
``` console
choi@airi:~/myproject$ ab -k -c 400 -n 50000 localhost:80/
```

``` console
This is ApacheBench, Version 2.3 <$Revision: 1807734 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient)
apr_socket_recv: Connection reset by peer (104)
Total of 29 requests completed
```
number를 올리면 time taken for test가 예상하던데로 올라갔다.
재미있는 부분은 위의 benchmark 결과처럼 concurrency가 올라갈 수록 time taken for test가 감소할 줄 알았는데 증가한다.
또한, concurrency를 300으로 설정하였을 때는 failed request가 기하급수 적으로 올라갔으며, 400으로 설정하였을 땐 connection이 refused되었다.


# More tips
먼저 Nginx 웹서버만으로는 이미지/css/js 등의 정적 컨텐츠만 처리거나 WAS 와의 프록시 용도로만 사용된다.
이런 용도는 사실 셀러론 CPU에서도 동시 처리수가 초당 1만회를 상회하므로 벤치 결과에 큰 의미가 없다.
실제 서버 하드웨어 성능 상의 이점을 비교하려면 WAS 단계에서 부하 테스트가 이루어져야 할 것 같다.
<br>
ab가 원래 웹서버 쪽 성능 측정을 위해 만들어진 도구이다보니 세션을 사용한 복잡한 인증절차등을 포함한 페이지를 테스트하기에는 약간 부적합해보이고 간단한 API(헤더 정보는 활용할 수 있으니)나 정적컨텐츠의 서비스 성능 측정에는 적합해 보인다.
<br>
다음에는 Flask REST API에 Database를 붙여서 CRUD operation을 무작위로 적용하는 방식으로 API를 변경하고 나서 ngrider나 jmeter를 이용하여 서비스 중에 있다고 생각하고 여러 시나리오를 적용해 보도록 하자.
