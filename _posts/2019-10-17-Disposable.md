---
layout: post
current: post
navigation: True
title: RxSwift Disposable
date: 2019-10-15 20:54:00
tags: RxSwift
class: post-template
subclass: 'post tag-fables'
author: gaki
---  


## Disposing과 종료

앞서 Observable은 구독(Subscription)을 받기 전까지는 아무 행동도 취하지 않고 이벤트 처리의 명시만 해줄 뿐이다.Observable의 종료는 이와 반대로 구독을 취소함으로써 Observable을 **수동적** 으로 종료시킬 수 있다.

`subscribe` 메서드는 **Disposable** 객체를 반환한다. "처분할 수 있는 , 사용후 버릴 수 있는" 이란 사전적 정의를 가진 이 객체는 Observable의 구독을 취소할 수 있게 해준다.

한가지 상황을 가정해보자 아래와 같이 Observable 에서는 `DispatchTImeInterval` 을 제공하여 take에서 넘겨받은 정수 크기 만큼 스트림을 허용하는 기능을 한다. 

```swift
        Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
            .take(10)
            .subscribe(onNext: { (event) in
                print(event)
            }, onError: { (error) in
                print(error)
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("onDisposed")
            })
/* 출력
0
1
2
3
4
5
6
7
8
9
completed
onDisposed
*/

```

위의 코드를 `viewDidLoad` 메서드에서 호출하게 되면 앱 실행과 동시에 로그 창에는 1초 간격으로 1씩 늘어난 정수가 출력된다. 그리고 모든 이벤트를  처리를 완료하면 `completed` 가 출력되고 뒤 이어 `onDisposed` 가 출력된다.

이는 Observable이 모든 요소에 대한 이벤트 처리를 완료 하였기때문에 도중에 error 없이 completed 되었다고 뜨고 마지막으로 더 이상 수행해야할 작업이 없기때문에 `onDisposed` 가 출력된다.

위의 Observable의 경우 `take(10)` 크기만큼의 작업량을 갖기 때문에 빠른시간에 수행이되고 작업이 완료된다. 하지만 작업이 무한히 많고 또 도중에 작업을 중단해야하는 경우 deadLine을 설정하거나 특정 작업을 할때 Observable의 구독을 취소해야할 것이다.

극단적인 상황으로 앱의 ViewController 를 해제하는 상황을 만들어 보자

```swift
        DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
            // deinit
            // 13 부터 deprecated
            // UIApplication.shared.keyWindow?.rootViewController = nil           
            UIApplication.shared.windows.first?.rootViewController = nil
        }
/* 출력
0
1
2
Deinit ViewController
3
4
5
6
7
8
9
completed
onDisposed
*/
```

rootViewController를 강제로 nil을 대입하면서 앱 실행 3 초 뒤에 강제로 뷰 컨트롤러를 삭제했다. 하지만 예상처럼 3초 후에 `viewDidDisappear` 메서드가 호출되고 종료될 줄 알았지만 Observable은 계속해서 log를 찍어내고 마지막으로 완료가 되었다는 수행을 한다.

우리는 특정 상황이나 작업량에 제한을 둬야하는 상황이 많기 떄문에 Observable을 수동적으로 제어해야한다. RxSwift 는 `dispose()` 메서드를 Disposable 프로토콜에서 제공하고 있다.



### `.dispose()`

Observable의 Subscribe 메서드는 Didsposable 타입의 객체를 반환한다.

<img width="793" alt="image" src="https://user-images.githubusercontent.com/33486820/66972418-857d7900-f0cf-11e9-865e-efbce1a1849b.png">

Disposable 은 프로토콜로 정의되어있고 `dispose()` 메서드를 가지고있다. 이 메서드를 사용해서 우리는 구독을 수동적으로 취소할 수 있다.

<img width="233" alt="image" src="https://user-images.githubusercontent.com/33486820/66972513-d1c8b900-f0cf-11e9-97d7-394df8608781.png">

```swift
let disposable =  Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
            .take(10)
            .subscribe(onNext: { (event) in
                print(event)
            }, onError: { (error) in
                print(error)
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("onDisposed")
            })
        
        DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
          	disposable.dispose()         
            UIApplication.shared.windows.first?.rootViewController = nil
        }

/* 출력
0
1
2
onDisposed
Deinit ViewController
*/
```

앞에서 언급한 것 처럼 Subscribe 메서드의 Disposable 객체를 반환 받기 위해 핸들링 해주었고 이 객체를 원하는 시점에서 종료하고 싶은 영엿에 `dispose()` 메서드를 사용하면 정상종료가 되는 것을 확인 할 수 있다.



