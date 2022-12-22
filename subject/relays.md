# Relays

* next 이벤트만 처리
* complete 이벤트, error 이벤트를 처리하지 않음
* 구독자가 dispose 될 때까지 계속해서 처리
* 주로 ui에 사용
* next 이벤트를 전달할 때, accept 명령어를 사용. onNext 사용 안&#x20;
* value 명령어로 저장하고 있는 next이벤트에 접근
