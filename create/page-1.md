# Create

## Just

* 하나의 값을 전딜
* 파라미터로 받은 요소를 그대로 전달
* from과 혼동 주의

## Of

* 여러 요소 전달
* 배열은 한 덩어리로 방출

## from

* 배열의 요소를 전달

## range

* 시작값에서 1씩 증가하는 씨퀀스 생성
* 증가되는 크기를 바꾸거나 감소하는 것 불가능

## generate

* 원하는 값으로 변화하는 씨퀀스를 다룸
* initialState: 초기값
* condition: true일때만 방출, false면 complete 방출하고 종료
* iterate: 변화

## repeatElement

* 동일한 요소를 반복적으로 방출
* 무한정 반복하기때문에 방출되는 요소의 수를 제한해줘야 함
* take로 방출 갯수 조

## deferred

* 특정 조건에 따라 Observable 생성

## Create

* Observable의 동작을 직접 구현 가능

## Empty

* Observable.empty()
* completed 만 방출하고 종료
* 옵저버가 아무런 동작없이 종료해야할 때 사용

## Error

* Observable.error(MyError.error)
* error 이벤트를 전달하고 종료
* error 처리시 사용

