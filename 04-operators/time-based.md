# Time‑Based

## ⏱️ Time‑Based Operators — `debounce`, `throttle`, `timeout` 외

> **“이벤트를 시간 축으로 다듬자.”**

시간 기반(Time‑Based) 오퍼레이터는 Observable의 **발행 간격**과 **지연**을 제어합니다. UI 입력 제어, 요청 폭주 방지, 네트워크 타임아웃 등에 필수적입니다.

***

### 1️⃣ 핵심 오퍼레이터 비교표

| Operator   | 동작                                  | 주요 파라미터                     | 대표 사용 사례                 |
| ---------- | ----------------------------------- | --------------------------- | ------------------------ |
| `debounce` | 지정 시간 동안 **새 이벤트 없을 때** 마지막 값 방출    | `dueTime, scheduler`        | 검색창 자동완성, 텍스트 입력         |
| `throttle` | **첫/마지막 이벤트**만 통과 (옵션) — 기간 내 중복 억제 | `dueTime, latest`           | 버튼 연타 방지, API Rate‑limit |
| `timeout`  | 지정 시간 내 **onNext 없으면 에러**           | `dueTime, scheduler, other` | 서버 응답 대기 제한              |
| `delay`    | 각 이벤트를 지정 시간 **지연** 후 방출            | `dueTime, scheduler`        | 애니메이션 시퀀스, 알림 지연         |
| `sample`   | 트리거 스트림/주기마다 **최신 값을 샘플**           | `sampler`                   | 주식 차트 Snapshot           |
| `buffer`   | 일정 기간/개수로 **배치 묶기**                 | `timeSpan, count`           | 센서 Batch Upload          |

***

### 2️⃣ 실전 스니펫

#### A. `debounce` — 검색어 입력

```swift
searchBar.rx.text.orEmpty
    .debounce(.milliseconds(300), scheduler: MainScheduler.instance)
    .distinctUntilChanged()
    .flatMapLatest(api.search)
    .bind(to: results)
```

> 300 ms 내 추가 입력이 없을 때만 호출 → API 트래픽 절감

#### B. `throttle` — 좋아요 버튼 중복 방지

```swift
likeButton.rx.tap
    .throttle(.seconds(1), scheduler: MainScheduler.instance)
    .subscribe(onNext: viewModel.like)
```

> 1 초 안에 여러 탭 중 첫 탭만 처리 (latest: false)

#### C. `timeout` — 서버 응답 실패 처리

```swift
api.fetchData()
    .timeout(.seconds(5), scheduler: MainScheduler.instance)
    .catchError { _ in Observable.just(.timeoutPlaceholder) }
    .bind(to: viewModel.state)
```

#### D. `buffer` — 센서 10개 이벤트 묶어 업로드

```swift
sensorStream
    .buffer(timeSpan: .seconds(2), count: 10, scheduler: SerialDispatchQueueScheduler(qos: .utility))
    .filter { !$0.isEmpty }
    .flatMap(uploadBatch)
    .subscribe()
```

***

### 3️⃣ debounce vs throttle 시각 비교

| 이벤트 스트림                            | `──x─x──x─x────x──`        |
| ---------------------------------- | -------------------------- |
| **debounce(300 ms)**               | `─────x───────x─` (마지막 값)  |
| **throttle(300 ms, latest:false)** | `──x─────x─────x` (첫 값)    |
| **throttle(300 ms, latest:true)**  | `──x───────x────x` (첫+마지막) |

> `latest` 플래그가 true면 기간 끝에 **추가**로 최신 값 전달.

***

### 4️⃣ Scheduler & Testing Tips

* 대부분의 시간 연산은 **Scheduler 의존** → 필요 시 `TestScheduler`로 가상 시간 테스트 (`advanceTo`).
* UI 작업은 `MainScheduler`, 백그라운드 Polling은 `ConcurrentDispatchQueueScheduler`.

***

### 5️⃣ Edge Cases & pitfalls

1. **debounce 0 ms** 는 의미 없고, 플랫폼마다 최소 해상도(주로 1 ms).
2. `timeout` 뒤 `retry` 연속 사용 시 **Cartesian 폭발** 조심 — 지수 Back‑off 필요.
3. `buffer`에 큰 `count` + 긴 `timeSpan` 설정하면 메모리 버퍼 증가 위험.

***

### 6️⃣ Mini Quiz

1. `throttle`의 `latest` 옵션을 true로 하면 어떤 값이 추가로 방출되는가?
2. `debounce`와 `sample`의 차이를 간단히?
3. `timeout` 후 _다른 Observabl&#x65;_&#xB85C; 대체하려면 무엇을 사용?

<details>

<summary>Answers</summary>

1. 기간 종료 시점의 **마지막 이벤트**를 추가로 전달한다.
2. `debounce`는 _입력 스트림 자체에 지&#xC5F0;_&#xC744; 주어 **마지막 값**만, `sample`은 _별도 트리&#xAC70;_&#xC5D0;서 최신 값을 **샘플링**.
3. `timeout`의 `other` 파라미터에 대체 Observable 지정 or `.catchError { _ in other }` 연산.

</details>

***

> 이어서 ▶️ **errorHandling.md** 로 이동해 `catchError`, `retry`로 안정적인 스트림 만들기를 배워봅시다. 🚀
