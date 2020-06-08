---
layout: post
title: RxSwift flatMap과 flatMapLatest 사용법
subtitle: ""
categories: swift
tags: rxSwift
comments: false
---

<br>


 Swift의 고차함수인 **map**의 경우 매개변수로 함수를 받아 함수를 적용한 새로운 컨테이너를 반환 하는 목적으로 많이 사용하고 있습니다. 마찬가지로 Swift에서 시퀀스의 nil 필터링을 목적으로 **flatMap** 을 사용하곤 했습니다. 하지만 Swift 4.1로 올라오면서 **compactMap** 으로 그 기능이 이관되었지만 flatMap은 여전히 **여러개의 시퀀스를 하나의 시퀀스로 이어주는 역할** 로써 사용하고 있습니다.



RxSwift에서는 이 고차함수들이 어떻게 사용되고 또 차이점은 어떤게 있는지 알아보도록 하겠습니다.



## Map과 FlatMap 차이

<br>

#### Map

map은 새로운 이벤트 타입으로 변경시켜 반환 해주는 역할을 합니다. E Type 이벤트를 R Type이벤트로 반환 해줍니다.

```swift
 public func map<R>(_ transform: @escaping (Self.E) throws -> R) -> RxSwift.Observable<R>

```

map의 경우 Observable<E> 에서 Observable<R> 로 내가 원하는 타입으로 변경할때 클로저에 해당 변경해주는 작업을 추가하여 사용하곤합니다.

<br>



#### Map의 실전 예제

1) 지도를 터치했을때 Observable<NMGLatlng> 로 반환 하나는 `didTapMapView ` delegate 메서드를 Observable<String> 으로 변경시켜 반환하고 그걸 label의 text에 바인드 하는 작업입니다.

```swift
naverMapView.rx.didTapMapView
		.asObservable()
		.map {
      return "위도: \(latlng.lat) 경도 \(latlng.lng)"
    }
		.bind(to: label.rx.text)
		.disposed(by: disposeBag)     

// 또는 아래와 같이 사용

naverMapView.rx.didTapMapView
		.asObservable()
		.map { convertToString($0) }
		.bind(to: label.rx.text)
		.disposed(by: disposeBag)

func convertToString(_ latlng: NMGLatLng) -> String {
  return "위도: \(latlng.lat) 경도 \(latlng.lng)"
}
```

<br>



2) 서버에서 데이터를 요청하고 즉시 UI에 String으로 컨버팅 해서 보여주는 작업

서버에 길찾기 데이터를 요청한 ViewModel의 navigationData에 있는 거리(distance) 값을 label에 bind를 거는 작업입니다.

```swift
viewModel.getNavigationData
		.map { "\($0.distance)" }
		.bind(to: distLabel.tx.text)
		.disposed(by: disposeBag)
```



<br>

**map** 의 경우 위의 예제와 같이 다른 타입의 Observable<Type> 으로 변환할때 사용하고 있습니다.

**faltMap** 의 경우 조금 더 복잡하여 자세하게 알아보도록 하겠습니다.



<br>

#### FlatMap

Observavble 시퀀스의 element 당 한 개의 새로운 Observable 시퀀스를 생하고 이렇게 생성된 여러개의 새로운 시퀀스를 하나의 시퀀스로 합쳐줍니다.

```swift
 public func flatMap<O: ObservableConvertibleType>(_ selector: @escaping (E) throws -> O)
        -> Observable<O.E>
```

flatMap의 경우 **inner Observable** 을 다루기 위해 사용합니다.

> inner Observable이란?  Observable안에 있는 Observable을 말한다.

<br>

위의 정의로는 잘 와닿지는 않습니다.. 그래서 그림으로 한번 확인을 해보도록 하겠습니다!

<img width="971" alt="image" src="https://user-images.githubusercontent.com/33486820/72409309-ae81d880-37a8-11ea-9e4a-2e87eefb4408.png">

우선 초기에 1, 2, 3의 element가 있습니다. 이 element가 flatMap을 만나게 되면 value에 10을 꼽하는 작업을 합니다.

그리고 flatMap은 element가 flatMap을 통해 값을 변경함과 동시에 **element에 해당하는 새로운 sequence** 를 반환하게 됩니다. 마찬가지로 1, 2, 3은 flatMap을 통해 가각의 value에 10을 곲한 채로 새롭게 이벤트를 방출하는 시퀀스가 됩니다. 그리고 Original Sequence의 value에 값이 변경이 되면 해당 element의 새롭게 추가된 시퀀스에 마찬가지로 추가가 됩니다.

위의 예제에서 element 1이 4로 value값이 변경이 되면 element 1의 sequence 즉 10 뒤에 40이 추가가 되어 이벤트를 방출하게 됩니다.

마찬가지로 element 2의 값이 5로 변경이 되어 새롭게 추가된 sequence에 50이 추가가 되어 최종 sequence에 50을 방출하는 것을 볼 수 있습니다.

<br>

조금 더 자세하게 이해하기 위해 코드로 이해를 해보겠습니다!

#### 코드로 확인해보기

