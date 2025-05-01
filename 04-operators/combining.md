# Combining

## 🧩 Combining Operators — 여러 스트림을 ‘조합’하기

> **“Two (or more) Observables enter, one leaves.”**

Combining 오퍼레이터는 **다수 Observable을 하나로 합쳐** 풍부한 컨텍스트를 만들거나, 이벤트 순서를 제어합니다. UI·네트워킹·동기화 로직에서 핵심 도구로 쓰입니다.

***

### 1️⃣ 대표 Combining 오퍼레이터 표

| Operator         | 핵심 역할                             | 특징 요약                                        |
| ---------------- | --------------------------------- | -------------------------------------------- |
| `merge`          | 여러 스트림 이벤트를 **동시에** 방출            | 순서 보장 X, 최대 동시 n개 제어 `merge(maxConcurrent:)` |
| `concat`         | **직렬** 연결, 앞 스트림 완료 후 다음 구독       | 순서 보장, back-pressure 제어 쉬움                   |
| `concatMap`      | `flatMap`+`concat` (Transform+직렬) | 각 요소→Observable 직렬 실행                        |
| `combineLatest`  | 각 스트림의 **최신값** 결합                 | 모든 소스 1개 이상 값 후 시작                           |
| `zip`            | 동일 인덱스끼리 **쌍** 생성                 | 짝 맞지 않으면 대기, 개수 = 최소(stream)                 |
| `withLatestFrom` | 트리거 스트림에 최신 파라미터 주입               | 버튼 탭 + 최신 폼 값                                |
| `sample`         | 지정 주기/스트림 시, 최신값 샘플링              | 폴링 UI Throttle                               |
| `amb`            | 먼저 이벤트를 발생시킨 스트림만 사용              | 레이스 Winner 선택                                |
| `switchLatest`   | 내부 Observable 최신 것만 구독            | `flatMapLatest` + identity                   |

***

### 2️⃣ 실전 스니펫

#### A. `combineLatest` — 이메일 & 비밀번호 유효성

```swift
Observable.combineLatest(emailValid, passwordValid) { $0 && $1 }
    .bind(to: loginButton.rx.isEnabled)
```

#### B. `merge` — 다중 센서 데이터 통합

```swift
Observable.merge(accelerometer, gyroscope, magnetometer)
    .subscribe(onNext: processMotion)
```

#### C. `zip` — 평행 네트워크 요청 결과 매핑

```swift
Observable.zip(api.userInfo(), api.userPosts()) { (user, posts) in
    ProfileViewModel(user: user, posts: posts)
}
```

#### D. `withLatestFrom` — ‘전송’ 버튼 + 최신 텍스트

```swift
sendButton.rx.tap
    .withLatestFrom(messageField.rx.text.orEmpty)
    .filter { !$0.isEmpty }
    .bind(to: viewModel.sendMessage)
```

***

### 3️⃣ 오류 전파 규칙

| Operator            | 에러 발생 시 동작                          |
| ------------------- | ----------------------------------- |
| `merge`             | **즉시 전파**(fail-fast) — 다른 스트림 구독 해제 |
| `concat`            | 앞 스트림 에러 → 전파 & 시퀀스 종료              |
| `combineLatest/zip` | 어느 하나 에러 → 전파 & 종료                  |
| `amb`               | 선택된 스트림 에러만 고려                      |

에러 복구는 `catchError`, `retry` (다음 장)로 처리.

***

### 4️⃣ 성능 & 메모리 고려

* `combineLatest`는 각 소스의 **최신값을 캐시** → 큰 객체는 참조 전달 or `map` 축소.
* `merge` 동시성 무제한 시 이벤트 폭발 가능 → `maxConcurrent` 설정.
* `zip`은 짝 맞추기 위해 _버퍼_ 사용 → 비대칭 스트림 속도면 메모리 증가.

***

### 5️⃣ 패턴 별 Best Operator

| 시나리오                   | 추천 Operator     | 이유            |
| ---------------------- | --------------- | ------------- |
| 폼 입력 2개+ 버튼 활성         | `combineLatest` | 최신 상태 결합      |
| 파일 A→B→C 순차 업로드        | `concatMap`     | 직렬 보장 + 변환    |
| 서버1/2 중 빠른 응답 사용       | `amb`           | 레이스 승자 스트림 채택 |
| 주기적 Heartbeat + 실시간 알림 | `merge`         | 서로 독립 동시 처리   |

***

### 6️⃣ Mini Quiz

1. `merge`와 `combineLatest` 차이를 **이벤트 함수** 관점에서 설명해 보라.
2. `zip` 사용 시 두 소스의 이벤트 속도가 크게 다르면 어떤 문제가? 대응책은?
3. `withLatestFrom`과 `sample`의 차이 한 줄 요약.

<details>

<summary>Answers</summary>

1. `merge`는 **onNext를 interleave**하여 그대로 전달, 결합 함수 없음. `combineLatest`는 각 소스 onNext 시 **결합 클로저**를 호출해 1개의 새 값 생성.
2. 빠른 스트림 이벤트가 **버퍼에 쌓여 메모리 증가**; `zipLatest` 대안(없다면 combineLatest) 또는 `buffer/skip`로 속도 맞추기.
3. `withLatestFrom`은 _트리거 스트림_ onNext 때 **파라미터 스트림 최신값**을 가져오고, `sample`은 지정 간격/트리거 시 최신값만 전달(트리거값 무시).

</details>

***

> 다음 ▶️ **timeBased.md** 로 이동해 `debounce`, `throttle`, `timeout` 등 시간 기반 오퍼레이터를 살펴봅시다. 🚀
