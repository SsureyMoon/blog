---
layout: post
comments: true
title:  "Docker에서 돌리는 Django React 스택 (1-1편) - 개발환경세팅(컨테이너 빌드)"
date:   2017-03-01 02:17:08 +0900
categories: development
---

# Docker에서 돌리는 풀스택(Django & React) 웹 애플리케이션 (1편) - 개발환경 세팅

## 로컬 개발 환경 구성도
![local system structure]({{ site.url }}{{ site.baseurl }}/assets/images/local-system-structure.png)

## 필요한 이유
각 팀원들이 맡은 기능들을 개발하고 그 자리에서 바로 확인할 수 있는 로컬 개발환경이 개발시에 필요합니다.

### 데이터베이스 컨테이너 생성
여기서는 Postgres 공식 도커 이미지를 이용하여 데이터베이스 컨테이너를 생성하고 동작시키도록 하겠습니다. 앱서버용 컨테이너 내에 PostgreSQL을 설치해서 동작시켜도 된다고 생각할 수도 있지만, 다음과 같은 이유로 데이터베이스 컨테이너를 별도로 분리하였습니다.
- 로컬 개발 환경에서의 컨테이너와 프로덕션 배포시의 컨테이너를 최대한 똑같이 만들수 있습니다. 프로덕션 배포시에는 원격 데이터베이스 서버(eg. AWS RDS)에 접속하기 때문에 앱 서버 컨테이너 내부에는 접속을 위한 드라이버만 설치되어 있으면 됩니다. 만약 로컬 앱서버 컨테이너에 데이터베이스를 설치한다면, 로컬환경 세팅시 추가적인 작업이 필요하게 됩니다.
- 개발을 하다보면 데이터베이스를 자주 지우거나, 다른 사람이 쓰던 데이터베이스를 연결해야 할 경우가 있습니다. 데이터베이스 컨테이너를 따로 분리하면, 새로운 데이터베이스를 연결하고 앱서버 컨테이너를 다시 생성할 수 있고, 기존 개발시 사용했던 데이터베이스 컨테이너에 다시 연결할 수도 있어 편리합니다.
-  Ubuntu의 경우는 확인을 하지 못했지만, Centos 컨테이너에 Postgres 서버를 설치하고 동작시킬때 `systemctl` 명령어의 권한 문제가 발생합니다. 관련 이슈: https://github.com/docker/docker/issues/2296

docker가 설치되어 있다면 터미널에서 다음의 명령어로 데이터베이스 컨테이너를 쉽게 생성할 수 있습니다.
```
docker pull postgres:9.4.10 # Postgres 9.4 공식 이미지
docker run --name {1.컨테이너 이름} -p {2.호스트포트}:{3.컨테이너포트} -e POSTGRES_DB={4.데이터베이스이름} -e POSTGRES_USER={4.데이터베이스} -e POSTGRES_PASSWORD={5.데이터베이스접속암호} -d {7.이 컨테이너를 생성할 이미지}
```
1. 컨테이너 이름: 이 데이터베이스 컨테이너의 별칭을 지정합니다. 이 별칭을 일종의 IP 처럼 사용해서 이용해서 도커명령어나 다른 컨테이너에서 이 데이터베이스 컨테이너에 접근할 수 있게 해줍니다.
2.
3. 호스트컴퓨터와 컨테이너의 포트를 연결해줍니다. Postgres의경우 기본 포트로 5432이용합니다. 1에서 정한 컨테이너 별칭과 혼합하여 `{별칭}:{포트번호}`로 앱서버 컨테이너에서 데이터베이스에 접근할 수 있습니다.
4. Postgres 공식 이미지에서 지원하는 환경변수입니다. 컨테이너에 생성될 데이터베이스의 이름을 지정합니다.
5. 4에서 지정한 데이터베이스의 사용자입니다.
6. 5에서 지정한 사용자의 암호입니다.
7. 이 컨테이너를 생성할 도커 이미지의 `이름:태그`를 지정합니다. 우리는 받아둔 postgres:9.4.10를 사용할 예정입니다.


