---
layout: post
current: post
navigation: True
title: RxSwift Subject, RxCocoa Relay
date: 2019-12-23 18:00:00
tags: RxSwift
class: post-template
subclass: 'post tag-fables'
author: gaki
---  

## RxSwift Subject


기본적으로 RxSwift를 실제로 적용시킬때는 크게 **실시간으로 Observable에 새로운 값을 수동으로 추가하고 Subscriber에게 방출하는 것** 이다. 즉 **Observable**과 **Observer** 이 둘 모두 동작할 수 있는 것이 필요하다.

이를 Subject가 관리한다. 데이터를 추가(emit) 할 수도 있고, 방출(Subscribe)할수도 있다.



#### Subject Example

```swift
        // 1 
        let subject = PublishSubject<String>()
        // 2
        subject.onNext("데이터 받을 준비 되어있음?")	// Print 되지 않음
        // 3
        let subscriptionOne = subject
            .subscribe(onNext: { (string) in
                print(string)									 // Print 되지 않음
            })
        // 4
        subject.on(.next("1"))      // Print 1
        // 4와 동일 
        subject.onNext("2")					// Print 2
```

1. `PublishSubject` 생성
   - 추가된 정보를 수정한 다음에 Subscriber에게 배포한다.
   - 데이터의 타입은 사용자가 정의한 것을 기반으로 하는데, 여기서는 String 타입이다. 따라서 받는정보, 배포하는 정보 모두 String 타입이다.
2. Subject에 추가를 해도 일단은 아무것도 출력 되지 않는다.
3. Subject를 `subscribe` 해도 출력 되지 않는다.
   - `PublishSubject`는 현재의 subcriber에만 이벤트를 방출한다. 따라서 어떤 정보가 추가 되었을때 구독 하지 않았다면 그 값을 얻을 수 없다.
4. subject가 구독이 바로 이전에 '3'에서 되었기 때문에 이제부터 출력이 된다.
   - `subscriber`연산자와 비슷하게, `.on(.next(_:))`는 새로운 `.next` 이벤트를 subject에 삽입하고, 이벤트 내의 값들을 파라미터로 통과시킨다.



위의 예제는 4가지 Subject중 하나인 `PublishSubject`에 대해서 간단하게 다루어 보았다. Subject의 역할을 분명히 해야할 필요가 있다.

ReactiveX Observable에는 "Hot Observable" 과 "Cold Observable"의 개념이 존재한다. **Subject는 Cold Observable을 Hot 하게 변형하는 효과를 얻을 수 있다.** 우선 Hot 과 Cold 의개념을 간단하게 집고 넘어가보자

> 출처 : https://brunch.co.kr/@tilltue/18
>
> 정말 좋은 비유로 설명이 잘되어있다..



##### Hot Observable

생성과 동시에 이벤트를 방출하기 시작한다. 구독(`subscribe` )시점과 상관 없이 Observer 들에게 이벤트를 중간부터 전송해준다. 같은말로 Connectable Observable 이라고도 한다.

##### Cold Observable

Observer가  구독(`subscribe` ) 되는 시점부터 이벤트를 생성하여 방출하기 시작한다. Hot Observable로 생성 되지 않은 것들은 모두 여기에 속한다.

> 유튜브 시청을 예로 들면 **Hot Observer** 실시간 방송은 시청자(Observer)가 어느 시점에서 방송을 청취하던, 방송은 일정 시간동안만 방송이 된다. 즉 시청자(Observer)는 방송을 시청(Subscribe) 하는 시점 부터(middle emit event) 방송을 볼 수 있다.(이벤트를 받을 수 있다) -> 구독하는 시점과는 상관없어서 이벤트 방출의 시점이 보장 되지 않는다.
>
> **Cold Observer**의 경우 일반 VOD와 같다. 시청자(Observer)가 시청을 시작하면(Subscribe) 처음부터 방송이 시작된다.(emit 1, 2, 3 ... ) -> 구독하는 시점부터 이벤트가 순차적으로 방출되므로 보장이 된다.