### DisposeBag

실제 상황에서 구독을 받는 Observable들이 여러개가 존재할 수 있다. 이를 dispsable 컬렉션을 만들어 관리를 할 수 도있다. 하지만 각각의 구독에 대해서 일일히 관리하는 것은 효율적이지 못하기 때문에, RxSwift 에서 제공하는 `DisposeBag` 타입을 이용할 수 있다.

```swift
// DisposeBag 객체 선언
var disposeBag = DisposeBag()

Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
            .take(10)
            .subscribe(onNext: { (event) in
                print(event)
            }, onError: { (error) in
                print(error)
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("onDisposed")
            })
						.disposed(by: disposeBag)	// 방출되는 disposable 객체를 disposeBag에 삽입
        
        DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
          	disposeBag = DisposeBag()         
            UIApplication.shared.windows.first?.rootViewController = nil
        }
```

disposeBag 객체를 생성하고 Observable이 subscribe를 통해 이벤트를 처리하고 방출하는 disposable 객체를 `disposed(:)` 메서를 통헤 disposeBag에 삽입한다.

disposeBag의 경우 `.dispose()` 와 같은 메서드를 지원하지 않아 단순하게 새로 생성하면 초기화가 된다.



### create

`next` 이밴트를 이용해서 Observable을 만든것 처럼 `.create` 연산자로 만드는 다른 방법이 있다.

```swift
     Observable<String>.create({ (observer) -> Disposable in
         // 1
         observer.onNext("1")		// == on(.next(_:)) 와 동일
         
         // 2
         observer.onCompleted()	// == on(.compledted) 와 동일
         
         // 3
         observer.onNext("?")		
         
         // 4
         return Disposables.create()
     })
 				.subscribe(
             onNext: { print($0) },
             onError: { print($0) },
             onCompleted: { print("Completed") },
             onDisposed: { print("Disposed") }
    		 ).disposed(by: disposeBag)

 /* 출력
  1
  Completed
  Disposed
 */
```

`create` 는 escaping 클로저로, escaping 에서는 `AnyObserver` 를 취한 뒤 Disposable 을 반환한다. 여기서 `AnyObserver` 란 generic 타입으로 Observable sequence에 값을 쉽게 추가할 수 있다. 추가한 값은 subscriber에 방출한다.

1. `.next` 이벤트를 Observer에 추가한다. 
2. `.completed` 이벤트를 Observer에 추가한다.
3. 추가적으로 `.next` 이벤트를 추가한다.
4. disposable 을 리턴한다.

하지면 위의 예제에서 이벤트 요소 중 하나인 `?` 는 호출되지 않는다. `1` 이벤트가 `onNext(_:)`에 의해 방출된 후에 `.onCompleted()` 를 통해서 Observable은 종료되었으므로 두번재 `onNext(_:)`는 방출되지 않는다.

여기서 `1` 과 `?` 모두 방출이 되어 출력이 되기 위해서는 아래와 같이 주석 처리를 해주면 된다.

```swift
 Observable<String>.create({ (observer) -> Disposable in
         // 1
         observer.onNext("1")		// == on(.next(_:)) 와 동일
         
         // 2
         // observer.onCompleted()	// == on(.compledted) 와 동일
         
         // 3
         observer.onNext("?")		
         
         // 4
         return Disposables.create()
     })
 				.subscribe(
             onNext: { print($0) },
             onError: { print($0) },
             onCompleted: { print("Completed") },
             onDisposed: { print("Disposed") }
    		 )
				// .disposed(by: disposeBag)
```

`onCompleted()` 메서드와 `disposed(by: disposeBag)` 메서드를 생략하게 되면 종료를 위한 이벤트도 방출되지 않고 `dispose` 도 하지 않디 때문에 결과적으로 메모리에 낭비가 발생한다. 

> 결론 : 처리해야할 요소 당 이벤트가 많아 작업이 무한정으로 길어질때나 특정 작업시에 수동적으로 Observable의 구독을 취소하고 종료할 수 있다. 그리고 Observable은 여러개가 존재하고 또 각각의 Subscribe를 관리하는 것에는 DisposeBag을 사용하여 효율적으로 관리할 수 있다.



<hr>

## Reference

- https://github.com/fimuxd/RxSwift
- [순한맛 RxSwift](https://mym0404.blog.me/221585744991)
- [Reactive X: Observable](http://reactivex.io/documentation/ko/observable.html)
- [RxJs Marbles](https://rxmarbles.com/)

