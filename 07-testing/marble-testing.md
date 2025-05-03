# Marble Testing

## 🪄 Marble 테스트 완전 입문

> **“스트림을 그림으로 그려서 디버깅하고, 단위 테스트까지!”**
>
> 이 문서는 **Marble Diagram**을 처음 접하는 분도 바로 이해할 수 있도록, **기호 설명 → 코드 작성 → 테스트 결과 비교** 순으로 친절하게 안내합니다.

***

### 1️⃣ Marble 다이어그램이란?

* **Marble(구슬)** : 스트림에서 방출되는 **이벤트 한 개**를 구슬 모양으로 표현
* **타임라인(—)** : 왼쪽에서 오른쪽으로 흐르는 **가상의 시간 축**

```
--a-b--|
```

* `-` = 10 ms 같은 **시간 간격 한 칸** (단위는 테스트에서 자유롭게 정의)
* `a`,`b` = onNext 이벤트 값 (알파벳 / 숫자 / 이모지 무엇이든 가능)
* `|` = **onCompleted** (스트림 정상 종료)
* `#` 또는 `x` = **onError** (에러로 종료)
* `(ab)` = 같은 시간칸에 동시에 두 이벤트 발생

> 💡 실제 앱에서는 밀리초보다 훨씬 짧은 **가상 시간**을 사용하기 때문에, 테스트가 매우 빠르게 실행됩니다.

***

### 2️⃣ RxTest 기본 설정

```swift
import XCTest
import RxSwift
import RxTest
import RxBlocking // 필요 시 동기 검증용

final class MarbleExampleTests: XCTestCase {
    var scheduler: TestScheduler!
    var disposeBag: DisposeBag!

    override func setUp() {
        super.setUp()
        // 0 을 시작 시각으로 하는 가상 시계 생성
        scheduler = TestScheduler(initialClock: 0)
        disposeBag = DisposeBag()
    }
}
```

* **TestScheduler** : 가상 시간 컨트롤러 (tick 단위: 10 by default)
* **DisposeBag** : 테스트마다 새로 생성해 메모리 누수 방지

***

### 3️⃣ Recorded 이벤트 타입 이해하기

RxTest는 **타임스탬프 + Event**를 `Recorded` 구조체로 표현합니다.

```swift
Recorded.next(20, "a")   // 20틱 시점, onNext("a")
Recorded.completed(50)     // 50틱 시점, onCompleted
Recorded.error(30, MyErr)  // 30틱 시점, onError
```

#### `.next` 의 의미

* 구독자가 `onNext(value)` 콜백을 받은 **정확한 시점**과 **값**을 기록합니다.
* 이벤트가 객체라면, 동일성 비교를 위해 **Equatable** 채택 필요.

***

### 4️⃣ Cold vs Hot Observable 예제

```swift
let cold = scheduler.createColdObservable([
    .next(10, "A"), .next(20, "B"), .completed(30)
])

let hot = scheduler.createHotObservable([
    .next(5, "X"),  .next(15, "Y"), .completed(40)
])
```

* **Cold** : 구독한 순간부터 0틱으로 카운트 (테스트에 추천)
* **Hot** : 테스트 시계 0부터 이벤트가 예약 (구독이 늦으면 일부 값 손실)

***

### 5️⃣ 실제 테스트 작성 – debounce 연산자 검증 (수정 버전)

목표: `debounce(15)` 가 **마지막 입력만** 전달하는지 확인합니다.\
⚠️ _중간에 새 입력이 들어오면 이전 타이머가 **리셋**된다는 &#xC810;_&#xC774; 핵심입니다.

```swift
func testDebounce() {
    // 1) 입력 스트림 정의 (Hot)
    // 타임라인: 10 ─a─ 20 ─b──────── 40 ─c──────── 60(|)
    let input = scheduler.createHotObservable([
        .next(10, "a"),  // a
        .next(20, "b"),  // b (a 타이머 리셋)
        .next(40, "c"),  // c
        .completed(60)   // 완료
    ])

    // 2) Operator 적용
    let output = scheduler.start {                          // 기본 0~200틱 실행
        input.debounce(.seconds(15), scheduler: scheduler)  // 15틱 = 150 ms 가정
    }

    // 3) 기대 결과
    // b : 20 + 15 = 35
    // c : 40 + 15 = 55 (c 이후 15틱 동안 추가 입력 없음)
    let expected = [
        Recorded.next(35, "b"),
        Recorded.next(55, "c"),
        Recorded.completed(60)
    ]

    XCTAssertEqual(output.events, expected)
}
```

