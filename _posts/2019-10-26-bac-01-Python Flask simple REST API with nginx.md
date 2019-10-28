---
title: "[Backend] Python Flask simple REST API with nginx"
excerpt:
last_modified_at: 2019-10-24

categories:
  - Backend

tags:
  - python flask
  - REST API
  - WAS
  - gunicorn
  - nginx
  - docker
---

![Python-version-3.6.8](https://img.shields.io/badge/python-v3.6.8-blue.svg)
![Flask-version-1.1.1](https://img.shields.io/badge/flask-v1.1.1-red.svg)
![werkzeug-version-0.16.0](https://img.shields.io/badge/werkzeug-v0.16.0-black.svg)
![nginx-version-1.14.0](https://img.shields.io/badge/nginx-v1.14.0-green.svg)
![docker-version-18.09.7](https://img.shields.io/badge/docker-v18.09.7-yellow.svg)


# Install nginx
``` console
choi@airi:~$ sudo add-apt-repository ppa:nginx/stable
choi@airi:~$ sudo apt-get update && sudo apt-get -f install nginx
choi@airi:~$ /etc/init.d/nginx status
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2019-10-28 16:19:11 KST; 8min ago
     Docs: man:nginx(8)
 Main PID: 11982 (nginx)
    Tasks: 9 (limit: 4915)
   CGroup: /system.slice/nginx.service
           ├─11982 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
           ├─11983 nginx: worker process
           ├─11984 nginx: worker process
           ├─11985 nginx: worker process
           ├─11986 nginx: worker process
           ├─11987 nginx: worker process
           ├─11988 nginx: worker process
           ├─11989 nginx: worker process
           └─11990 nginx: worker process
choi@airi:~$ /usr/sbin/nginx -v
nginx version: nginx/1.16.1
```
하나의 master process는 많은 worker process들을 가지고 있다.
마스터 프로세스는 configuration file를 처리하고 worker process들을 관리하는 역할을 한다. 
위치에 대한 정보는 아래와 같다.
- nginx는 /usr/sbin/nginx 에서 실행이 가능하다. 
- configuration files은 /etc/nginx 에 저장되어 있다.
- Log file들은 /var/log/nginx 에서 볼 수 있다.

# Set Virtual Environment and Install Flask
'''console
choi@airi:~/myapp$ python -m virtualenv rest
choi@airi:~/myapp$ source rest/bin/activate
(rest) choi@airi:~/myapp$ pip install flask flask_restful
'''

# Simple Flask REST API server
In myproject.py

```python
from flask import Flask
from flask_restful import Resource, Api

app = Flask(__name__)
api = Api(app)

class HelloWorld(Resource):
    def get(self):
        return {'hello': 'world'}

api.add_resource(HelloWorld, '/')

if __name__ == '__main__':
    app.run(debug=True)
```
``` console 
flask run --host=0.0.0.0 --port=5000
```
만약 `address already in used`가 발생 시 해당 포트를 점유하는 process를 kill한다.
```console
netstat -lntp
kill -9 pid
```
# Install gunicorn


# Reference
 - [https://m.blog.naver.com/PostView.nhn?blogId=ijoos&logNo=221115649847&proxyReferer=https%3A%2F%2Fwww.google.com%2F](https://m.blog.naver.com/PostView.nhn?blogId=ijoos&logNo=221115649847&proxyReferer=https%3A%2F%2Fwww.google.com%2F)
 - [https://flask-restful.readthedocs.io/en/0.3.5/quickstart.html](https://flask-restful.readthedocs.io/en/0.3.5/quickstart.html)
 - [https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-gunicorn-and-nginx-on-ubuntu-18-04](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-gunicorn-and-nginx-on-ubuntu-18-04)


