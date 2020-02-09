---
layout: post
title:  "Nginx와 gunicorn을 이용한 django, flask 앱 배포"
date:   2020-02-09 15:32:12 +0900
categories:
tags:
    - nginx
    - 파이썬
    - django
    - flask
    - uwsgi
    - reverse_proxy
image: https://user-images.githubusercontent.com/8157830/74097673-48dbfe80-4b52-11ea-8576-4a15be71f7b1.png
---

# 목표
파이썬 웹 프레임워크로 작성된 웹서비스를 실제 서버에서 서비스 할 수 있도록 하자.

## 들어가기 전에
이 글에서는 새로운 앱을 만들어서 예제를 작성했다.
django는
```bash
django-admin startproject webservice
```
flask는 `app.py`를 생성하고 아래와 같이 작성했다.
```python
from flask import Flask

app = Flask(__name__)


@app.route('/')
def index():
    return 'Hello Flask'


if __name__ == '__main__':
    app.run('localhost', 8080)
```

# 생각해 볼 수 있는 방법
1. 웹 프레임워크 내장 서버를 이용
1. 외부 서비스를 이용

## 웹 프레임워크 내장 서버를 이용
django나 flask 또는 그외 파이썬 웹 프레임워크로 작성한 웹 서비스들은 대부분 자체적으로 서버를 열수 있는 도구를 가지고 있다. django는 `python manage.py runserver <ip>:<port>`로, flask는 코드 내에서 `app.run()` 함수를 실행시켜서 서버를 열 수 있다.

![django runserver 결과](https://i.imgur.com/fEQsZnb.png)

django는 `runserver`를 이용해서 서비스를 하는 것은 보안이나 성능상의 문제가 있을 수 있어서 추천하지 않고 해당 명령어는 개발용으로만 사용할 것을 권장한다.

[해당 docs 내용](https://docs.djangoproject.com/en/3.0/ref/django-admin/#runserver)

또한 80번 포트로 직접 열어서 웹서비스를 하게 되면 https 통신도 불가능하기 때문에 django이건 flask이건 좋은 방법은 아니라고 생각한다.

## 외부 서비스를 이용
그래서 보통은 application server를 이용하게 된다. 파이썬에서는 uwsgi를 많이 사용하는데, 개인적으로 uwsgi보다는 gunicorn을 선호하기 때문에 이 글에서는 gunicorn을 기준으로 설명하려고 한다.

uwsgi를 이용해서 서비스를 하고싶다면 아래 링크를 참고하면 좋을 것 같다.

[장고(django) 떠먹여 주는 UWSGI와 NGINX 연동](https://baejino.com/programing/django/how-to-run-server-with-uwsgi)

gunicorn을 이용해서 서비스 하기 위해서 gunicorn을 설치해주자.

```bash
pip install gunicorn
```

그 후 gunicorn에 wsgi 경로를 주면 웹서비스를 실행해준다.
### django
```bash
.../webservice$ gunicorn webservice.wsgi
[2020-02-09 16:34:16 +0900] [38139] [INFO] Starting gunicorn 20.0.4
[2020-02-09 16:34:16 +0900] [38139] [INFO] Listening at: http://127.0.0.1:8000 (38139)
[2020-02-09 16:34:16 +0900] [38139] [INFO] Using worker: sync
[2020-02-09 16:34:16 +0900] [38162] [INFO] Booting worker with pid: 38162
```
`localhost:8000`으로 들어가면 아까와 같은 웹페이지가 뜨는 것을 볼 수가 있다.

### flask
```bash
.../flask-project$ prun gunicorn app:app
[2020-02-09 16:36:29 +0900] [38427] [INFO] Starting gunicorn 20.0.4
[2020-02-09 16:36:29 +0900] [38427] [INFO] Listening at: http://127.0.0.1:8000 (38427)
[2020-02-09 16:36:29 +0900] [38427] [INFO] Using worker: sync
[2020-02-09 16:36:29 +0900] [38450] [INFO] Booting worker with pid: 38450
```
`<file_name>:<flask_app_name>`으로 넘겨줘서 실행하면 django와 마찬가지로 gunicorn 서버가 실행된다.


gunicorn은 기본적으로 `127.0.0.1:8000`(=localhost)을 host로 서버를 실행한다. 따라서 외부에서는 접근할 수 없다.
![localhost로 열렸을 때 외부접속 결과](https://i.imgur.com/yCpe3UW.png)

gunicorn을 실행할 때 `-b`(=bind) 옵션을 같이 주면 host을 지정할 수 있고, host ip와 port를 변경할 수 있다.
```bash
/webservice$ prun gunicorn -b <ip>:8000 webservice.wsgi
[2020-02-09 16:52:43 +0900] [19022] [INFO] Starting gunicorn 20.0.4
[2020-02-09 16:52:43 +0900] [19022] [INFO] Listening at: http://<ip>:8000 (19022)
[2020-02-09 16:52:43 +0900] [19022] [INFO] Using worker: sync
[2020-02-09 16:52:43 +0900] [19030] [INFO] Booting worker with pid: 19030
```
![host 지정 후 열었을 때 외부접속 결과](https://i.imgur.com/tIlHs4L.png)

이 방법으로는 ip로 직접 접속할 수 있으므로 추천하지 않는다. 서버는 localhost로 열고 외부와 연결은 뒤에 나올 nginx에서 처리하도록 하자.

# Nginx와 연결

# 동시에 여러 접속 처리하기
// FIXME: gunicorn worker
