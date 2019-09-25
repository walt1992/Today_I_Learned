# MySql Docker 컨테이너 Dbeaver 연동하기

>사람은 언제나 똑같은 실수를 반복한다. 몇 달전 Docker를 통해 postgreSQL 컨테이너를 생성한 후 [DBeaver를 통해 접속하는 중 가상머신의 포트포워딩 설정을 하지 않아 헤맸던 경험을 한 적이 있다.](./PostgresDB_연결_Refused.md) 이번에도 비슷한 경험을 다시하게 되어 이렇게 정리하게 되었다. 다만 그때 정리한 내용과 비슷한 내용이지만, mysql에 연동하기 위한 추가 설정에 대한 내용도 있어 다시 한번 정리한다.

처음 문제는 동일했다. 가상머신의 포트포워딩 설정을 안해준 것. Window 10에서 도커를 사용해 컨테이너를 띄우고 연결하기 위해서는 반드시 함께 설치된 가상머신 인스턴스에서 아래와 같이 타겟 포트를 포워딩 해주어야 한다. 

![가상머신 포트포워딩](/img/virtual_machine_port_forwarding.png)


그런데 mysql의 경우 포트포워딩을 해주었음에도 불구하고 다음과 같은 에러가 나며 연결이 거부되었다. 


    com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException: Public Key Retrieval is not allowed


이를 간단히 해결하는 방법은 다음과 같다. 먼저, DBeaver의 Connection Settings에 들어간다. 이후 Dirver Properties탭을 클릭한다.

![DBeaver connection](/img/DBeaver_connection_opt.png)

해당 설정 프로퍼티들 중에서 다음 값들을 변경한다

1. allowPublicKeyRetrieval = true
2. useSLL = false

이후 Test Connection을 실행하면 Mysql에 정상 연결되는 것을 확인할 수 있다.
