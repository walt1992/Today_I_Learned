# Docker로 Mysql 설치 및 접속하기

도커는 지난번 스프링부트를 공부하며 잠시 맛보기로 다뤄본 경험이 있다. 이번엔 Node.js를 공부하는 과정에서 Mysql과 연동해야하는 튜토리얼이 있는데, 이를 도커를 통해 해보고자 시도해 보았다.

먼저 도커를 통해 MySql을 설치하는 명령어는 다음과 같다.

    docker pull mysql

위와같이 mysql을 pull하면 최신 MySql 컨테이너를 설치할 수 있다. 
이제 설치된 MySql 이미지를 확인해 보자.


    $ docker images

    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    mysql               latest              b8fd9553f1f0        12 days ago         445MB
    neo4j               latest              f6e40b47cbd6        7 months ago        203MB
    redis               latest              0f55cf3661e9        7 months ago        95MB
    postgres            latest              5a02f920193b        7 months ago        312MB
    debian              latest              d508d16c64cd        7 months ago        101MB
    mongo               latest              0da05d84b1fe        7 months ago        394MB

오...지난번 실습했을 때 사용했었던 다른 DB와 NoSql의 이미지과 함께 새로 설치한 mysql이 설치되었음을 확인할 수 있다. 

이제 mysql 컨테이너를 실행시켜 보자

    docker run --name mysql -e MYSQL_ROOT_PASSWORD=mypass -d -p 3306:3306 mysql

> 실행할 이미지의 이름과 루트패스워드 설정, 연결 포트 지정 그리고 생성할 컨테이너의 이름을 지정한다.

위와같이 컨테이너를 생성한 후 다음 명령어를 통해 생성된 컨테이너를 조회할 수 있다.

    $ docker ps

    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
    8f3ca19fe69f        mysql               "docker-entrypoint.s…"   17 minutes ago      Up 17 minutes       0.0.0.0:3306->3306/tcp, 33060/tcp   mysql

이제 컨테이너에 접속해보자

    $ docker exec -it mysql /bin/bash

    root@8f3ca19fe69f:/#                

`8f3ca19fe69f`라고 되어있는 `CONTAINER ID`를 통해 접속되었음을 확인할 수 있다. 

이제 컨테이너 안에서 mysql을 실행해 보자

    root@8f3ca19fe69f:/# mysql -u root -p
    Enter password: 

위에 컨테이너 생성시 입력했던 패스워드를 입력하고 나면 성공적으로 mysql에 접속한것을 확인할 수 있다.


