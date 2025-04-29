# ReplaySubject

## 🔁 ReplaySubject — 값 N개를 ‘되감기’하는 캐시 Subject

> **핵심 키워드:** _Buffer Size_ · _Hot Stream_ · _Late Subscriber Catch‑up_

`ReplaySubject<Element>`는 **지정한 버퍼 크기만큼 과거 값을 저장**하여, 새로운 구독자에게 한꺼번에 재생(Replay)합니다. 동영상 플레이어의 ‘되감기’처럼 **이전 이벤트 컨텍스트**가 중요할 때 유용합니다.

***

### 1️⃣ 특징 한눈에 보기

| 항목      | 설명                                      |
| ------- | --------------------------------------- |
| 초기값     | ❌ (옵션)                                  |
| 버퍼 크기   | 1 \~ ∞ (`bufferSize:`)                  |
| 구독 시 전달 | 버퍼 내 모든 이벤트 재생 후 실시간 수신                 |
| 스트림 종료  | `.onCompleted()` / `.onError()` 후 버퍼 해제 |

> 버퍼 크기를 `Int.max`로 설정하면 **전체 기록**을 재생할 수 있으나 메모리 사용에 유의하세요.

***

### 2️⃣ 기본 사용 예제

```swift
// 최근 2개 값 캐시
let subject = ReplaySubject<String>.create(bufferSize: 2)
let bag = DisposeBag()

subject.onNext("1️⃣ One")
subject.onNext("2️⃣ Two")
subject.onNext("3️⃣ Three")

// 구독자는 Two, Three부터 수신
subject
    .subscribe(onNext: { print($0) })
    .disposed(by: bag)

subject.onNext("4️⃣ Four")
```

**출력**

```
2️⃣ Two
3️⃣ Three
4️⃣ Four
```

***

### 3️⃣ 실전 패턴 — 채팅 메시지 캐싱

```swift
final class ChatRoomService {
    private let messageSubject = ReplaySubject<Message>.create(bufferSize: 50)
    private let bag = DisposeBag()

    // 외부 노출 스트림
    var messages: Observable<[Message]> {
        messageSubject
            .scan([]) { acc, new in acc + [new] }
    }

    func receive(message: Message) {
        messageSubject.onNext(message)
    }
}

// 새로 들어온 사용자도 최근 50개 메시지 즉시 표시 가능
```

***

### 4️⃣ 메모리 & 퍼포먼스 최적화

1. **버퍼 크기**: 실제 UI/비즈니스 요구 범위로 제한 — 무제한 X.
2. **`windowTime:`**: 시간 기반 캐싱(`create(bufferSize:timeScheduler:)`)로 오래된 데이터 자동 제거.
3. **스트림 종료 처리**: 방 나가기 등 필요 시 `.onCompleted()` 호출해 버퍼 해제.

***

### 5️⃣ Subject 간 비교 요약

| Subject           | 캐시 크기     | 초기값 | 사용 사례                  |
| ----------------- | --------- | --- | ---------------------- |
| PublishSubject    | 0         | ❌   | 버튼 탭, 알림               |
| BehaviorSubject   | 1         | ✔️  | 토글 상태, Form Validation |
| **ReplaySubject** | n         | ❌   | 채팅 히스토리, 로그 재생         |
| AsyncSubject      | 1 (완료 시점) | ❌   | 파일 업로드 결과, 단일 응답       |

***

### 6️⃣ Mini Quiz

1. 버퍼 크기를 `3`으로 설정했는데 이벤트를 5개 발행 후 새로 구독하면 몇 개를 받나요?
2. `ReplaySubject`의 버퍼가 메모리를 과도하게 사용하지 않도록 두 가지 최적화 방법은?
3. Combine에서 동일한 기능을 담당하는 Subject는?

<details>

<summary>정답</summary>

1. **3개** — 가장 최근 3개(이벤트 #3, #4, #5)를 재생 후 이후 실시간 값 수신.
2. **최적화 방법**
   * 버퍼 크기를 요구사항에 맞게 **작게 설정** (`bufferSize`).
   * 시간 제한 **`windowTime`** 인자를 사용해 오래된 이벤트를 자동 제거(`.create(bufferSize:timeInterval:scheduler:)`).
3. Combine의 **`ReplaySubject`**(Swift 5.10, CombineExt) 또는 서드파티 _CombineExt_ 패키지의 `ReplaySubject`가 유사 동작을 제공합니다.

</details>

***

> 다음 문서 ▶️ **AsyncSubject** 로 이동해 완료 시점 단일 값 전달 패턴을 알아봅니다. 🚀
