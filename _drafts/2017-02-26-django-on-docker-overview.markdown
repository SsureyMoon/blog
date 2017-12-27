---
layout: post
comments: true
title:  "Docker에서 돌리는 Django React 스택 (0편) - 시스템 개요"
date:   2017-02-26 17:24:46 +0900
categories: development
---

# 개요
개발환경이 구축되어 있지 않은 회사에 들어가서 개발환경 설정, 배포 자동화 등을 처음부터 해야 했었습니다. 기왕 하는 것, 최대한 유행하는 기술들을 쓰기로 마음먹고 공부하여 개발 업무를 진행해나갔습니다. 저 같은 초보 개발자들이 단순히 코드만 배우는 것이 아니라, 개발환경을 어떻게 설정하고, 어떻게 프로덕션에 배포하는지 등의 플로우를 공부하는데 이 포스트들이 도움이 되기를 바랍니다.

개발자로 일하게 된다면 다음과 같은 체계가 갖추어진 회사에서 일하는 것이 좋습니다. 만약 그런 것들이 갖추어지지 않은 회사라면 여러분들이 대표를 설득해서 그런 체계를 만들거나 빨리 탈출하는 것이 좋습니다. 아래는 개발 환경, 체계들에 대한 저 나름대로의 생각입니다.

### 개발 환경 구축
각자의 작업 컴퓨터에 코드 개발과 코드 실행이 가능하도록 해주는 모든 것들을 개발 환경 구축이라고 생각합니다. 예를 들어 PHP로 쇼핑몰을 만드는 회사가 있다면, 그 회사에서는 코드를 어디서 받고, 그 코드를 동작시키기 위해 작업용 컴퓨터에 어떤 소프트웨어들을 설치해야 하는지, 어떤 스텝들을 밟아서 로컬 호스트에 그 코드로 동작하는 쇼핑몰을 띄울 수 있는지 까지 안내해주는 매뉴얼을 작성하는 것이 최소한 개발 환경입니다. 거기에 더해 코드의 구조에 대한 설명, 테스트 코드와 테스트하는 방법 등이 구비되어 있으면 좋습니다.

### 배포 자동화
개발한 소프트웨어는 배포를 통해 외부로 전달됩니다. 배포 시에 정해진 과정이 없다면 배포된 후에 예상치 못한 문제가 발생할 수 있습니다. 자동화된 배포 과정으로 이러한 문제를 최대한 줄일 수 있습니다. 또한 배포된 버전에 문제가 발생되었을 경우, 이전의 안정된 버전으로 롤백하는 기능도 필요합니다.

프로덕션 배포 시 실제 프로덕션과 똑같은 환경이지만 특정 유저만 접속해서 상용할 수 있는 스테이징 서버를 두어 프로덕션 배포 시의 위험성을 낮출 수 있습니다.

### 지속적 통합(CI) 세팅
개발환경을 세팅하고 팀원들끼리 작업을 나누어서 개발을 하다 보면 서로의 코드를 합쳐야 할 경우가 생깁니다. 이때 특정 코드 베이스에 지속적으로 통합하고, 통합된 코드가 빌드가 잘되고 테스트를 통과하는지 확인하는 방법으로 품질을 높일 수 있습니다. 또한 개발자 각자의 컴퓨터에 최신 코드를 통합으로써 생산성을 높일 수 있습니다.

## 도커로 선택한 이유
다음과 같은 장점 때문에 도커를 선택하였습니다.
- 개발에 사용했던 환경 세팅 그대로 프로덕션 배포에 사용할 수 있습니다. 로컬에서 동작했는데 배포하고 보니 동작하지 않는 등의 골치 아픈 환경 의존적인 문제로부터 자유로울 질 수 있습니다.
- 다른 사람들이 다양한 OS를 사용해도 개발환경을 통일할 수 있습니다. 동일인이 다른 컴퓨터에서 작업하는 경우에도 의존성 문제를 쉽게 해결할 수 있습니다.
- 완전한 새 OS를 설치하는 Virtual Machine 도커엔진이 보다 가볍고 빠릅니다.
- 이미지를 쉽게 공유할 수 있고, 버전업을 하여 관리할 수 있습니다.

