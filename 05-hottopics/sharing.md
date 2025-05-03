# Sharing

## 🔥 Sharing Operators — `share`, `replay`, `multicast`

> **“Hot으로 만들고, 구독자를 줄세워라.”**

Sharing 계열 오퍼레이터는 **Cold Observable을 Hot으로 변환**하거나, 이미 Hot 스트림을 구독자 간 효율적으로 공유합니다. 네트워크 중복 호출 방지, ‑UI 중복 업데이트 등에서 필수입니다.

***

### 1️⃣ share() — 가장 간단한 Multicasting

```swift
let shared = api.fetchItems()
    .share()

shared.subscribe(...)
shared.subscribe(...)
```

* `share()` = `multicast(subject: PublishSubject()).refCount()`.
* **첫 구독** 시 소스 Observable 구독, **모든 구독 해제**되면 자동 dispose.
* Default `replay:0`, `scope:.whileConnected`.

#### 파라미터

| 파라미터     | 기본값               | 설명                               |
| -------- | ----------------- | -------------------------------- |
| `replay` | 0                 | 캐시할 최신 이벤트 개수                    |
| `scope`  | `.whileConnected` | `.forever` 설정 시 첫 연결 후 계속 Hot 유지 |

```swift
.share(replay:1, scope:.forever) // Behavior-like Hot Cache
```

***

### 2️⃣ replay(\_:) — 지정 개수 Replay 후 자동 Connect

```swift
let hot = coldObservable
    .replay(1) // 최신 1개 캐시

hot.connect() // 수동 연결
```

* `publish().refCount()` 패턴과 달리 **버퍼** 지원.
* `.connect()` 수동 호출 or `.autoconnect()`.

***

### 3️⃣ multicast(subject:) — 완전 수동 제어

```swift
let subject = PublishSubject<Int>()
let multicasted = coldObservable.multicast(subject: subject)

multicasted.subscribe(...)
multicasted.connect()
```

* 원하는 Subject 타입 지정 (e.g., `ReplaySubject`, `BehaviorSubject`).
* **connect/dispose**를 외부에서 제어해야 함.

***

### 4️⃣ Hot 변환 Flowchart

```
Cold
 │  share()       (auto connect)
 ▼
Hot (PublishSubject)
 │  replay(…)     (auto connect)
 ▼
Hot + Cache
 │  multicast()   (manual connect)
 ▼
Custom Hot (Behavior/Re…)
```

***

### 5️⃣ 실전 패턴

#### A. 이미지 다운로드 캐싱

```swift
func loadImage(url: URL) -> Observable<UIImage> {
    URLSession.shared.rx.data(request: URLRequest(url: url))
        .map(UIImage.init)
        .share(replay: 1, scope: .forever)
}
```

#### B. BLE 센서 실시간 스트림 공유

```swift
bleManager.rx.sensorData()
    .share()
    .bind(to: chart, logger, fileWriter)
```

#### C. Splash 병렬 API 호출 → merge

```swift
Observable.merge(api.a().share(), api.b().share())
```

***

### 6️⃣ Pitfalls & Tips

1. `share()`의 `scope:.whileConnected`는 **모든 구독 종료** 시 재시작 → 기억하자.
2. memory: `replay` 값이 크면 캐시 메모리 증가 — 필요 요소만.
3. `Driver`/`Signal`(RxCocoa) 내부도 `share(replay:1)` 사용.

***

### 7️⃣ Mini Quiz

1. `share(replay:1, scope:.whileConnected)`과 `replay(1).refCount()` 차이?
2. Hot Observable을 다시 Cold로 만드는 방법은?
3. Subject를 외부에 노출하지 않고 multicast를 쓰는 장점?

<details>

<summary>Answesrs</summary>

1. 기능 유사하지만 `share`는 Operator 체인 내 간단 구문, `replay(...).refCount()`로 동일 구현; `share`가 concise.
2. 불가능 — Hot은 이미 연결; 필요하면 **새 Cold Observable**을 생성하거나 `.takeUntil(deallocated)`로 구독 범위를 한정.
3. 캡슐화: Subject를 private로 숨겨 외부에서 임의 `onNext` 방지하고 connect 제어만 노출.

</details>

***

> 다음 ▶️ **memoryManagement.md** 로 이동해 DisposeBag·leak 예방 전략을 심화합니다. 🚀