다시 정리를 해보자면 Subject는 구독의 시점부터 순차적으로 이벤트를 발행하던 것(Cold Observable)을 구독 시점에 관계 없이 이벤트를 발행(Hot Observable)할 수 있다는 것이다.



### Subject의 4가지 종류



## 1. PublishSubject

- 구독(`subscribe`)된 순간 새로운 이벤트 수신을 알리고 싶을때 사용한다.
- 구독 되는 시점에서 구독을 멈추거나, `.completed`, `.error` 이벤트를 통해 subject가 완전 종료 될 때까지 계속 지속된다.

![image](https://user-images.githubusercontent.com/33486820/71338901-e70d0880-2594-11ea-861d-72cba396f5bf.png)

Subscriber들이 새로운이 벤트 2, 3을 수신 하기 위해 이벤트 전에 구독(`subscribe`)을 통해 이벤트를 수신하고 이는 원래의 subject가 완전히 종료가 되거나 에러가 발생할 때까지 subscriber 들은 계속해서 구독한 시점부터 후의 이벤트를 수신한다.

- 예제

```swift
        let subject = PublishSubject<String>()
        // 아직 구독 되지 않았기 떄문에 출력 x
        subject.onNext("Is anyone listening?")
        // 첫번째 subscriber가 구독을 실시함 -> 이때 부터 이벤트 발행 가능
        let subscriptionOne = subject
            .subscribe(onNext: { (string) in
                print("1)", string)
            })
        subject.on(.next("1"))
        subject.onNext("2")

        // 1: 두번째 Subscriber가 구독함
        let subscriptionTwo = subject
            .subscribe({ (event) in
                // 출력될 이벤트에 2)를 붙여서 출력함
                print("2)", event.element ?? event)
            })

        // 2: 이벤트 3을 subject에 추가
        subject.onNext("3")

        // 3: subscriber 1 dispose호출로 방출 -> 구독 취소로 종료
        subscriptionOne.dispose()
        subject.onNext("4")

        // 4: subject complete로 종료
        subject.onCompleted()

        // 5: 기존의 subject를 완전 종료를 시킨다
        subject.onNext("5")

        // 6: 두번째 구독자도 완전 종료를 시킨다.
        subscriptionTwo.dispose()

        let disposeBag = DisposeBag()

        // 7: subject가 완전 종료 된 후에 구독해봤자 생명을 불어 넣을 순 없다! 하지만 complete 이벤트는 방출한다.
        subject
            .subscribe {
                print("3)", $0.element ?? $0)
        }
            .disposed(by: disposeBag)

        subject.onNext("?")
```

> 좋은 예시로 시간에 민감한 서비스를 개발할때 사용할 수 있을거 같다.
>
> 시간에 따라 사용자에게 알림을 해야할 경우를 생각해보자. 경매 이벤트 앱에서 10시에 에어팟 프로를 10만원에 판매하는 이벤트를 진행한다. 10시 5분 전부터 접속한 사용자에게 1분 간격으로 남은 시간을 알림으로 띄워주는 기능을 만들려고 할때 사용자는 5분전에 와서 대기하여 모든 알림 이벤트를 확인할 수 있겠지만. 3분전, 2분전, 30초전에 들어올 경우 위와 같은 이벤트가 모두 수신 될 필요가 없다. 즉 subject는 모든 이벤트를 가지고 있고 구독하는 시점에 따라 이벤트를 보내야할 경우에 대해서 설계할때 유용할것 같다.





## 2. BehaviorSubject

- `PublisherSubject` 과 차이점은 구독 시점 바로 이전의 `.next` 이벤트를 반복한다는 점 그 외에는 모두 동일하다.

![image](https://user-images.githubusercontent.com/33486820/71341049-0eb39f00-259c-11ea-9867-1bfc6c00aefa.png)

- 예제

```swift
        // 1: BehaviorSubject는 초기 값이 있어야한다. -> 항상 이전의 최신값을 방출 하기 때문에 반드시 초기값이 있어야한다.
				let subject = BehaviorSubject(value: "Initial Value")
        let disposeBag = DisposeBag()
        
        // 2
        subject
            .subscribe{
                // 이전의 "Initial Value" 초기값있기 떄문에 해당 이벤트를 먼저 수신한다.
                print("1)", $0.element ?? $0)
            }
            .disposed(by: disposeBag)
        
        // 3: 다시 최신값은 "Initial Value"에서 "서버에 데이터 요청" 이벤트로 변경이 된다.
        subject.onNext("서버에 데이터 요청")

        // 4: Subject에 에러가 발생하게 되면 구독하던 다른 Subscriber들에 대한 error 이벤트를 찍게 되므로 아래와 같이 두번 찍히게 된다.
        subject.onError(NetworkingError.requestError)

        // 5
        subject
            .subscribe {
                print("2)", $0.element ?? $0)
            }
            .disposed(by: disposeBag)

/* 출력
1) Initial value
1) 서버에 데이터 요청
// 1, 2에대해 에러가 두번 찍히게 됨 
1) Reuqest Error!
2) Request Error!
*/
```

> BehaviorSubject의 경우 구독자(Subscriber)가 항상 Subject 이벤트의 최신을 가지고 있다는 점이 특징이다. 고로 이 점은 View를 가장 최신의 데이터로 미리 채우기 위한 경우 용이하다. 예를 들어 유튜브 같은 동영상 서비스에서 서버에 대용량 데이터를 요청했을때 사용자는 하얀 바탕에 네트워크 인디 케이터가 돌아가는것만 보면서 기다리기 보다는 먼저 가져온 데이터를 미리 세팅하여 UI를 가장 이전의 최신으로 세팅해놓고 보여주는 방식이면 효과적일 것이다.





## 3. ReplaySubject

- `ReplaySubject` 는 생성시 선택한 특정 크기까지, 방출하는 최신 요소를 일시적으로 **캐시나 버퍼** 한다. 그리고 저장된 이 요소들을 새 Subscriber에게 방출한다.

![image](https://user-images.githubusercontent.com/33486820/71343224-7d93f680-25a2-11ea-8771-9f5703ed1d15.png)

- 예제

```swift
       // 1: 미리 버퍼 사이즈를 지정한다 -> 2
        let subject = ReplaySubject<String>.create(bufferSize: 2)
        let disposeBag = DisposeBag()
        
        // 2
        subject.onNext("1")
        subject.onNext("2")
        subject.onNext("3")
        
        // 3
        subject
            .subscribe {
                print("1)", $0.element ?? $0)
        }
        .disposed(by: disposeBag)
        
        subject
            .subscribe {
                print("2)", $0.element ?? $0)
        }
        .disposed(by: disposeBag)

/*
1) 2
1) 3
2) 2
2) 3
*/
```

1.  `.create(bufferSize:)` 를 사용하여 버퍼 사이즈가 2인 `ReplaySubject` 우선 만든다.
   - 참고 : `.createUnbounded()` 는 모든 이벤트 저장 -> 메모리에 유의
2. 3개의 이벤트 요소를 Subject에 추가한다.
3. Subscriber 1 이 Subject에 요소 3이 추가된 시점에서 구독한다. 이제 위의 2개와 비교해보자
   - PublishSubject: 없음
   - BehaviorSubject: 3
   - ReplaySubject: 2, 3 -> 버퍼가 2이기때문에 최신의 2개가 구독자에게 보여진다. 
   - 마찬가지로 Subscriber 2도 똑같이 buffer에 2, 3이 저장되어있기 때문에 보여진다.



여기서 Subject에 요소 4를 하나 더 추가하고 구독자를 추가해보자

```swift
        // 4. 요소 4 추가 -> 버퍼에는 3, 4
        subject.onNext("4")
        // 5. Subscriber 3 추가
        subject
            .subscribe {
                print("3)", $0.element ?? $0)
        }
        .disposed(by: disposeBag)

/*
1) 4
2) 4
3) 3
3) 4
*/
```

4. 요소 4를 추가했기 때문에 버퍼에는 `subject` 의 최신 요소 3, 4가 저장되어있다.
5. 하지만 이때 앞의 Subject 종류들과 동일하게 `subject` 에 요소가 추가 되었기 때문에 구독자 1, 2에게도 요소 4가 추가된다.
   - 구독자 3의 경우 3, 4가 추가되어 아래와 같이 출력이 된다.



이때 `subject`에 에러를 발생시켜 보자. 

```swift
        // 4
        subject.onNext("4")
				// 에러 발생!
				subject.onError(NetworkingError.requestError)
        // 5
        subject
            .subscribe {
                print("3)", $0.element ?? $0)
        }
        .disposed(by: disposeBag)

/*
1) 4
2) 4
1) anError
2) anError
3) 3
3) 4
3) anError
*/
```

- `subject`에 에러가 발생하여 완전히 종료가 되었음에도 불구하고 바로 아래 구독자 3에게 버퍼에 저장이 되어있던 요소 3, 4가 전달이 되어 아래와 같이 출력이 되는 것을 볼 수 있다.
- 구독자의 경우 `subect` 가 종료가 되면 바로 종료가 되게 구현을 해야하고 불필요한 재방출을 방지하여야한다. 우린 이미 수동적으로 요소를 방출시키는 법을 알고 있다. 바로 `dispose()` 를 이용하면 된다.

```swift
        // 4
        subject.onNext("4")
				// 에러 발생!
				subject.onError(NetworkingError.requestError)
				// dispose를 통해 subject의 요소 방출
				subject.dispose()
        // 5
        subject
            .subscribe {
                print("3)", $0.element ?? $0)
        }
        .disposed(by: disposeBag)

/*
1) 4
2) 4
1) error(anError)
2) error(anError)
3) error(Object `RxSwift.(unknown context at $106525000).ReplayMany<Swift.String>` was already disposed.)
*/

```

- 위와 같이 `dispose`를 통해 버퍼에 들어있던 요소를 강제로 방출 시키고 나면 각 구독자들은 버퍼에 저장되어있는 요소값, 특히 구독자 3의 경우 어떠한 이벤트 요소도 수신할 수 었이 error 이벤트만 받는 것을 확인할 수 있다.
- 하지만 `ReplaySubject` 에 명시적으로 `dispose()`를 호출하는 것은 지양해야한다. 만약에 `subject`의 구독을 disposeBag에 모두 추가하고, 가령 ViewController나 ViewModel과 같이 `subject`의 소유자가 dealloc이 되면 위와 같이 모든 것을 강제로 dispose 해버리면 안되기 때문에 유념해야한다.

> ReplaySubject는 다양하게 사용되어 사용자의 편의를 제공할 기능에 추가될 수 있다.
>
> 예를들어 사용자의 특정 이벤트를 알고있어야할때 이것을 쓰면 된다. 검색이나, 최근 항목, 최근 행한 이벤트 등을 버퍼에 저장하여 구독하는 즉시 그 이벤트 들을 처리 할 수 있을 것이다.





## 4. AsyncSubject

- 구독의 시점과 상관 없이 Subject가 Complete 이벤트를 방출하여 정상 종료가 될때 구독자는 마지막 요소를  발생하고 종료를 한다.

![image](https://user-images.githubusercontent.com/33486820/71347162-0a43b200-25ad-11ea-812e-a975340bd749.png)

- 예제

```swift
        let subject = AsyncSubject<String>()
        let disposeBag = DisposeBag()
        
        subject
            .subscribe {
                print("1)", $0.element ?? $0)
        }
        .disposed(by: disposeBag)
        
        subject.onNext("1")
        subject.onNext("2")
        subject.onNext("3")
        subject.onCompleted()
/*
1) 3
1) completed
*/
```

- 마지막 요소를 구독자가 수신하고 구독자도 정상 종료가 되는 것을 확인 할 수 있다.



만약 도중에 에러가 발생하면 구독자도 에러이벤트를 처리하고 종료가 된다.

![image](https://user-images.githubusercontent.com/33486820/71347314-6c041c00-25ad-11ea-8b03-8f4910fab24e.png)


## Relay



Relay Class는 RxCocoa의 RxRelay에 구현이 되어 있다. 두가지의 클래스가 존재한다

- `PublishRelay`
- `BehaviorRelay`

<br>

RxSwift의 Subject와 어떤 차이점이 있고 언제 사용하는지는 다음과 같다.

- RxSwift의 Subject의 경우 `onNexct` 로 이벤트를 처리하는데 Relay는 `accept` 로 처리한다.
- error와 complete를 내보낼 수 없다. 즉 오직 `accept` 만 내보낼 수 있다. 이는 도중에 에러가 발생하거나, 완료가 되면 무시를한다. 
- 에러로 인한 종료, 완료로 인한 종료가 되지 않는다 이는 스트림이 죽지않는 다는 것을 의미하고 UI와 같은 경우 에러가 나거나 완료가 났다고 종료가 되면 안되기 때문에 **UI 전용으로 사용** 하는 것이 주 목적이다.

<br>

## RxCocoa Relay

<br>

### Publish Relay

<br>

`PublishRelay`는 `PublishSubject`의 `Wrapper 클래스` 이다.

`PublishSubject` 처럼 구독 이후의 발생하는 이벤트 들만 알 수 있다.

```swift
/// PublishRelay is a wrapper for `PublishSubject`.
///
/// Unlike `PublishSubject` it can't terminate with error or completed.
public final class PublishRelay<Element>: ObservableType {
    public typealias E = Element

    private let _subject: PublishSubject<Element>
    
    // Accepts `event` and emits it to subscribers
    public func accept(_ event: Element) {
        self._subject.onNext(event)
    }
    
    /// Initializes with internal empty subject.
    public init() {
        self._subject = PublishSubject()
    }

    /// Subscribes observer
    public func subscribe<O: ObserverType>(_ observer: O) -> Disposable where O.E == E {
        return self._subject.subscribe(observer)
    }
    
    /// - returns: Canonical interface for push style sequence
    public func asObservable() -> Observable<Element> {
        return self._subject.asObservable()
    }
}
```

<br>



### Behavior Relay

<br>

`BehaviorRelay` 는  `BehaviorSubject`의 `Wrapper 클래스` 이다.

마찬가지로 `BehaviorSubject` 와 동일하게 이전의 최신 이벤트값을 가져 올 수 있다. 고로 초기 값이 있어야한다

값은 `.value`를 통해서 현재의 값을 가져 올 수 있다.



```swift
/// BehaviorRelay is a wrapper for `BehaviorSubject`.
///
/// Unlike `BehaviorSubject` it can't terminate with error or completed.
public final class BehaviorRelay<Element>: ObservableType {
    public typealias E = Element

    private let _subject: BehaviorSubject<Element>

    /// Accepts `event` and emits it to subscribers
    public func accept(_ event: Element) {
        self._subject.onNext(event)
    }

    /// Current value of behavior subject
    public var value: Element {
        // this try! is ok because subject can't error out or be disposed
        return try! self._subject.value()
    }

    /// Initializes behavior relay with initial value.
    public init(value: Element) {
        self._subject = BehaviorSubject(value: value)
    }

    /// Subscribes observer
    public func subscribe<O: ObserverType>(_ observer: O) -> Disposable where O.E == E {
        return self._subject.subscribe(observer)
    }

    /// - returns: Canonical interface for push style sequence
    public func asObservable() -> Observable<Element> {
        return self._subject.asObservable()
    }
}
```









## Reference

- https://brunch.co.kr/@tilltue/4
- [RxSwift-fimuxd](https://github.com/fimuxd/RxSwift/blob/master/Lectures/03_Subjects/Ch3.%20Subjects.md#c-publishsubjects로-작업하기)
- [RxSwift Documents](http://reactivex.io/intro.html)