하지만 단점도 있습니다.
- 도커 컨테이너 내에서 메모리 효율이 떨어집니다.
- 수시로 안 쓰는 이미지를 지워주지 않으면 저장공간이 어느새 도커 이미지로 가득 차게 됩니다.
- 배우는데 시간이 걸립니다. 리눅스 설정법과 더불어 도커 명령어와 Dockerfile 작성법을 배워야 합니다. 하지만 제 개인적인 생각으로는 Vagrant+Puppet 보다 학습 난이도는 낮은 것 같습니다.

무엇보다도 상대적으로 적은 인력 자원으로 개발환경 세팅과 배포 자동화라는 두 마리 토끼를 잡을 수 있었기 때문에 선택하게 되었습니다. 하나의 도커 이미지를 개발, 프로덕션 두 경우 모두에 사용할 수 있었습니다.

### 개발 & 배포 환경
각 환경에 대한 사양은 다음과 같습니다. 실제업무와 최대한 가깝도록 할 예정입니다.
- 로컬 환경(각자의 개발용 컴퓨터, localhost로 테스트 하는 수준)
- 온라인 개발 환경(서버 1대에 베이스 리)
- 스테이징 환경
- 프로덕션 환경

### 개발환경
호스트 머신: 개인 개발용 Mac 또는 Windows
도커 컨테이너:
  - appserver용 1개
    - 미리 제작한 이미지와 Dockerfile를 이용하여 생성
    - Centos7
    - Python 3.4.x
    - Postgres9.4 Driver 만 설치(모든 환경에서 원격 서버로 데이터베이스 접속)
    - Geos9.5 for Postgis(위치기반 쿼리를 가능하게 해주는 패키지, 추후 예제를 위해 설치)
    - Nodejs 6(프론트엔드 애플리케이션)
    - Ruby & SASS(프론트엔드 스타일언어로 SASS 사용)
    - **이후 사용자가 원하는 파이썬 웹 프레임워크를 설치해서 사용가능**
  - dbserver용 1개
    - Postgres 공식 Docker 이미지
    - Postgres 9.4.x


#### appserver용 도커 컨테이너 이미지
기본으로 필요한 모든 패키지를 설치해 놓고 Docker hub에 이미지를 빌드하여 올려놓았습니다.
이 이미지를 생성하는데 사용된 Dockerfile을 한줄 한줄 살펴보도록 합시다:
{% highlight docker %}
FROM centos:centos7

MAINTAINER Any Names (any.names@anyemails.tech)
{% endhighlight %}
- `FROM centos:centos7` 이 이미지의 메인 OS로 Centos7을 사용하였다. Centos7 공식 이미지에서 시작해서 우리가 필요한 설정들을 추가할예정입니다.
- `MAINTAINER ...` 이 이미지를 유지&보수할 사람의 이름과 메일주소를 적어줍니다.

{% highlight docker %}
RUN yum install -y deltarpm
RUN yum -y update
RUN yum -y groupinstall 'Development Tools'
RUN yum -y install epel-release
RUN yum -y install centos-release-scl
RUN yum -y install wget zlib-devel bzip2-devel openssl-devel libjpeg-devel nano
{% endhighlight %}
- 기본으로 설치해야 할 리눅스 패키지들을 설치 및 업데이트 합니다.
- 시간을 절걍하기 위해서 해당 패키지들을 설치한 상태에서 새로운 이미지를 빌드할 수도 있습니다. 그 경우 그 패키지들이 이미지에 이미 존재하게 됩니다.

{% highlight docker %}
RUN yum -y install supervisor
{% endhighlight %}
- 앱서버를 돌릴때(로컬환경 제외) Task Manager로 Supervisor를 사용하기 때문에 설치해둡니다.

