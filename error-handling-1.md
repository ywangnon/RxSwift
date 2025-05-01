# Error Handling

## 🚨 Error Handling Operators — `catchError`, `retry`, `materialize`

> **“실패는 당연하다. 문제는 복구다.”**

RxSwift의 오류 처리 오퍼레이터는 **스트림 오류를 가로채고, 대체하거나, 재시도**할 수 있게 해 줍니다. 앱 안정성의 핵심이므로 확실히 이해해 둡시다.

***

### 1️⃣ 주요 오퍼레이터 요약

| Operator               | 역할                               | 시그니처 요약                                   |
| ---------------------- | -------------------------------- | ----------------------------------------- |
| `catchError`           | 오류 발생 시 **대체 Observable**로 스위치   | `(Error) -> Observable<R>`                |
| `catchErrorJustReturn` | 특정 **기본값**으로 대체                  | `(R)`                                     |
| `retry(_ count:Int?)`  | 오류 발생 시 **재구독**(무한 or n회)        | `Int?`                                    |
| `retryWhen`            | 오류 스트림을 **변환해 조건 재시도**           | `(Observable<Error>) -> Observable<Void>` |
| `materialize`          | `Event<Element>`로 래핑(값·완료·오류 포함) | —                                         |
| `dematerialize`        | 반대 연산                            | —                                         |

***

### 2️⃣ 실전 스니펫

#### A. `catchError` — Offline Placeholder

```swift
api.fetchNews()
    .catchError { _ in localCache.news() }
    .bind(to: tableView.rx.items(...))
```

#### B. `retry` — 네트워크 재시도 3회

```swift
api.post(data)
    .retry(3)
    .subscribe(onError: showToast)
```

#### C. `retryWhen` — 지수 백오프 재시도

```swift
api.fetch()
    .retryWhen { errors in
        errors.enumerated().flatMap { attempt, error -> Observable<Int> in
            guard attempt < 3 else { return Observable.error(error) }
            let delay = pow(2.0, Double(attempt))
            return Observable<Int>.timer(.seconds(Int(delay)), scheduler: MainScheduler.instance)
        }
    }
```

#### D. `materialize` — 오류를 UI 이벤트로 표시

```swift
api.fetch()
    .materialize()
    .subscribe(onNext: { event in
        switch event {
        case .next(let value): render(value)
        case .error(let e): showError(e)
        case .completed: break
        }
    })
```

***

### 3️⃣ 패턴 요약

| 시나리오                          | 추천 오퍼레이터                             | 이유                |
| ----------------------------- | ------------------------------------ | ----------------- |
| 네트워크 다운 → 로컬 캐시 대체            | `catchError`                         | 스트림 유지 & Fallback |
| Transient API 실패 재시도          | `retry(3)`                           | 간단 반복             |
| HTTP 429 Rate‑Limit 재시도 헤더 반영 | `retryWhen`                          | 서버 제공 시간까지 delay  |
| 사용자 알림 없이 실패 무시               | `catchErrorJustReturn(defaultValue)` | UX 간결             |

***

### 4️⃣ Error vs Completion 흐름

* `catchError`로 대체한 Observable이 완료되면 **상위 스트림도 완료**.
* `retry`는 **source Observable**을 재구독하고, 에러가 아닌 정상 완료 시 그 즉시 완료.

> `retryWhen` 내부 Observable이 `onCompleted`하면 재시도 **종료**; `onError`면 **전파**.

***

### 5️⃣ pitfalls

1. **retry 무한 루프**: 서버 영구 다운 시 배터리 소모 — `retry(∞)` 대신 `retry(3)`.
2. **catchErrorJustReturn** 남발: 에러 로깅 누락 → 디버깅 어려움.
3. **materialize** 후 `dematerialize` 깜빡: 이벤트 타입 잊어버려 체인 오류.

***

### 6️⃣ Mini Quiz

1. `retryWhen` 내부 Observable이 `.next()` 없이 `.completed`만 하면 결과는?
2. `catchError`와 `onErrorResumeNext` 차이는?
3. `materialize`된 스트림을 `share()` 후 다시 `dematerialize`할 때 주의할 점?

<details>

<summary>Answers</summary>

1. 재시도 **중단**하고 source 스트림을 **completed**로 종료한다.
2. `onErrorResumeNext`는 **오류 타입 무시**하고 마지막 Observable을 **무조건 연결**(Error→Next), `catchError`는 **조건부** 처리 후 에러를 전파하거나 대체.
3. share 이후 동일 Event 객체를 여러 Subscriber가 사용 — **thread‑safety** 문제 없지만, 다시 `dematerialize` 시 반드시 원본 Event 타입 유지.

</details>

***

> 🎉 Operators 챕터 끝! 이제 원하는 주제로 더 깊이 파보거나 테스트 케이스 작성으로 학습을 강화해 보세요. 🚀
