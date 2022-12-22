---
description: subject 개요
---

# 개요

**Subject**

* 다른 옵저버블에게 이벤트를 받아서 다른 구독자에게 전달 가능
* 옵저버블인 동시에 옵저버

**종류**

* PublishSubject: 서브젝트로 전달되는 이벤트를 구독자에게 전달
* BehaviorSubject: 생성시점에 시작이벤트를 지정. 가장 최신 이벤트를 저장해두었다가 새로운 구독자에게 전달
* ReplaySubject: 하나이상의 이벤트를 버퍼에 저장. 옵저버가 구독을 시작하면 버퍼의 이벤트를 전부 전달
* AsyncSubject: 서브젝트로 컴플리티드 이벤트가 전달되는 시점에 마지막으로 전달된 넥스트 이벤트를 구독자에게 전달

**Relays**

Complete와 Error 없이, Next 이벤트만 받는다. 종료없이 계속 전달되는 시퀀스 처리시 사용

* PublishRelay: PublishSubject 래핑
* BehaviorRelay: BehaviorSubject 래핑
