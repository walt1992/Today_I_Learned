# Elastic Search 기초

Cluster > Node > shard - shard는 복제되어 여러 노드에 저장 (프라이머리 샤드 + 복제본)
같은 머신에서 사용하더라도 클러스터는 서로 독립적으로 작동한다.

Node가 유실될 경우 Shard는 자동으로 다시 복제되어 다른 node에 저장된다. 
###
키바나 실행



### 검색과정 


type은 Default로 _doc이며 커스터마이징은 가능하지만 이 값을 통해 어떤 구분값으로 활용할 수 는 없다. 7q버젼에는 type이라는 개념이 완전히 사라질 예정이다.


REST API 제공 > 검색가능 

### PUT POST

### bulk - 여러 다큐먼트를 한번에 입력하기
POST index명/_doc/bulk



### 매핑정보 확인하기
GET index명/_mapping

### 검색 방식 
match  - 공백을 or로 인식하여 검색
match_phrase - 공백포함 하나의 문자열로 인지

### bool 쿼리

+ bool + must 
 must내의 모든 조건을 만족하는 결과 검색

+ bool + must_not

+ bool + should
    boost 옵션 사용

+ bool + must+ should
    Must를 만족시기키고 should를 만족시키면 더 높은 score

### highlight


### Filter

스코어를 계산하지 않고 캐싱을 하기 떄문에 속도가 빠름

match에서 score로 검색된 결과에서 해당 조건을 필터링 하는 것으로 새롭게 scoring하지 않는다. 즉, 필터로 인해 score는 변하지 않는다.

숫자의 범위는 정확도를 비교하기 어렵다. 따라서 수치 범위에 따른 결과 검색은 주로 FIlter를 사용해 수행한다.

### _analyze
tokenizer

    default는 standard 이며 이 경우 문자열을 " "공백을 기준으로 나누어 저장한다.

    letter의 경우 알파벳만 토큰으로 저장한다.

### Aggregation

keyword와 text???


### 

