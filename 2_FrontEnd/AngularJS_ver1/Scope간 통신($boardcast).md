# Scope간 통신 ($broadcast, $on, $emit

## $broadcast(name, args)

현재 스코프에서 모든 자식 스코프로 이벤트를 전달. 인자로는 이벤트 명 및 이벤트와 함께 전달할 보조 데이터가 들어 있는 객체를 받는다.


## $emit(name, args)

현재 스코프에서 루트 스코프까지 이벤트를 위로 전달

## $on(name, (event,args) => { .....}))

스코프에서 특정 이벤트를 수신할 떄 호출할 핸들러 함수 등록

