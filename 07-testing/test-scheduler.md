# Test Scheduler

## ⏰ TestScheduler 심화 가이드 — 가상 시간 완전 정복

> **“코드를 느리게 만들지 말고, 시간을 빠르게 돌려라.”**
>
> `TestScheduler`는 RxTest가 제공하는 **가상 시계**로, 밀리초 단위 비동기 로직도 _즉시_ 검증할 수 있게 해 줍니다. 이 문서는 API·패턴·디버깅 노하우를 한글로 자세히 설명합니다.

***

### 1️⃣ 기본 개념 다시 보기

| 개념           | 설명                                                                     |
| ------------ | ---------------------------------------------------------------------- |
| **Tick(틱)**  | TestScheduler가 정의한 **가상 시간 단위**. 보통 1틱=10ms 로 가정하지만, 실제 단위는 개발자 자유.    |
| **Clock**    | 현재 가상 시각(정수). `advanceTo(50)` → 50틱으로 바로 이동.                           |
| **Recorded** | `(time: Int, event: Event<Element>)` 구조. 테스트 결과·입력 모두 Recorded 배열로 표현. |

***

### 2️⃣ 필수 메서드 한눈에 정리

| 메서드                                   | 용도                                          | 예시                                                |
| ------------------------------------- | ------------------------------------------- | ------------------------------------------------- |
| `createHotObservable(_:)`             | 테스트 **입력** 정의(구독 전부터 이벤트 예약)                | `.next(10, "a")`                                  |
| `createColdObservable(_:)`            | 구독 시 0틱부터 카운트                               | `.next(5, 1)`                                     |
| `start(created:subscribed:disposed:)` | **편의 메서드**. Observable 만들고 200틱까지 실행해 결과 반환 | `scheduler.start { myStream }`                    |
| `advanceTo(_:)`                       | 클럭을 특정 시간으로 _점프_                            | `scheduler.advanceTo(150)`                        |
| `createObserver(Element.self)`        | **빈 Recorder**. 스트림을 수동으로 구독 후 결과 수집        | `let res = scheduler.createObserver(String.self)` |
| `scheduleAt(_:action:)`               | 특정 시점에 코드 실행                                | `scheduler.scheduleAt(50) { subject.onNext(1) }`  |

***

### 3️⃣ 단계별 커스텀 테스트 템플릿

```swift
func testCustomOperator() {
    // 1) 가상 시계 & Observer
    let scheduler = TestScheduler(initialClock: 0)
    let observer = scheduler.createObserver(Int.self)

    // 2) 입력 정의 (Cold)
    let numbers = scheduler.createColdObservable([
        .next(5, 1), .next(15, 2), .next(25, 3), .completed(35)
    ])

    // 3) 시스템 언더테스트(SUT)
    let sut = numbers.scan(0, +) // 누적 합

    // 4) 구독 스케줄링
    scheduler.scheduleAt(0) {
        sut.bind(to: observer).disposed(by: DisposeBag())
    }

    // 5) 가상 시계 실행 (0~100틱)
    scheduler.start()

    // 6) 기대값
    let expected = [
        .next(5, 1), .next(15, 3), .next(25, 6), .completed(35)
    ]

    XCTAssertEqual(observer.events, expected)
}
```

***

### 4️⃣ advanceTo vs start 차이

| 특징     | `start`                         | `advanceTo` + 수동 구독   |
| ------ | ------------------------------- | --------------------- |
| 구독 시점  | 자동 (`subscribed:` 파라미터, 기본 200) | 직접 `scheduleAt` 으로 지정 |
| 디스포즈   | 자동 (`disposed:` 파라미터, 기본 1000)  | 수동 dispose 가능         |
| 사용 난이도 | 간편(원라인)                         | 유연(복잡 시나리오)           |

> 대규모 시나리오·여러 단계 구독/해제를 테스트할 땐 **advanceTo** 방식을 권장합니다.

***

### 5️⃣ 실전: 지수 백오프 재시도 테스트

```swift
func testExponentialBackoff() {
    enum MyErr: Error { case fail }

    let scheduler = TestScheduler(initialClock: 0)
    let source = scheduler.createColdObservable([
        .error(10, MyErr.fail)
    ])

    // 1,2,4틱 지연 후 재시도 (총 4회)
    let backoff = source.retryWhen { errors in
        errors.enumerated().flatMap { attempt, error -> Observable<Int> in
            guard attempt < 3 else { return Observable.error(error) }
            let delay = Int(pow(2.0, Double(attempt)))
            return Observable<Int>.timer(.seconds(delay), scheduler: scheduler)
        }
    }

    let res = scheduler.start(created: 0, subscribed: 0, disposed: 50) { backoff }

    // 0+10=10  error
    // retry1 delay1  → 11 error(21)
    // retry2 delay2  → 23 error(33)
    // retry3 delay4  → 37 error(47)
    XCTAssertEqual(res.events.last?.time, 47)
}
```

***

### 6️⃣ 디버그 팁 🛠️

1. **클럭 로깅** : `print(scheduler.clock)` 중간 확인
2. **Recorded 배열 출력** : `observer.events.forEach(print)` 로 순서 체크
3. **불필요한 마블 중복 제거** : 반복되는 패턴은 `generateRecorded` 헬퍼 함수 사용

***

### 7️⃣ 체크리스트 ✅

| 체크             | 설명                                            |
| -------------- | --------------------------------------------- |
| Cold/Hot 유형 이해 | 구독 시점이 결과에 미치는 영향 파악                          |
| 시간 단위 일관성      | 1틱=10ms 등 프로젝트 공통 규칙                          |
| start 파라미터 수정  | `created`, `subscribed`, `disposed` 필요에 맞게 조정 |
| Equatable 구현   | 커스텀 모델 비교 정확도 확보                              |
| 단일 책임 테스트      | 하나의 연산자/경로만 검증해 실패 원인 명확화                     |

***

### 8️⃣ Mini Quiz

1. `createObserver` 로 수집한 Recorded 이벤트를 `scheduler.advanceTo(100)` 전에 조회하면 어떤 값이 있나요?
2. `start(subscribed: 50)` 로 설정하면 `.next(10, x)` Cold 이벤트는 몇 틱에 발생하나요?
3. Hot Observable 테스트에서 `scheduleAt(0)` 과 `scheduleAt(100)` 구독 시 결과 차이는?

<details>

<summary>Answers</summary>

1. advanceTo 전엔 **이벤트가 아직 발생하지 않아** 배열이 비어 있습니다.
2. 50(구독) + 10 = **60틱** 에 onNext.
3. 100틱 구독이면 **구독 이전**에 발생한 이벤트는 수신되지 않아 일부 값이 누락됩니다.

</details>

***

> 이제 TestScheduler까지 마스터했습니다! 실제 프로젝트의 비동기 코드를 자신 있게 테스트해 보세요. 🚀
