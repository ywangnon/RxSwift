---
description: RxSwift가 해결하려는 문제·철학
---

# Overview

## 🌀 Overview — RxSwift란 무엇인가?

> **“데이터 흐름과 변화를 코드 한 줄로 표현한다.”**  — Reactive Manifesto

RxSwift는 **ReactiveX**(Reactive Extensions) 패러다임을 Swift 언어로 옮겨온 라이브러리입니다. 전통적인 _callback_·_delegate_ 기반 iOS 개발에서 발생하는 **비동기 지옥**(Callback Hell)과 상태 관리의 복잡성을, "**데이터 스트림**과 **오퍼레이터**"라는 단순한 개념으로 해결합니다.

***

### 1️⃣ 우리가 겪는 문제들

| Pain Point | 전통적 해결책                      | 한계                    |
| ---------- | ---------------------------- | --------------------- |
| **중첩된 콜백** | Completion Handler, Delegate | 가독성 ↓, 테스트 어려움        |
| **상태 폭발**  | KVO, NotificationCenter      | 의존성 분산, 메모리 관리 부담     |
| **쓰레드 전환** | GCD (`DispatchQueue`)        | 호출 위치가 분산돼 흐름 파악 어려움  |
| **UI 바인딩** | Target‑Action, DataSource    | 코드 중복, 비동기 이벤트 통합 어려움 |

RxSwift는 이 모든 문제를 **Observable → Operator → Observer** 파이프라인으로 추상화하여, _선언형(Declarative)_&#xC2A4;타일로 표현하도록 돕습니다.

***

### 2️⃣ 핵심 개념 한 눈에 보기

| Building Block | 요약                 | 예시 코드                                 |
| -------------- | ------------------ | ------------------------------------- |
| **Observable** | 시간에 따라 변하는 데이터 스트림 | `Observable<Int>.from([1,2,3])`       |
| **Observer**   | 스트림을 구독해 값·이벤트를 처리 | `subscribe(onNext:)`                  |
| **Operator**   | 스트림을 변환/결합/필터링     | `map`, `filter`, `combineLatest`      |
| **Scheduler**  | 실행 컨텍스트(쓰레드) 지정    | `observe(on: MainScheduler.instance)` |
| **DisposeBag** | 구독 해제 메모리 관리       | `disposed(by: bag)`                   |

> 각 개념은 뒤 챕터에서 실습 중심으로 다룹니다. 💡

***

### 3️⃣ RxSwift를 사용하면 좋은 때

* **입력 이벤트가 많은 UI**: 검색창 자동완성, 채팅, 음악 플레이어
* **복수 API 호출 결과 결합**: 로그인 후 프로필·권한 동시 요청
* **실시간 데이터 스트림**: BLE 센서 값, WebSocket, 주가 차트
* **폼 유효성 검사**: 각 필드의 상태를 `combineLatest`로 묶어 버튼 활성화

❗ _단순 시나리&#xC624;_&#xC5D0;서는 오히려 과도한 추상화가 될 수 있으므로, “스트림으로 사고해야 이득이 큰 곳”에 집중하세요.

***

### 4️⃣ 학습 로드맵

1. **Core Concepts** ▶️ Observable, Observer, Subject
2. **Creating Sequences** ▶️ `just`, `from`, `create`
3. **Operators 기초** ▶️ Filtering · Transforming · Combining
4. **Time‑based & Error Handling**
5. **Schedulers & Testing**
6. **RxCocoa로 UIKit/SwiftUI 바인딩**
7. **Hot Topics** ▶️ Sharing, Memory Management, Architecture 패턴

> 👉 각 단계별 실습 Playground와 Mini‑Quiz가 준비되어 있습니다.

***

### 5️⃣ 한 줄 설치 (SPM)

```swift
.package(url: "https://github.com/ReactiveX/RxSwift.git", from: "6.6.0")
```

> 자세한 설치 가이드는 **Setup** 페이지에서 이어집니다.

***

### 6️⃣ 참고 자료

* 공식 사이트: [http://reactivex.io/](http://reactivex.io/)
* RxSwift GitHub: [https://github.com/ReactiveX/RxSwift](https://github.com/ReactiveX/RxSwift)
* RxMarbles(오퍼레이터 시각화): [http://rxmarbles.com/](http://rxmarbles.com/)

***

> 앞으로 _10분 실습 → Mini Quiz_ 사이클을 반복하면서, **선언형·비동기 사고방식**을 몸에 익혀봅시다! 🚀
