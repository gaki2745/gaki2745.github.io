---
layout: post
title: RxSwift Observable 합성
subtitle: ""
categories: swift
tags: rxSwift
comments: false
---


## Observable 합성

RxSwift에서 서로 연관된 이벤트 들을 같이 처리해야 할 때 가 있다.

API를 호출할때 두개의 API를 동시에 호출할 수도 있고 사용자가 앱에 전근할때 몇가지 이벤트들을 복합적으로 처리해야하는 상황이 있다.

그러므로 RxSwift에서는 Observable를 결합할수 있는 함수를 4개 제공한다.



### 1. combineLatest

두 Observable의 각각의 이벤트가 발생할 때 두  Observable의 마지막 이벤트 들을 묶어서 전달한다. 두개의 이벤트는 타입이 달라도 된다.

<img width="768" alt="image" src="https://user-images.githubusercontent.com/33486820/71503190-8e50af00-28b7-11ea-8fb4-ffcfda17036c.png">

위의 표를 보면 2의 이벤트 사이에 B, C, D이벤트가 들어와 차례대로 2B, 2C, 2D가 나오는 것을 볼 수 있다. 즉 바로 이전의 상대방의 최신이벤트와 결합되어 방출되는 효과를 볼 수 있다.

- 예제

```swift
        let disposeBag = DisposeBag()
        let korean = Observable.from(["가","나", "다"])
        let english = Observable.from(["A","B","C"])
        Observable.combineLatest(
            korean,
            english
        ) { ($0, $1) } // -> korean, english
            .subscribe {
                print($0)
        }
        .disposed(by: disposeBag)

/*
next(("가", "A"))
next(("나", "A"))
next(("나", "B"))
next(("다", "B"))
next(("다", "C"))
completed
*/

```







### 1-2. withLatestFrom

두개의 Observable을 합성하지만 한쪽 Observable의 이벤트가 발생할때마다 합성해주는 메서드다. 위의 combine과 반대로 한쪽 Observable의 이벤트가 기준이 된다. 만약 합성할 다른쪽 이벤트가 없으면 이벤트는 스킵이 된다.



<img width="733" alt="image" src="https://user-images.githubusercontent.com/33486820/71503605-3ca92400-28b9-11ea-9088-c6debcf03cef.png">

위의 예제에서 첫번째 Observable이 기준이 된다. 이벤트 1 이전의 두번째 Observable의 이벤가 없기 때문에 스킵이 되고 2 부터 A -> 2A, 3은 최신인 D와 결합 -> 3D, 4도 마찬가지로 4D, 5D... 이렇게 진행이 된다.







### 2. merge

**같은 타입**의 이벤트를 발생하는 Observable을 합성하는 함수이다. 각각의 이벤트 모두 수신이 가능하다.

<img width="779" alt="image" src="https://user-images.githubusercontent.com/33486820/71504463-e3db8a80-28bc-11ea-82a6-7aa9e02950be.png">

같은 타입의 두 이벤트를 말그대로 "병합"해서 수신한다. 만약 도중에 에러가 발생하게 되면 merge 작업은 중단 된다.

- 예제

```swift
        let disposeBag = DisposeBag()
        let obs1 = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
            .map{ "obs1: \($0)" }
        let obs2 = Observable<Int>.interval(.seconds(2), scheduler: MainScheduler.instance)
            .map{ "obs2: \($0)" }
        
        let startTime = Date().timeIntervalSince1970
        
        Observable
            .of(obs1, obs2)
            .merge()
            .subscribe {
                print("\($0): \(Int(Date().timeIntervalSince1970 - startTime))")
        }
        .disposed(by: disposeBag)

```







### 3. switchLatest

Observable을 Switch 할수 있는 Observable이다. 

<img width="793" alt="image" src="https://user-images.githubusercontent.com/33486820/71505522-d58f6d80-28c0-11ea-9e83-edf8e31dd399.png">

- 예제

```swift
        let disposeBag = DisposeBag()
        let subjectA = PublishSubject<String>()
        let subjectB = PublishSubject<String>()
        let switchTest = BehaviorSubject<Observable<String>>(value: subjectA)
        switchTest.switchLatest().subscribe {
            print($0)
        }.disposed(by: disposeBag)
        subjectA.onNext("A-1")
        switchTest.onNext(subjectB)
        subjectA.onNext("A-2")
        subjectB.onNext("B-1")
        subjectB.onNext("B-2")

/*
next(A-1) -> A-2 가 출력이 안된다.
next(B-1)
next(B-2)
*/
```

두개의 Subject A,B가 있고 초기 switch는 SubjectA를 값으로 가지는 BehaviorSubject로 선언했다. 그리고 A Subject에 이벤트를 추가하다가 A-1 이벤트 다음 B로 switch 했다. 이때 다음으로 추가된 Subject A의 이벤트 A-2는 스킵이 된다. 그리고 Subject B의 이벤트 들이 수신 되는 것을 확인할 수 있다.







