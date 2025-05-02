# Transforming

## 🔄 Transforming Operators — 스트림을 ‘변형’하는 마법

> **“데이터를 원하는 모습으로 가공하라.”**

Transforming 계열 오퍼레이터는 Observable의 **값 구조·타입·개수를 변경**하여 다음 단계에서 활용하기 쉽게 만듭니다. 특히 `map`·`flatMap`·`scan`은 Rx의 삼대장이라 불릴 만큼 자주 사용됩니다.

***

### 1️⃣ 핵심 오퍼레이터 요약

| 오퍼레이터                       | 설명                              | 대표 시그니처                                 |
| --------------------------- | ------------------------------- | --------------------------------------- |
| `map`                       | 요소를 1:1 변환                      | `(Element) -> R`                        |
| `compactMap`                | nil 필터링 & 변환                    | `(Element) -> R?`                       |
| `flatMap`                   | 요소를 Observable로 변환 후 **병합**     | `(Element) -> Observable<R>`            |
| `flatMapLatest`             | 가장 최신 내부 Observable만 구독(switch) | same                                    |
| `flatMapFirst`              | 첫 내부 스트림 지속, 이후 무시              | same                                    |
| `concatMap`                 | 이전 내부 Observable 완료 후 다음 실행     | same                                    |
| `scan`                      | 누적(accumulate) reduce           | `(Accumulator, Element) -> Accumulator` |
| `buffer`                    | 주기·개수 단위 배열로 묶기                 | `timeSpan,count,scheduler`              |
| `materialize/dematerialize` | Event ↔︎ Element 변환             | -                                       |

***

### 2️⃣ 실전 예제

#### A. `map` — JSON → Model 파싱

```swift
urlSession.rx.data(request: req)
    .map { try JSONDecoder().decode(User.self, from: $0) }
    .subscribe(onNext: showUser)
```

#### B. `flatMapLatest` — 텍스트 검색 자동완성

```swift
searchBar.rx.text.orEmpty
    .debounce(.milliseconds(300), scheduler: MainScheduler.instance)
    .flatMapLatest(api.search) // 새로운 키워드 입력 시 이전 요청 취소
    .bind(to: tableView.rx.items(...))
```

#### C. `scan` — 카운터 누적

```swift
button.rx.tap
    .map { _ in 1 }
    .scan(0, accumulator: +)
    .subscribe(onNext: counterLabel.rx.text)
```

#### D. `buffer` — 3개씩 배치 처리

```swift
sensorStream
    .buffer(timeSpan: .seconds(1), count: 3, scheduler: MainScheduler.instance)
    .filter { !$0.isEmpty }
    .subscribe(onNext: uploadBatch)
```

***

### 3️⃣ flatMap 패밀리 선택 가이드

| 상황                | 권장 오퍼레이터            | 이유             |
| ----------------- | ------------------- | -------------- |
| 네트워크 요청, 이전 결과 무시 | **`flatMapLatest`** | 최신 요청만 유지 (취소) |
| 첫 요청 고정, 중복 무시    | `flatMapFirst`      | 로그인 버튼 더블 탭 방지 |
| 순서 보장, 직렬 실행      | `concatMap`         | 파일 업로드 차례대로    |
| 모든 내부 스트림 병합      | `flatMap`           | 이미지 프리로드 동시 실행 |

***

### 4️⃣ 성능 & 에러 전략

* `flatMap`에서 내부 Observable이 **무한 스트림**이면 dispose 시점 관리 필요.
* `scan` 초기값 설정이 중요 — 불변 객체라면 메모리 누수 위험 낮음.
* `buffer`로 큰 배열 생성 시 **메모리 피크** 가능, `window`(Observable) 대안.

***

### 5️⃣ Transform + Scheduler 예시

```swift
images
    .flatMap { UIImageJPEGRepresentation($0, 0.8) ?? Data() } // 변환
    .observe(on: ConcurrentDispatchQueueScheduler(qos: .utility)) // IO
    .scan(Data(), +) // 누적 압축
    .observe(on: MainScheduler.instance)
    .subscribe(onNext: updateProgress)
```

***

### 6️⃣ Mini Quiz

1. `flatMapLatest` 내부에서 `.share()`를 사용하면 어떤 효과?
2. `scan`으로 누적 합을 구할 때 메모리 증가를 방지할 방법은?
3. `buffer(timeSpan:.seconds(5), count: Int.max, ...)` 설정 시 주의점?

<details>

<summary>Answers</summary>

1. 여러 구독자가 `flatMapLatest` 결과를 **공유**하되, 최신 내부 스트림 1개로 한정 — 네트워크 캐싱 효과 & 리소스 절약.
2. 누적값이 계속 커질 경우 **`map { $0 % 10 }`** 처럼 크기를 제한하거나, `reduce`로 최종값만 필요할 때 스트림 완료 후 전달.
3. count가 무한대이므로 5초 동안 발생한 **모든 이벤트를 메모리에 버퍼**; 이벤트 폭주 시 메모리 급증 위험 — 적절한 count 설정 필요.

</details>

***

> 다음 ▶️ **combining** 로 이동해 `merge`, `combineLatest`, `zip` 등을 학습합시다. 🚀
