# just, of, from

## ⚡️ `just`, `of`, `from` — 가장 빠른 시퀀스 팩토리 3총사

> **“Hello, World!”도 1줄이면 스트림이다.”**

이 문서에서는 RxSwift에서 가장 자주 쓰이는 세 가지 **동기(Synchronous) 시퀀스 생성 메서드**를 정리합니다.

***

### 1️⃣ `Observable.just(_:)` — 단일 요소

```swift
Observable.just("🍎")
    .subscribe(onNext: { print($0) },
               onCompleted: { print("done") })
```

**특징**

* 하나의 값 `onNext` → 즉시 `onCompleted` → Dispose
* **Cold**: 매 구독마다 같은 값 재생산
* 메모리·스케줄러 부담 거의 0

> 유닛 테스트, 상수 모델, 디폴트 응답 Mock 등에 유용

***

### 2️⃣ `Observable.of(_:)` — 가변 인자 리스트

```swift
Observable.of(1, 2, 3)
    .toArray()
    .subscribe(onSuccess: { print($0) }) // [1,2,3]
```

**특징**

* 전달받은 인자 순서대로 `onNext`
* 요소가 **N개**인 `just` 확장판
* 배열 전달 시 오버로드 방지: `Observable.of([1,2,3])` → 요소 1개(배열) 방출

> `just([1,2,3])` vs `of(1,2,3)` 혼동 주의

***

### 3️⃣ `Observable.from(_:)` — Sequence → Observable

```swift
let names = ["A", "B", "C"]
Observable.from(names)
    .subscribe(onNext: { print($0) })
```

**특징**

* `Sequence` 프로토콜 채택 타입(Array, Set, Dictionary.keys...) 전환
* **for-in** 반복과 같은 동기 흐름 → 깔끔한 Operator 체인 결합 가능

```swift
URL(string: "...")
    .map(fetchHTML) // RxSwiftExt optional로 변환
```

***

### 4️⃣ 성능 & 스케줄러

* 모두 \*\*현재 스레드(CurrentThreadScheduler)\*\*에서 즉시 실행.
* 값 개수가 많아도 stack-safe (내부 `generate` 사용).
* 필요한 경우 `.observe(on: Main)` 등으로 스레드 전환.

***

### 5️⃣ 실전 활용 스니펫

#### A. 디폴트 Section 모델 발행

```swift
let emptySections = Observable.just([SectionModel(model: "", items: [])])
```

#### B. Enum Case 스트림

```swift
enum Page { case home, settings, about }
Observable.of(Page.home, .settings, .about)
```

#### C. JSON 파일 → 배열 → Observable

```swift
let items: [Todo] = load("todo.json")
Observable.from(items)
    .bind(to: tableView.rx.items(...))
```

***

### 6️⃣ Mini Quiz

1. `Observable.just([1,2,3]).subscribe(...)`는 몇 번 `onNext`를 호출할까?
2. `from(Set(["A","B"]))`를 구독하면 요소 순서는 보장될까?
3. `of()`과 `from([])` 둘 다 빈 Observable을 생성할 수 있을까?

<details>

<summary>Answers</summary>

1. **1번** — 배열 전체가 하나의 요소로 전달.
2. **보장되지 않는다.** Set은 순서가 없는 `Sequence`; 순서가 내부 구현에 의존.
3. 가능하다. `Observable.of()`는 인자가 없으면 즉시 `.completed`를 방출, `Observable.from([])` 역시 요소 없이 `.completed`.

</details>

***

> 다음 ▶️ **deferred, timer, interval** 로 넘어가 _지연·주기 시퀀&#xC2A4;_&#xB97C; 배워봅시다. 🚀
