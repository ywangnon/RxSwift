# deferred, timer, interval

## ⏲️ `deferred`, `timer`, `interval` — 시간 기반 시퀀스 맛보기

> **“스트림에도 시계가 필요하다!”** RxSwift는 지연·주기·지속적인 이벤트를 손쉽게 작성할 수 있는 팩토리 메서드를 제공합니다.

***

### 1️⃣ `Observable.deferred` — 구독 시점마다 새 시퀀스

```swift
var flip = false
let deferred = Observable<String>.deferred {
    flip.toggle()
    return flip ? .just("🔴") : .just("🔵")
}

deferred.subscribe(onNext: print) // 🔴
deferred.subscribe(onNext: print) // 🔵
```

* **Cold**: 각 구독 시 클로저 재평가.
* 의존성이 있는 초기화(토큰 갱신, 위치 권한 요청) 패턴에 활용.

***

### 2️⃣ `Observable.timer` — 지연 후 1회 또는 주기 방출

```swift
// 3초 뒤 1회 0 방출
Observable<Int>.timer(.seconds(3), scheduler: MainScheduler.instance)
    .subscribe(onNext: { print($0) })

// 1초 뒤 시작, 그후 5초마다 증가 카운트 방출
Observable<Int>.timer(.seconds(1), period: .seconds(5), scheduler: MainScheduler.instance)
```

* **period 파라미터**: 생략 시 단발성, 지정 시 `interval`과 유사.
* 앱 알림·Splash 스킵 버튼 타이머 등에 사용.

***

### 3️⃣ `Observable.interval` — 고정 주기 스트림

```swift
Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
    .map { "Tick \($0)" }
    .subscribe(onNext: print)
```

* 0,1,2,3… 카운트 업데이트.
* 주가 차트, 운동 타이머, 실시간 센서 등 반복 작업.

***

### 4️⃣ Scheduler & Threading

| Scheduler 선택                       | 효과                               |
| ---------------------------------- | -------------------------------- |
| `MainScheduler`                    | UI 타이머, 애니메이션 진행                 |
| `ConcurrentDispatchQueueScheduler` | 백그라운드 폴링 작업                      |
| `TestScheduler`                    | 가상 시간 테스트 (`advanceTo`, `start`) |

> iOS 백그라운드 제한에서 긴 `interval`은 **`WKBackgroundTask`** 대안 고려.

***

### 5️⃣ 조합 예시 — 10초 카운트다운

```swift
func countdown(from n: Int) -> Observable<Int> {
    Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
        .map { n - $0 }
        .take(n + 1) // 0까지 포함
}

countdown(from: 10)
    .subscribe(onNext: { print($0) },
               onCompleted: { print("Go!") })
```

***

### 6️⃣ Memory & Cancellation

* `DisposeBag` 또는 `.takeUntil(trigger)`로 예약 작업 취소.
* `timer/interval`은 내부 GCD Timer — **retain 주의** (`disposed(by:)`).

```swift
let task = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
    .subscribe(...)

DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
    task.dispose() // 타이머 취소
}
```

***

### 7️⃣ Mini Quiz

1. `timer`와 `interval`의 주기 방출 차이?
2. `deferred`를 활용해 사용자 로그인 상태에 따라 다른 API 스트림을 제공하려면? (개념 설명)
3. `interval` 스트림을 일시 중지했다가 다시 시작하려면 어떤 오퍼레이터 조합이 유용할까?

<details>

<summary>Answers</summary>

1. `timer`는 **지연 후 1회**(period 없을 때) 또는 **지연 후 주기**(period 사용) / `interval`은 **즉시 or 0 지연 후 주기** — 실행 방식 유사하나 `timer`는 첫 딜레이 지정 가능.
2. `Observable.deferred { isLoggedIn ? api.userInfo() : Observable.error(AuthError.noLogin) }` 처럼 **구독 시점**에 상태를 체크해 알맞은 시퀀스를 반환.
3. `interval` → `.pausable(trigger)`(RxExt) 또는 `withLatestFrom(pause)` + `flatMapLatest`를 사용해 Trigger가 false일 땐 판정 스킵.

</details>

***

> 🎉 CreatingSequences 챕터 완료! 이제 Operators(Filtering) 파트로 이동해 스트림 가공을 시작해 봅시다. 🚀