***

### 6️⃣ Marble 문자열 헬퍼로 간단하게 (선택) 헬퍼로 간단하게 (선택)

```swift
let cold = scheduler.createColdObservable(marbles: "--a-b--|", values: ["a":1, "b":2])
```

* 서드파티 `RxMarbles` 또는 `RxExpect` 등을 사용하면 문자열만으로 테스트 정의 가능

***

### 7️⃣ 오류 및 재시도 테스트 예시 — 타임라인 상세 설명

`retry(2)` 는 **최대 2회 추가 재시도(총 3회 실행)** 를 의미합니다. `TestScheduler.start` 는 기본적으로 **200틱** 시점에 Cold Observable 을 구독합니다. 따라서 내부 이벤트 시각 `+200` 으로 계산됩니다.

```
시각  |  200 210 220 230 240 250 260 270
      |   ↘︎ 1  X  ↘︎ 1  X  ↘︎ 1  X
회차  |   1st     2nd     3rd
```

* 각 회차: 값(10틱) → 에러(20틱)
* 3회차(마지막) 에러 발생 후 `retry` 종료

```swift
func testRetryLogic() {
    enum TestErr: Error { case fail }

    // Cold Observable: 10틱 후 값, 20틱 후 에러
    let failing = scheduler.createColdObservable([
        .next(10, 1),
        .error(20, TestErr.fail)
    ])

    let output = scheduler.start { failing.retry(2) } // 총 3회 실행

    let expected = [
        // 1회차 (200 + 10/20)
        .next(210, 1), .error(220, TestErr.fail),
        // 2회차 (220 dispose + 즉시 재구독 → 200+30=230,200+40=240)
        .next(230, 1), .error(240, TestErr.fail),
        // 3회차 (240 dispose 후 재구독)
        .next(250, 1), .error(260, TestErr.fail)
    ]

    XCTAssertEqual(output.events, expected)
}
```

> **TIP:** 더 긴 스트림이나 지수 백오프를 테스트할 땐 `retryWhen` 과 `TestScheduler.advanceTo()`를 조합하세요.

***

### 8️⃣ 베스트 프랙티스 체크리스트 ✅

| 체크포인트                           | 이유 / 효과                                   |
| ------------------------------- | ----------------------------------------- |
| **Cold Observable 우선 사용**       | Hot 스트림은 구독 시점에 따라 결과 달라져 재현성 떨어짐         |
| **테스트마다 `** · **` 재생성**         | 테스트 간 상태 공유로 인한 간섭 방지                     |
| **틱 계산을 종이에 먼저 작성**             | 시간 기반 오퍼레이터(`debounce`, `throttle`) 실수 감소 |
| \`\`\*\* 파라미터 조정\*\*            | 200틱 이후 이벤트도 검증 가능                        |
| **커스텀 모델 Equatable 구현**         | `XCTAssertEqual` 비교 오류 방지                 |
| **Hot 스트림 구독 시점 명시 (**\`\`**)** | 이벤트 손실 여부 명확화                             |
| **실제 로직과 동일 Scheduler 사용**      | `observe(on:)` 누락으로 인한 스레드 차이 방지          |

***

### 9️⃣ Mini Quiz

1. 마블 `--(ab)-|` 는 어떤 순서로 이벤트가 전달되나요?
2. `TestScheduler.start` 의 기본 **테스트 실행 종료 시각**은 몇 틱일까요?
3. Hot 스트림 테스트 시, 구독을 `scheduler.scheduleAt(5)` 대신 `scheduler.scheduleAt(25)` 에 하면 어떤 차이가 있나요?

<details>

<summary>Answers</summary>

1. 20틱 후 동시에 `a`, `b` 이벤트(onNext 순서는 배열 정의 순), 그 다음 10틱 후 `onCompleted`.
2. 기본값 200틱 (필요 시 `disposed:` 매개변수로 수정 가능).
3. 25틱에 구독하면 5·15틱 이벤트(`X`,`Y`)가 이미 발행돼 **수신되지 않음**.

</details>

***

> 다음 문서 ▶️ **test\_scheduler.md** 에서 TestScheduler API 를 더욱 깊이 다뤄봅시다. 🚀
