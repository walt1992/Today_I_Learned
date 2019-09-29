# Sequelize의 모델명과 테이블명 기본형의 차이

시퀄라이즈 튜토리얼 중, 이상한 현상을 발견하였다.

먼저, mysql에 미리 정의한 테이블이다. 

    CREATE TABLE nodejs.user (
        id INT NOT NULL AUTO_INCREMENT,
        name VARCHAR(20) NOT NULL,
        age INT UNSIGNED NOT NULL,
        married TINYINT NOT NULL,
        comment TEXT NULL,
        created_at DATETIME NOT NULL DEFAULT now(),
        PRIMARY KEY(id),
        UNIQUE INDEX name_UNIQUE (name ASC)
    )
    COMMENT = '사용자 정보'
    DEFAULT CHARSET=utf8
    ENGINE=INNODB;

다음은 시퀄라이즈에 정의한 모델이다.

    module.exports = (sequelize, DataTypes) => {
        return sequelize.define( 'user', {
            name : {
                type : DataTypes.STRING(20),
                allowNull : false,
                unique : true,
            },
            age : {
                type : DataTypes.INTEGER.UNSIGNED,
                allowNull : false,
            },
            married : {
                type : DataTypes.BOOLEAN,
                allowNull : false,
            },
            create_at : {
                type : DataTypes.DATE,
                allowNull : false,
                dafaultValue : DataTypes.NOW,
            },
        },{
            timestamps : false,
        })
    }

겉보기로는 특이한 문제가 없어 보이지만, 시퀄라이즈를 통해 user 테이블의 값을 생성하는 과정에서 자꾸만 에러가 발생하였다. 

    Unhandled rejection SequelizeValidationError: notNull Violation: user.create_at cannot be null
    at Promise.all.then (C:\Web-Develop\node_tutorial\mysql_with_node\node_modules\sequelize\lib\instance-validator.js:74:15)

왜그럴까... 분명 create_at이라는 변수는 자동으로 생성되게끔 테이블을 생성할 때 설정해 두었는데, 에러가 나길래 확인해 보았다.

근데 이상하게 테이블을 조회해보니

    - users
    - user

두개의 테이블이 생성되어 있었다. 

user라는 이름의 테이블과 user라는 모델을 직접 만들어 주었는데 저 users라는 테이블은 대체 왜 생성이 된거지...

하고 로그를 살펴보았다. 다음은 프로그램을 실행시켰을 때 시퀄라이즈가 찍은 로그이다.

    Executing (default): CREATE TABLE IF NOT EXISTS `users` (`id` INTEGER NOT NULL auto_increment , `name` VARCHAR(20) NOT NULL UNIQUE, `age` INTEGER UNSIGNED NOT NULL, `married` TINYINT(1) NOT NULL, `create_at` DATETIME NOT NULL, PRIMARY KEY (`id`)) ENGINE=InnoDB;

아... users라는 테이블이 자동으로 생성된거구나

왜그럴까 그 이유를 찾아보았다.

나와 비슷한 문제가 역시나 많았다. 시퀄라이즈에서 자동으로 테이블을 생성할 때에는 자동으로 모델명을 복수형으로 바꾸어 테이블을 생성한다고 한다. 
만약 이러한 자동 설정이 싫다면 모델을 정의할 때 다음과 같이 매개변수를 추가해주면 된다.


    module.exports = (sequelize, DataTypes) => {
        return sequelize.define( 'user', {
            name : {
                type : DataTypes.STRING(20),
                allowNull : false,
                unique : true,
            },
            age : {
                type : DataTypes.INTEGER.UNSIGNED,
                allowNull : false,
            },
            married : {
                type : DataTypes.BOOLEAN,
                allowNull : false,
            },
            create_at : {
                type : DataTypes.DATE,
                allowNull : false,
                dafaultValue : DataTypes.NOW,
            },
        },{
            timestamps : false,
            freezeTableName: true // 테이블명 고정하기
        })
    }

`freezeTableName: true` 라는 설정을 해주면 현재 생성한 모델명과 DB상의 테이블명이 동일하게 설정된다.

