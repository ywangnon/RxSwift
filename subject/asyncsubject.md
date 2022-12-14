# AsyncSubject

* Complete 이벤트가 전달될 때까지 어떤 이벤트도 전달하지 않음
* complete 이벤트가 전달되면, 가장 최신 Next 이벤트 하나 전달
* 이벤트가 없으면 complete만 전달하고 종료
* 에러 이벤트가 오면 데이터를 전달하지 않고, error 이벤트만 전달
