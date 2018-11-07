# git 명령어 정리

이제 가급적이면 git을 Command 환경에서 사용하는 연습을 해보려 한다. 

따라서 필요한 git 명령어가 생길때 마다 이곳에 기록하려한다.


+ ### **git 등록하기**
    
    ```
    git init
    ```

+ ### **Stage 올리기**

    ```
    git add (파일명) 또는 .
    ```

+ ### **변경사항 취소하기**

    ```
    git checkout (파일명) 또는 .
    ```

+ ### **커밋하기**

    ```
    git commit -m "메시지"
    ```

+ ### **원격 저장소 등록하기**

    ```
    git remote origin (URL)
    ```

+ ### **원격에 pushg하기(master)**

    ```
    git push -u orgin master
    