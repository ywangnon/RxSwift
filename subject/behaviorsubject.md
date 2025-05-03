# BehaviorSubject

## 🪄 BehaviorSubject — 최신 값 보존 Subject

> **핵심 키워드:** _Last Emitted Value_ · _Initial Seed_ · Hot Stream

`BehaviorSubject<Element>`는 **항상 최신 값을 1개 저장**하고, 새 구독자에게 즉시 전달합니다. **초기값**(seed)을 요구하므로 상태를 초기화해야 하는 UX(토글, 폼 입력 등)에 적합합니다.

***

### 1️⃣ 특징 요약

| 항목      | 내용                                          |
| ------- | ------------------------------------------- |
| 초기값     | ✔️ 필수 (`init(value:)`)                      |
| 캐시 크기   | 1 (최신 값)                                    |
| 구독 시 전달 | 최신 값 즉시 발행                                  |
| 스트림 종료  | `.onCompleted()` 또는 `.onError()` 전파 후 캐시 제거 |

***

### 2️⃣ 기본 사용법

```swift
let subject = BehaviorSubject(value: "🚦 Red")
let bag = DisposeBag()

// 첫 구독자 → 초기값 전달
subject
    .subscribe(onNext: { print("1️⃣",
$0) })
    .disposed(by: bag)

subject.onNext("🟡 Yellow")

// 두 번째 구독자 → 최신값 Yellow 전달
subject
    .subscribe(onNext: { print("2️⃣",
$0) })
    .disposed(by: bag)

subject.onNext("🟢 Green")
```

**출력**

```
1️⃣ 🚦 Red
1️⃣ 🟡 Yellow
2️⃣ 🟡 Yellow
1️⃣ 🟢 Green
2️⃣ 🟢 Green
```

***

### 3️⃣ ViewModel 상태 보존 예제 — 로그인 버튼 활성화

```swift
struct Input {
    let email: Observable<String>
    let password: Observable<String>
}

struct Output {
    let isLoginEnabled: Observable<Bool>
}

func transform(input: Input) -> Output {
    let isValid = Observable
        .combineLatest(input.email, input.password)
        .map { email, pw in email.contains("@") && pw.count >= 6 }

    let enabledSubject = BehaviorSubject(value: false)

    isValid
        .bind(to: enabledSubject)
        .disposed(by: bag)

    return Output(isLoginEnabled: enabledSubject.asObservable())
}
```

* `BehaviorSubject`가 최신 유효성 결과를 캐싱 → 화면 재진입 시 즉시 반영.

***

### 4️⃣ vs PublishSubject 비교

| 항목      | PublishSubject | **BehaviorSubject** |
| ------- | -------------- | ------------------- |
| 초기값     | 없음             | **필수**              |
| 캐시      | 0              | **1**               |
| 구독 직후 값 | ❌              | **최신 1개**           |
| 사용 사례   | 단순 이벤트 브로드캐스트  | UI 상태, 설정 값, 데이터 캐시 |

***

### 5️⃣ 값 가져오기 & 에러 처리

```swift
// 현재 값 읽기
if let current = try? subject.value() {
    print("Current ->", current)
}

// 에러 발생 시 value() 호출은 throw
subject.onError(MyError.invalid)

// try? subject.value() -> nil, 구독 시 .error 즉시 전달
```

> `value()`는 Subject가 에러/완료 상태면 throw 하므로, 옵셔널 `try?` 패턴이 안전합니다.

***

### 6️⃣ 메모리 관리 & 주의

1. **Strong reference**: BehaviorSubject는 마지막 값 보유 → 크기가 큰 모델/이미지 저장 주의.
2. **State explosion**: 너무 많은 BehaviorSubject → UI 상태 분산, _Relay_ 또는 `combineLatest`로 구조화.
3. **Thread safety**: 다중 쓰레드 `onNext` 시 Data Race 가능 → Serial Scheduler, Actor 활용.

***

### 7️⃣ Mini Quiz

1. `BehaviorSubject`를 `.onCompleted()` 후 다시 `onNext` 호출하면?
2. 캐시된 최신 값이 크기가 큰 이미지라면 메모리 대응 방법 두 가지는?
3. Combine 프레임워크에서 비슷한 컨셉은 무엇인가?

<details>

<summary>정답</summary>

1. **불가** — 스트림은 Completed 상태로, 이후 이벤트는 무시됩니다. 새 Subject를 생성해야 합니다.
2. **대응 방법**
   1. 이미지의 **URL 또는 식별자**만 보관하고 실제 데이터는 캐시 레이어로 분리.
   2. `BehaviorRelay`로 교체 후 값 타입을 `WeakWrapper`(참조 타입)로 래핑
3. Combine의 \*\*`CurrentValueSubject`\*\*가 BehaviorSubject와 동일하게 최신 값 1개를 저장 후 새 구독자에게 전달합니다.

</details>

***

> 다음 문서 ▶️ **ReplaySubject** 로 넘어가 다중 값 캐시 전략을 살펴봅니다. 🚀
