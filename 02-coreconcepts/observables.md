# Observables

## 🌊 Observables — RxSwift의 핵심 스트림

> **“Everything is a sequence over time.”** — Rx 디자인 철학

이 문서에서는 `Observable`이 무엇인지, 어떻게 생성하고(Factory), 구독하며(Subscribe), 관리(Dispose)하는지 단계별로 살펴봅니다.

***

### 1️⃣ Observable 생명주기

`Observable<Element>` 는 **이벤트(Event) 스트림**입니다. 각 스트림은 최대 네 종류의 시그널을 순서대로 방출합니다.

| 이벤트           | 설명      | Swift 콜백         | 중단 여부    |
| ------------- | ------- | ---------------- | -------- |
| **next**      | 요소 값 전달 | `onNext(T)`      | ❌        |
| **error**     | 오류 발생   | `onError(Error)` | ⛔ 스트림 종료 |
| **completed** | 정상 종료   | `onCompleted()`  | ⛔ 스트림 종료 |
| **disposed**  | 리소스 해제  | `onDisposed()`   | —        |

> `next`는 여러 번 발생할 수 있지만, `error` 또는 `completed`는 단 한 번만 발생하며 둘 중 하나만 방출됩니다.

***

### 2️⃣ Observables 만들기 (Factory Methods)

| 메서드            | 설명               | 예시 코드                                                                      |
| -------------- | ---------------- | -------------------------------------------------------------------------- |
| `just(_:)`     | 하나의 요소만 방출 후 완료  | `Observable.just("🍎")`                                                    |
| `of(_:)`       | 여러 요소 순차 방출      | `Observable.of(1,2,3)`                                                     |
| `from(_:)`     | 배열·시퀀스 → 스트림     | `Observable.from(["A","B"])`                                               |
| `create(_:)`   | 커스텀 이벤트 정의       | `Observable<String>.create { observer in ... }`                            |
| `deferred(_:)` | 구독 시점마다 새 스트림 생성 | `Observable.deferred { Bool.random() ? .just("✅") : .error(err) }`         |
| `interval`     | 일정 주기 숫자 방출      | `Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)` |
| `timer`        | 지연 후 1회 또는 주기 방출 | `Observable<Int>.timer(.seconds(3), period: .seconds(1), scheduler: Main)` |

```swift
// 예: 0.5초마다 카운트 업 스트림
let counter = Observable<Int>
    .interval(.milliseconds(500), scheduler: MainScheduler.instance)
```

💡 _Tip_: 플레이그라운드에서 `PlaygroundPage.current.needsIndefiniteExecution = true` 설정 후 실행해 보세요.

***

### 3️⃣ Cold vs Hot Observables

| 구분       | 특징                   | 예시                                                             |
| -------- | -------------------- | -------------------------------------------------------------- |
| **Cold** | 구독마다 **새 데이터** 생산    | `Observable<Int>.range(0, 5)`, `URLSession.rx.response`        |
| **Hot**  | 이미 흐르고 있는 **공유 데이터** | `PublishSubject`, `NotificationCenter.default.rx.notification` |

```swift
// Cold 예시
let range = Observable.range(start: 0, count: 3)
range.subscribe(onNext: { print("🅰️", $0) })
range.subscribe(onNext: { print("🅱️", $0) }) // 두 구독자 모두 0,1,2 받음

// Hot 예시
let subject = PublishSubject<Int>()
subject.subscribe(onNext: { print("1st ->", $0) })
subject.onNext(1)
subject.subscribe(onNext: { print("2nd ->", $0) }) // 두 번째는 2부터 수신
subject.onNext(2)
```

***

### 4️⃣ 구독 & DisposeBag

```swift
let bag = DisposeBag()

Observable.from(["🍎","🍌","🍇"])
    .subscribe(
        onNext: { print("fruit:",$0) },
        onError: { print("error:",$0) },
        onCompleted: { print("completed") },
        onDisposed: { print("disposed") }
    )
    .disposed(by: bag)
```

* **DisposeBag**: 구독(disposable)을 담아 **스코프 단위**로 메모리를 해제합니다. 클래스별로 `let bag = DisposeBag()`를 보유하는 패턴이 일반적입니다.

> `rx.deallocated` 스트림을 활용하면 뷰 소멸 시 자동 해제 패턴을 구성할 수 있습니다.

***

### 5️⃣ 실전 예제 — URLSession + JSON 디코딩

```swift
struct Post: Decodable { let id: Int; let title: String }

func fetchPosts() -> Observable<[Post]> {
    let url = URL(string: "https://jsonplaceholder.typicode.com/posts")!

    return URLSession.shared.rx.data(request: URLRequest(url: url))
        .map { data in try JSONDecoder().decode([Post].self, from: data) }
}

fetchPosts()
    .observe(on: MainScheduler.instance)
    .subscribe(onNext: { print($0.first?.title ?? "-") })
    .disposed(by: bag)
```

***

### 6️⃣ Cheat Sheet

| 카테고리      | 메서드                                                                                                 | 한 줄 설명 |
| --------- | --------------------------------------------------------------------------------------------------- | ------ |
| Factory   | `just`, `of`, `from`, `create`, `deferred`, `empty`, `never`, `error`, `interval`, `timer`, `range` |        |
| Filtering | `filter`, `distinctUntilChanged`, `take`, `skip`, `debounce`, `throttle`                            |        |
| Transform | `map`, `flatMap`, `flatMapLatest`, `buffer`, `scan`                                                 |        |
| Combining | `merge`, `concat`, `zip`, `combineLatest`, `withLatestFrom`                                         |        |
| Error     | `catchError`, `retry`, `retryWhen`                                                                  |        |

***

### 7️⃣ Mini Quiz

1. `flatMapLatest`와 `switchMap`(Kotlin Flow) 차이를 설명해 보세요.
2. Cold Observable에서 두 번째 구독이 이전 값의 재생산을 막으려면 어떤 오퍼레이터를 사용할 수 있을까요?
3. `DisposeBag` 없이도 메모리 누수가 발생하지 않는 경우는?

<details>

<summary><strong>정답</strong></summary>

1. **`flatMapLatest` vs `switchMap`**
   * RxSwift의 `flatMapLatest`(a.k.a _switchMap_)는 **가장 최근에 생성된 내부 Observable**만 구독하고, 이전 스트림은 구독을 해제합니다. Kotlin Flow·RxJS의 `switchMap`과 동작이 동일하며 단순히 네이밍 차이입니다.
2. **Cold Observable 재생산 방지**
   * `share()` 또는 `share(replay:scope:)`, `publish().refCount()` 등 _공유 오퍼레이&#xD130;_&#xB85C; 스트림을 **Hot**하게 변환해 두 번째 구독 시 재실행을 막을 수 있습니다.
3. **DisposeBag 없이 메모리 누수가 없는 경우**
   * `Observable.just`, `of`, `from` 같이 **동기적으로 즉시 완결**(completed)되는 시퀀스는 구독 직후 종료되므로 명시적 dispose가 필요 없습니다. 또한 `take(1)`처럼 일찍 완료시키는 오퍼레이터를 사용했을 때도 메모리 누수 위험이 낮습니다.

</details>
