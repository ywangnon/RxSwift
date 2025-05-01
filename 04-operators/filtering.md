# Filtering

## 🚿 Filtering Operators — 필요한 이벤트만 ‘걸러내기’

> **“Signal-to-noise ratio를 높여라!”**

Filtering 계열 오퍼레이터는 Observable 스트림에서 **특정 조건에 맞는 값이나 이벤트만 통과**시키고 나머지는 무시합니다. UI 입력 디바운스, 이벤트 무시, 중복 제거 등 흔히 사용됩니다.

***

### 1️⃣ 대표 Filtering 오퍼레이터 요약

| 오퍼레이터                     | 설명                      | 시그니처 요약             |
| ------------------------- | ----------------------- | ------------------- |
| `filter`                  | 조건에 맞는 값만 통과            | `(Element) -> Bool` |
| `take` / `takeLast`       | 처음(N) / 마지막(N)개 가져오기    | `Int`               |
| `skip` / `skipLast`       | 처음(N) / 마지막(N)개 건너뛰기    | `Int`               |
| `takeWhile` / `skipWhile` | 조건 true 동안 가져오기 / 건너뛰기  | `(Element) -> Bool` |
| `distinctUntilChanged`    | 동일 연속 값 제거              | `==` 기준 (Equatable) |
| `elementAt`               | 특정 인덱스 값 1개             | `Int`               |
| `ignoreElements`          | `.completed`만 전달 (값 무시) | —                   |

> **Time 기반**(`debounce`, `throttle`)은 별도 문서 _timeBased.m&#x64;_&#xC5D0;서 다룹니다.

***

### 2️⃣ 실전 스니펫

#### A. 로그인 버튼 활성화 — 공백 제거 후 비어있지 않은지 체크

```swift
textField.rx.text.orEmpty
    .map { $0.trimmingCharacters(in: .whitespaces) }
    .filter { !$0.isEmpty }
    .bind(to: viewModel.username)
```

#### B. 연속 탭 방지 — distinctUntilChanged()

```swift
button.rx.tap
    .map { Date() }
    .distinctUntilChanged { prev, next in next.timeIntervalSince(prev) < 0.3 }
    .subscribe(onNext: performAction)
```

#### C. 처음 3개 샘플만 프리뷰 후 자동 종료

```swift
photoStream
    .take(3)
    .subscribe(onNext: showSample,
               onCompleted: { print("sample end") })
```

***

### 3️⃣ Operator 연산 순서 주의

* `take(1).skip(1)` ➡️ 결과 0개 (take가 먼저 실행)
* `skip(1).take(1)` ➡️ 두 번째 요소 1개 전달
* 체인 순서를 바꿀 때 **단위 테스트**로 기대 값 확인 필요

***

### 4️⃣ Error & Completion 전파 규칙

* Filtering 오퍼레이터는 **onNext**만 필터링, **onError** / **onCompleted**는 그대로 전파.
* `ignoreElements`는 onError도 **무시하지 않음** — 에러는 상위로 전달되어야 디버깅 용이.

***

### 5️⃣ Performance & Memory Tips

* 큰 배열을 배출하는 스트림에서 `filter` 후 `map` 순서로 최적화 (불필요한 매핑 줄이기).
* `distinctUntilChanged` 커스텀 비교는 값 타입일 때 비용 고려.

***

### 6️⃣ Mini Quiz

1. `skipUntil(trigger)`와 `takeUntil(trigger)` 차이를 설명하라.
2. `distinctUntilChanged`가 `Equatable`이 아닌 타입에서 동작하려면?
3. `takeLast(1)`가 메모리 사용량에 미치는 영향은?

<details>

<summary>Answers</summary>

1. **skipUntil**: Trigger Observable이 _첫 이벤&#xD2B8;_&#xB97C; 방출할 때까지 원본 값을 무시, 이후 모두 통과. **takeUntil**: Trigger가 값 방출하는 순간 _그 이후_ 원본 스트림을 **완료**시켜 더 이상 통과하지 않음.
2. 클로저 버전을 사용: `distinctUntilChanged { prev, next in /* 비교식 */ }`.
3. `takeLast`는 **버퍼**에 최대 N개 요소를 저장하므로, N이 작아도 메모리 버퍼가 존재; 대용량 스트림에서 큰 N이면 메모리 증가.

</details>

***

> 다음 ▶️ **transforming.md** 로 이동해 `map`, `flatMap`, `scan` 등 변환 오퍼레이터를 학습합니다. 🚀