```swift

struct Element {
    var element: BehaviorSubject<Int>
}
...

				disposeBag = DisposeBag()
        
        // 초기 element 1, 2, 3
        let element1 = Element(element: BehaviorSubject(value: 1))
        let element2 = Element(element: BehaviorSubject(value: 2))
        let element3 = Element(element: BehaviorSubject(value: 3))

        let finalSequence = PublishSubject<Element>()
        // transform된 element를 subscribe한다. 이벤트가 발생할시 값을 출력
        finalSequence
            .flatMap { $0.element }
            .subscribe(onNext: {
                print($0 * 10)
            })
            .disposed(by: disposeBag)
        
        finalSequence.onNext(element1)
        finalSequence.onNext(element2)
        finalSequence.onNext(element3)
        
        element1.element.onNext(4)
				element2.element.onNext(5)
				finalSequence.dispose()
/*
10
20
30
40
50
*/

```

flatMap은 Observable의 모든 Element의 변화를 관찰하고 있습니다. 즉 Original Sequence 모든 element를 관찰하고 그 전에 관찰했던 Observable들을 계속 관찰하기 때문에 누락 되는 부분이 없이 전부 최종 sequence에 이벤트를 확인할 수 있습니다.



이와는 다르게 Original Sequence의 **가장 마지막 Element만 관찰** 하는 **flatMapLatest** Operator가 있습니다.



<br>



## FlatMapLatest

flatMapLatest는 오직 **Latest Observable** 만 관찰합니다.

앞에서 flatMap은 inner Observable을 처리할때 사용하며 Observable의 새로운 sequence를 만들어 모든 요소를 관찰한다고 했다. 하지만 **flatMapLatest** 는 오직 **마지막으로 추가된 sequence의 inneer Observable 이벤트만 구독** 한다고 생각하시면 됩니다.

그림으로 좀더 명확하게 짚고 넘어가겠습니다.

<img width="801" alt="image" src="https://user-images.githubusercontent.com/33486820/72413368-03c2e780-37b3-11ea-9133-c4cb3aef437a.png">

코드로 보기 전에 위의 상황을 설명 드리겠습니다.

우선 Original Sequence에 Element 1, 2, 3이 있습니다. 이 요소들이 차례데로 들어오고 flatMapLatest를 총해 구독하면서 값을 10씩 곱해주는건 앞의 예제와 동일합니다.

1. Element 2가 추가 한 후에 Element 1의 value값이 3으로 변경 시켰지만 무시가 되어 Fina lSequence에 전달이 되지 않습니다. 그 이유는 현재 Element 2가 flatMapLatest에 의해 Final Sequence에 추가가 된 상황, 즉 현재 **Latest(마지막) Sequence는 Element 2 이기 때문에 Element 2이 외의 모든 값들은 무시가 되어 finalSequence에 전달이 되지 않습니다**

2. 그 다음 Element 3이 추가가 되고 정상적으로 finalSequence에 30 값이 전달이 되고 이때 Latest Sequence는 Element 3 value의 Sequence이기 때문에 마찬가지로 ( Element 1의 value 4로변경 -> 40 ) 이 Final Sequence에 전달되지 않습니다.

3. 2와 같은 이유로 ( Element 2의 value 5로 변경 -> 50 ) 이 Final Sequence에 전달되지 않습니다.

<br>



#### 코드로 확인해보기

```swift
        disposeBag = DisposeBag()
        
        // 초기 element 1, 2, 3
        let element1 = Element(element: BehaviorSubject(value: 1))
        let element2 = Element(element: BehaviorSubject(value: 2))
        let element3 = Element(element: BehaviorSubject(value: 3))
        var elementS = [Element]()
        
        elementS.append(Element(element: BehaviorSubject(value: 6)))
        elementS.append(Element(element: BehaviorSubject(value: 7)))

        let finalSequence = PublishSubject<Element>()
        // transform된 element를 subscribe한다. 이벤트가 발생할시 값을 출력하게 했다.
        finalSequence
            .flatMapLatest { $0.element }
            .subscribe(onNext: {
                print($0 * 10)
            })
            .disposed(by: disposeBag)
        
        finalSequence.onNext(element1)
        finalSequence.onNext(element2)
        // 무시됨 현재 마지막 Sequence: element2
        element1.element.onNext(3)
        finalSequence.onNext(element3)
        // 무시됨 현재 마지막 Sequence: element3
        element1.element.onNext(4)
        element2.element.onNext(5)
        // final Sequence에 전달됨
        element3.element.onNext(6)
				
				finalSeqeunce.dispose()

/*
10
20
30
60
*/
```



**flatMapLatest** 는 가령 지도앱을 통해 길찾기를 할때 '중국집' 을 검색한다고 생각해보자 '주' 에서 'ㅇ' 를 입력하기 전에 

추천검색어로 '주유소', '주안역' .. 등등이 tableView에 보여지게 하고 그리고 마침내 'ㅇ'을 쳤을때 중에 관한 키워드를 추천해주는 기능을 구현할때 주 에서 중으로 넘어갈때 이전의 '주'를 무시하고 '중'으로 시작하는 키워드로 추천해주고 싶을때 flatMapLatest를 사용한다(너무 큰 범주에서 설명을 했지만. . 현재 상황의 요소의 시퀀스에만 집중하는 애다! 라고 생각하면 편할듯 하다)



<br>



## Reference

- [RxSwift Reactive Programming with Swift](https://store.raywenderlich.com/products/rxswift)