### 4. zip

zip의 경우 두 Observable의 발생 순서가 같은 이벤트를 조합해서 이벤트를 발생시킨다.

<img width="781" alt="image" src="https://user-images.githubusercontent.com/33486820/71506316-cf4ec080-28c3-11ea-938e-85251a63ad51.png">

두개의 Observable에서 이벤트 순서가 동일한 거 끼리 짝지어서 발생시키는 것을 확인할 수 있다.

- 예제

```swift
				let disposeBag = DisposeBag()
        let korean = Observable<String>.from(["가", "나", "다", "라"])
        let number = Observable<Int>.from([1, 2, 3])
        
        Observable
            .zip(
                korean,
                number
        )
            .subscribe {
                print($0)
        }
        .disposed(by: disposeBag)
/*
next(("가", 1))
next(("나", 2))
next(("다", 3))
completed
*/
```

위의 예제에서 짝이 맞아야 이벤트가 방출되는 것을 확인할 수 있다 그렇기 때문에 절대적으로 이벤트 요소가 작은 Observable의 이벤트 요소 개수에 맞추어 진다. 그리고 두개의 타입은 상관이 없는 것을 확인할 수 있다.







### 5. concat

concat은 두개 이상의 Observable을 하나로 연결해주는 함수이다. 하나의 Observable이 Complete 되어야 그 다음 Observable을 바로 뒤에 이어준다.

<img width="788" alt="image" src="https://user-images.githubusercontent.com/33486820/71507466-f7d8b980-28c7-11ea-8cef-4c2e58c1b854.png">

- 예제

```swift
        let disposeBag = DisposeBag()
        let korean = Observable<String>.from(["가", "나", "다", "라"])
        let english = Observable<String>.from(["A","B","C"])
        
        korean
            .concat(english)
            .subscribe {
                print($0)
        }
        .disposed(by: disposeBag)
/*
next(가)
next(나)
next(다)
next(라)
next(A)
next(B)
next(C)
completed
*/
```

Observable의 타입이 같아야한다. Subject에서도 테스트 해보자

```swift
        let disposeBag = DisposeBag()
        let subjectA = PublishSubject<String>()
        let subjectB = PublishSubject<String>()
        
        subjectA.subscribe {
            print($0)
        }
        .disposed(by: disposeBag)
        
        subjectB.subscribe {
            print($0)
        }
        .disposed(by: disposeBag)
        
        
        Observable
            .concat([subjectA, subjectB])
            .subscribe {
                print("concat event: \($0)")
        }
        .disposed(by: disposeBag)
        
        subjectA.onNext("A - event1")
        subjectA.onNext("A - event2")
        subjectA.onCompleted()
        subjectB.onNext("B - evnet1")
        subjectB.onNext("B - event2")
      // subjectB.onError(MyError.anError)
        subjectB.onCompleted()

/*
next(A - event1)
concat event: next(A - event1)
next(A - event2)
concat event: next(A - event2)
completed
next(B - evnet1)
concat event: next(B - evnet1)
next(B - event2)
concat event: next(B - event2)
completed
concat event: completed
*/
```

subject를 concat으로 이어서 구독을 해보았다. 이때 먼저 나온 subject가 completed가 되어야 다음 subject를 concat 할 수 있다.







### 6. amb

가장 처음 발생한 Observable 이벤트만 사용하게 하는 함수이다.

<img width="775" alt="image" src="https://user-images.githubusercontent.com/33486820/71509835-0d51e180-28d0-11ea-8c66-31d47a62a9e7.png">

3개의 Observable이 있지만 두번째 Observable의 이벤트 1, 2, 3 이 수신 된 것을 확인할 수 있다.

이처럼 가장 먼저 이벤트를 발생 시키는 Observable의 이벤트를 연결하는 함수이다.

- 예제

```swift
        let disposeBag = DisposeBag()
        let first = Observable<Int>.interval(.seconds(5), scheduler: MainScheduler.instance).map {
            "first: \($0)"
        }
        let second = Observable<Int>.interval(.seconds(2), scheduler: MainScheduler.instance).map {
            "second: \($0)"
        }
        let third = Observable<Int>.interval(.seconds(4), scheduler: MainScheduler.instance).map {
            "third: \($0)"
        }
        
        first.amb(second).amb(third).subscribe(onNext: { e in
            print(e)
            })
            .disposed(by: disposeBag)

/*
second: 0
second: 1
second: 2
...
*/
```







### 7. startWith

기존의 Observable에 초기 이벤트를 지정 해줄 수 있다.

<img width="776" alt="image" src="https://user-images.githubusercontent.com/33486820/71510692-0b3d5200-28d3-11ea-9459-402f476d016a.png">









## Reference

- [https://brunch.co.kr/@tilltue/6](https://brunch.co.kr/@tilltue/6)
- [RxSwift 기본익히기](https://medium.com/@trilliwon/rxswift-기본-익히기-1-d4d77ce63ca8?)