예시:
```
docker run --name remotedb -p 5432:5432 -e POSTGRES_DB=db -e POSTGRES_USER=dbuser -e POSTGRES_PASSWORD=dbpassword1234 -d postgres:9.4.10
```
생성된 컨테이너를 확인해봅시다.
```
docker ps -a # 존재하는 모든 컨테이너들을 표시
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS                      PORTS                                            NAMES
11e39387be53        postgres:9.4.10     "/docker-entrypoin..."   About a minute ago   Up About a minute           0.0.0.0:5432->5432/tcp                           remotedb
```

### 앱서버 컨테이너 생성
기존에 만들어 두었던 도커 이미지는 도커허브에 업데이트 해두어 언제든지 그 이미지를 기반으로 다른 이미지를 생성할 수 있도록 하였습니다.(링크: https://hub.docker.com/r/ssureymoon/docker-image-build-centos7-python34-psql94/)
이번 포스트에서는 도커 허브에 있는 이미지를 바탕으로 장고앱서버용 이미지를 빌드하고, 컨테이너를 실행시켜보도록 하겠습니다.
사용할 Dockerfile은 다음과 같습니다.
https://github.com/SsureyMoon/django-react-docker-template/blob/master/Dockerfile

이 Dockerfile을 한줄 한줄 살펴보도록 합시다:
{% highlight docker %}
FROM ssureymoon/docker-image-build-centos7-python34-psql94:1.2

MAINTAINER Ssuery Moon (ssureymoon@gmail.com)
{% endhighlight %}
지난 포스트에서 만들어둔 이미지를 도커 허브에 업데이트해주었을때 지정한 이름:태그로 부터 새로운 이미지를 빌드합니다.
`ssureymoon/docker-image-build-centos7-python34-psql94:1.2` 태크로 버전을 사용할수도 있고, 다른 의미있는 정보를 표시할 수도 있습니다.
{% highlight docker %}
# create app dir & copy project folder
RUN mkdir -p /var/www/appserver
{% endhighlight %}
컨네이터 내에서 사용할 프로젝트 루트 디렉토리 입니다. 지금 이 명령으로 생성만 해놓은 상태이며, 추후 `docker run` 으로 호스트의 디렉토리와 동기화될 예정입니다.
이렇게 동기화를 시킴으로써, 호스트 컴퓨터에는 파이참이나, Atom, Visual Studio 코드같은 에디터를 사용하여 손쉽게 코드를 수정할 수 있고, 실제 프로그램의 실행은 도커 컨테이너 내에서 이루어지게 됩니다.
{% highlight docker %}
RUN npm cache clean

RUN npm install node-gyp@"^3.5" -g
{% endhighlight %}
그전 까진 node-gyp도 package.json에 넣고 `npm install` 로 설치했지만 계속 글로벌로 설치하라는 경고가 나와 Dockerfile에 추가하였습니다.

{% highlight docker %}
COPY package.json /tmp/package.json

COPY . /var/www/appserver

WORKDIR /var/www/appserver
{% endhighlight %}
`npm install` 을 좀더 빨리 설치할 수 있는 트릭입니다. 자세한 내용은 이 블로그 포스트(http://bitjudo.com/blog/2014/03/13/building-efficient-dockerfiles-node-dot-js/)를 참고해주세요.
{% highlight docker %}
RUN pip3.4 install -r requirements/local.txt
{% endhighlight %}
장고 앱서버를 동작시키기 위한 파이썬 패키지들을 설치합니다.
{% highlight docker %}
EXPOSE 8000
{% endhighlight %}
앱서버는 포트 8000번을 사용할 예정이므로 8000을 오픈합니다.
{% highlight docker %}
CMD ["sh", "/var/www/appserver/config/supervisor/run_supervisord.sh"]
{% endhighlight %}
실제 도커 컨테이너를 실행시키면 마지막에 어떤 프로그램을 돌리고 있을지 지정해줄 수 있습니다. 보통 쉘스트립트를 짜두고 여러가지 동작을 한번에 처리하고 마지막에 앱서버를 동작시키는 스크립트를 추가합니다. 도커로 앱서버를 배포했을때 마지막으로 실행되고 있는 스크립트라고 생각하시면 될 것 같습니다. 내부가 궁금하신 분은 다음 포스트나, 코드(링크: https://github.com/SsureyMoon/django-react-docker-template/blob/master/config/supervisor/run_supervisord.sh)를 참조해주세요.


#### 직접 따라해보자
호스트 컴퓨터에 코드를 다운로드 받습니다.
```
git clone git@github.com:SsureyMoon/django-react-docker-template.git
cd django-react-docker-template
```
현 디렉토리에 있는 Dockerfile을 이용하여 새로운 도커 이미지를 만듭니다.
```
docker build -t appserverimage .
```
컨테이너를 생성/실행합니다.
```
docker run --name appserver -p 8000:8000 -p 8080:8080 --link remotedb:remotedb -v [당신의 현재 프로젝트 디렉토리]:/var/www/appserver -ti appserverimage /bin/bash
```
위 명령어 실행 후 appserver 컨테이너 내부 터미널로 들어가셨다면 성공입니다. 마지막에 보시면 `/bin/bash` 명령어가 있는데, 이렇게 `docker run` 실행시 명시적으로 명령어를 지정해주면 Dockerfile 마지막의 CMD 명령어는 실행되지 않습니다. 이렇게 하여 현재 개발용으로 사용하기 편하도록 우리가 원할때 웹서버를 실행/정지, 콘솔 출력을 확인할 수 있습니다.
`-v` 로 디렉토리(볼륨) 동기화시킬때는 최대한 절대경로를 사용하는 것이 좋습니다

appserver 컨테이너 내에서 데이터베이스 서버로 잘 접속이 되는지 확인해 보도록 합시다.
아래와 같이 psql 쉘로 들어가면 성공입니다:
```
[root@5e394e400b02 appserver]# psql -h remotedb -p 5432 -d db -U dbuser
Password for user dbuser:
psql (9.4.10)
Type "help" for help.

db=#
```
이번엔 장고 커맨드로 접속을 확인해보겠습니다. 잠시 장고의 로컬 세팅을 확인해보도록 합시다.
{% highlight python %}
  DATABASES = {
      'default': {
          'ENGINE' : 'django.db.backends.postgresql_psycopg2',
          'NAME': env('PSQL_DB_NAME', default='db'),
          'USER': env('PSQL_DB_USER', default='dbuser'),
          'PASSWORD': env('PSQL_DB_PASSWD', default='dbpassword1234'),
          'HOST': 'remotedb',
          'PORT': '5432',
      }
  }  
{% endhighlight %}
데이터베이스 컨테이너 생성시 넘겨준 환경변수 값으로 잘 세팅이 되어있는지 확인하도록 합시다.
아래와 같이 psql 쉘로 들어가면 성공입니다.
```
[root@5e394e400b02 appserver]# python3.4 manage.py dbshell
psql (9.4.10)
Type "help" for help.

db=#
```

### 컨테이너 중지 시작
```
docker stop remotedb # 컨테이너 중지
docker start remotedb # 컨테이너 시작
```
데이터베이스 서버는 따로 콘솔출력을 확인할 필요가 없고, 내부에서 데몬으로 데이터베이스가 동작하기 때문에 쉘 연결이 없어도 컨테이너가 살아 있습니다.
```
docker stop appserver # 컨테이너 중지
docker start -i appserver # 컨테이너 시작
```
`-i`는 인터렉티브 쉘을 의미하며 컨테이너 시작과 동시에 컨테이너 내부의 터미널에 접속하겠다는 의미입니다.


## 마무리하며
이번 포스트에서는 앱서버용 컨테이너와 데이터베이스 컨테이너를 생성하고 연결이 되는지 확인하였습니다. 다음 포스트에서는 이 컨테이너들을 이용하여 본격적인 개발환경 세팅에 들어가도록 하겠습니다.
