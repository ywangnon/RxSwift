# AsyncSubject

## ⏳ AsyncSubject — 완료 시점의 ‘마지막 값’만 방출

> **핵심 키워드:** _Single Emission at Completion_ · _Hot Subject_ · _Single-result Tasks_

`AsyncSubject<Element>`는 **스트림이 완료될 때 단 한 번, 마지막 `onNext` 값만 전파**합니다. 완료 전에 구독해도 이벤트를 받지 못하다가, `.onCompleted()` 호출 순간 최신 값 1개를 전달하고 스트림을 종료합니다.

***

### 1️⃣ 특징 요약

| 항목      | 내용                                      |
| ------- | --------------------------------------- |
| 초기값     | ❌ 없음                                    |
| 캐시 크기   | 1 (마지막 값, 완료 시)                         |
| 구독 시 전달 | 완료 전 → 없음 / 완료 후 → 마지막 값과 `.completed`  |
| 스트림 종료  | `.onCompleted()` 필수 (에러 시 `.error`만 전파) |

***

### 2️⃣ 기본 사용 예제

```swift
let subject = AsyncSubject<String>()
let bag = DisposeBag()

subject.onNext("Step 1")
subject.onNext("Step 2")

subject.subscribe(onNext: { print("Subscriber:",$0) },
                  onCompleted: { print("Completed!") })
    .disposed(by: bag)

subject.onNext("Final Step")
subject.onCompleted()
```

**출력**

```
Subscriber: Final Step
Completed!
```

* ‘Final Step’만 방출됨 (마지막 onNext).
* `onCompleted()`가 없으면 **영원히 무응답**이므로 반드시 호출.

***

### 3️⃣ 실전 패턴 — 파일 업로드 결과 전달

```swift
func uploadFile(_ data: Data) -> Observable<UploadResponse> {
    let subject = AsyncSubject<UploadResponse>()

    api.upload(data: data) { result in
        switch result {
        case .success(let response):
            subject.onNext(response) // 마지막 결과
            subject.onCompleted()
        case .failure(let error):
            subject.onError(error)
        }
    }

    return subject.asObservable() // 호출 측: 결과 1개만 수신
}
```

* 여러 중간 진행률이 필요하면 `PublishSubject`로 따로 브로드캐스트.

***

### 4️⃣ Subject 비교 테이블

| Subject          | 발행 시점             | 방출 개수 | 사용 사례        |
| ---------------- | ----------------- | ----- | ------------ |
| PublishSubject   | onNext 즉시         | N     | 버튼 탭         |
| BehaviorSubject  | onNext 즉시 + 캐시 1  | N     | 상태 저장        |
| ReplaySubject    | onNext 즉시 + 캐시 N  | N     | 채팅 히스토리      |
| **AsyncSubject** | **onCompleted 시** | **1** | 비동기 작업 최종 결과 |

***

### 5️⃣ 에러 처리 & Empty 시나리오

* `.onError(e)` 호출 시 **값 없이 에러**만 전달.
* 완료 전에 `onNext`가 한 번도 없었다면 구독자는 **completed만 수신**(값=nil).

```swift
subject.onCompleted() // next 없었다면 completed 이벤트만 전달
```

***

### 6️⃣ Mini Quiz

1. `AsyncSubject`에 값을 3개 보내고 `.onCompleted()`를 생략하면 구독자는 무엇을 받는가?
2. RxSwift의 `Single<Element>`와 `AsyncSubject` 기능 차이 한 줄 요약.
3. Combine 프레임워크에서 유사한 기능을 구현하려면 어떤 Combine 객체나 오퍼레이터 조합을 사용하면 될까?

<details>

<summary>정답</summary>

1. **아무 것도 받지 못한다.** 스트림이 완료되지 않았기 때문에 이벤트가 전파되지 않는다.
2. `Single`은 **완료 또는 에러를 자동 관리**하는 _cold_ 단발 스트림 팩토리, `AsyncSubject`는 **hot subject로 직접 onNext/onCompleted 제어** 가능.
3. Combine에서는 `PassthroughSubject` + `.last()` 오퍼레이터 또는 `Future`를 사용하여 완료 시 단일 값 전달 패턴을 구성할 수 있다.

</details>

***

> 다음 ▶️ **schedulers.md** 로 이동해 스레드·실행 컨텍스트 제어를 학습합니다. 🚀
