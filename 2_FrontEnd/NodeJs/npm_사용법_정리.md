# npm 사용법 정리

최근 NodeJs에 대해 공부를 시작하면 조현영씨의 NodeJs교과서를 참고하고 있다. 항상 설정하는 방법이 가장 어렵게 느껴졌는지라, npm을 활용해 패키지를 생성하고 내려받는 일련의 과정을 간단히 요약해서 정리해 보았다. 

> npm이란? 
>Node Package Manager의 약자로 자바스크립트 패키지가 저장되어 있는 저장소

## npm을 활용해 프로젝트 시작하기

* ###  패키지 생성 package.json

프로젝트를 시작할 폴더로 이동하여 콘솔에 다음을 입력

    npm init

init을 하고나면 다음과 같이 입력값을 순차적으로 받는다.

```
package name: (node_tutorial) -> 프로젝트 명
version: (1.0.0) -> 현재 생성한 패키지의 버젼 정보
description: -> 상세 설명
entry point: (index.js) -> 자바스크립트 실행파일 진입점
test command: -> 코드를 테스트할 때 입력할 명령어 스크립트 지정
git repository: -> 코드가 저장된 git 저장소 주소
keywords: -> npm에 저장될 패키지를 찾기위한 키워드
author: MinsikPark -> 작성자
license: (ISC)
About to write to C:\Web-Develop\node_tutorial\package.json:
```

위의 입력을 모두 하고나면 다음과 같은 내용을 담은 package.json 함수가 생성된다.

```
{
  "name": "node_tutorial",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "MinsikPark",
  "license": "ISC"
}

```

이제 개발에 필요한 모든 자바스크립트 모듈에 대한 정보는 이 package.json에 저장되게 된다.

* ### 패키지 설치하기

패키지를 설치하는 방법은 기본적으로 다음과 같다

    npm  install [패키지명]

이를 활용해 express라는 패키지를 설치하면

    npm install express

방금 생성한 package.json에 다음과 같은 내용이 추가된다. 

```
 "dependencies": {
    "express": "^4.17.1",
 }
```

마치 spring에서 build.gradle파일에 설치한 의존성의 내용이 명시되는 것과 비슷한 방식으로 설정한 패키지의 내용이 저장된다.

뿐만아니라 현재 진행중인 프로젝트의 디렉토리 안을 살펴보면 node-modules라는 파일이 생성된다. 해당 파일안에는 방금 설치한 express라는 패키지와 그와 관련된 부수적인 패키지들이 설치되어있다.

그럼 이와 비슷한 방식으로 몇가지 패키지를 더 설치해보자.

    npm install morgan cookie-parser express-session

위처럼 연속해서 패키지를 설치할 수 있다. 


* ### 개발모드용 패키지 설치하기
또한 패키지에 따라 개발 환경에서만 활용하고 싶은 패키지들이 있을 수 있는데 이러한 경우 

    npm install --save-dev [패키지명]

와 같이 `--save-dev` 라는 키워드를 추가해주면 된다. 



해당 명령어를 사용해서 다음 패키지를 설치한다.

    npm install --save-dev nodemon

위 패키지는 코드를 수정시 자동으로 재실행해주는  기능을 제공한다. 

이렇게 개발모드로 패키지를 다운받게 되면 package.json에 다음과 같이 패키지 정보가 입력된다. 

    "devDependencies": {
        "nodemon": "^1.19.2"
    }


이렇듯, 프로젝트에 활용할 수 있는 패키지들을 설치하면 해당 프로젝트의 package.json에 설치된 패키지들의 정보가 입력된다.

해당 package.json 파일만 있으면 실제 패키지가 저장된 node-modules 파일이 없다 하더라도 필요한 패키지를 동일하게 불러와 저장할 수 있으므로 package.json은 매우 중요한 파일이라고 할 수 있다.

package.json 파일을 활용해 패키지를 다운받고 싶다면 손쉽게

    npm install

명령어만 입력해주면 된다.

* ### 글로벌 패키지 설치하기

또한 npm에서 패키지를 로컬 전역에서 활용할 수 있게끔 설치하는 방법이 있다.

    npm install --global [패키지명]

이 방법으로 다음 패키지를 설치한다.

    npm install --global rimraf

이렇게 전역으로 설치된 패키지는 현재 npm이 설치되어 있는 디렉토리 아래에 설치되며 다음 메시지를 통해 이를 확인할 수 있다.

    C:\Web-Develop\node_tutorial>npm install --global rimraf
    C:\Users\Minsik Park\AppData\Roaming\npm\rimraf -> C:\Users\Minsik Park\AppData\Roaming\npm\node_modules\rimraf\bin.js
    + rimraf@3.0.0
    updated 1 package in 0.789s

이렇게 전역에 설치된 패키지는 콘솔을 통해 바로 실행할 수 있다. 

지금 설치한 `rimraf` 는 지정한 파일을 삭제할 수 있도록 하는 명령어로 콘솔에

    rimraf [파일 혹은 폴더명]

을 사용해 해당 파일을 삭제할 수 있다.

* ### 기타 npm 명령어

1)  업데이트 가능한 패키지 버전정보 확인하기

        npm outdated

2) 패키지 업데이트 하기

        npm udpate [패키지명]

        npm update ( -> 전체 패키지 업데이트)

3) 패키지 제거

        npm uninstall [패키지명]
        
        npm rm [패키지명]
    