{% highlight docker %}
RUN yum-builddep -y python
RUN wget -qO-  https://www.python.org/ftp/python/3.4.5/Python-3.4.5.tgz | tar xz
WORKDIR /Python-3.4.5
RUN ./configure --prefix=/usr/local
RUN make && make altinstall
WORKDIR /
RUN rm -rf Python-3.4.5
{% endhighlight %}
- 파이썬 9.4를 설치합니다. 설치방법은 파이썬 공식 사이트를 참고하면 됩니다.
- 다른 버전을 설치하고자 하는 경우, 이 부분을 수정하고 새로 이미지를 빌드하면 됩니다.
- 만약 Ubuntu를 사용하고자 하시는 경우 Python 공식 도커 이미지를 사용하시는 것을 추천드립니다.

{% highlight docker %}
RUN yum -y install https://download.postgresql.org/pub/repos/yum/9.4/redhat/rhel-7-x86_64/pgdg-centos94-9.4-2.noarch.rpm
RUN yum -y install postgresql94-devel libjpeg-turbo-devel libpq94-devel gcc-c++ python34-devel
{% endhighlight %}
- Postgres 9.4 를 직접 설치하지는 않지만, 원격으로 Postgres9.4 데이터베이스에 연결해서 사용하지 위해서 드라이버만 설치합니다.

{% highlight docker %}
RUN wget -qO- http://download.osgeo.org/geos/geos-3.5.0.tar.bz2 | tar xj
WORKDIR /geos-3.5.0
RUN ./configure && make clean && make && make install
RUN ldconfig
WORKDIR /
RUN rm -rf geos-3.5.0
{% endhighlight %}
- Postgres에서는 경도, 위도 등을 이용하여 지역정보를 Query할 수 있습니다. 이를 위해 Postgres의 확장 프로그램인 Postgis를 사용해야 하는데, geos라는 패키지를 설치해야 합니다.
- 지금 당장은 사용하지 않지만 추후 예제를 위해 설치해놓았습니다. 개인적으로 이미지를 빌드할 때 위 부분을 빼도 무방합니다.

{% highlight docker %}
ENV PATH $PATH:/usr/pgsql-9.4/bin
ENV LD_LIBRARY_PATH=/usr/local/lib
{% endhighlight %}
- 추가적인 설정으로 psql 명령어가 실행될수 있도록  PATH와  LD_LIBRARY_PATH를 지정해줍니다.

{% highlight docker %}
RUN localedef -c -f UTF-8 -i en_US en_US.UTF-8
ENV LC_ALL=en_US.UTF-8
{% endhighlight %}
- 이 이미지의 locale 를 en_US.UTF-8로 설정합니다.

{% highlight docker %}
RUN curl --silent --location https://rpm.nodesource.com/setup_6.x | bash -
RUN yum -y install nodejs
{% endhighlight %}
- 프론트엔드 애플리케이션을 작성하기 위해 node.js 와 npm을 설치합니다.
- 별도의 프론트엔드 없이 장고의 템플릿만을 이용할 경우 위 부분을 빼도 됩니다.

{% highlight docker %}
RUN yum -y install ruby
RUN gem install sass
{% endhighlight %}
- 프론트엔드 스타일언어로 SASS를 사용합니다. SASS를 사용하기 위해서는 Ruby도 필요하기 때문에 함께 설치해주었습니다.

{% highlight docker %}
CMD ["echo", "done installation"]
{% endhighlight %}
- `CMD`는 `docker run ...` 커맨드 사용 시 별도의 커맨드를 입력하지 않을 경우 Default로 사용될 커맨드를 지정해줍니다. 베이스 이미지로 사용하기 위해 만들었기 때문에 이 이미지 자체로 무언가를 돌리는 경우는 없다고 볼 수 있기 때문에 단순한 `echo` 커맨드만을 실행시키도록 하였습니다.

### 다음 포스트로..
이제 도커 이미지가 생성되고 개발을 시작할 준비가 되었습니다. 다음 포스트에선 로컬 개발 환경에 대해 설명할 예정입니다.
