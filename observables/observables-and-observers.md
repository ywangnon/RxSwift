# Observables and Observers

## **Observable**

* 흐름을 구현을 할 뿐, 데이터를 전달하지는 않는다.
* 옵저버가 구독을 시작하는 시점에서 데이터 전달

명칭

* Observable == Observable Sequence == Sequence
* Observer == Subscriber

관계

* Observable은 Observer에게 **이벤트**를 전달
* Observer은 Observable을 \*\*구독(Subscribe)\*\*을 해서 Observable을 감시

Observable이 Observer에게 전달하는 이벤트 종류

* Observable의 새로운 이벤트는 **Next** 이벤트를 통해 전달(Emission)
* Observable의 에러 이벤트는 **Error** 이벤트를 통해 전달(Notification)
* Observable의 정상적인 종료 이벤트는 **Completed** 이벤트를 통해 전달(Notification)
* Error와 Completed는 종료 이벤트로 다른 이벤트는 전달되지 않는다.



이미지로 표시하는 법

* [RxMarbles 참조](https://github.com/RxSwiftCommunity/RxMarbles)

## Observer

* 구독자
* Observable에게 이벤트를 전달받아 처리하는데 이를 구독한다 함
* subscribe를 사용해 구독함
* Observable은 데이터의 흐름이고, Observer는 Observable을 Subscribe하면 생성되는 각 이벤트에 대한 관찰자? 직접 처리하는 객체? 정도로 이해
* 동시에 2개 이상의 이벤트를 처리하지 않음
* Observable은 Observer가 일을 끝낸 다음에 다음 이벤트를 전달함

## 사용한 연산자

* create: Observable이벤트 생성
* on: 이벤트를 전달
* next: 다음 이벤트 생성
* onNext: on과 next를 한번에 사용할 수 있는 연산자
* onCompleted: 이벤트 종료 연산자
* subcribe: 구독 연산자
* from: 배열의 요소를 차례로 방출하고 마지막에 Completed이벤트를 전달하는 연산자
