---
layout: post
current: post
navigation: True
title: SwiftUI Combine(1) - Publisher & Subscriber
date: 2020-04-24 20:18:00
tags: SwiftUI
class: post-template
subclass: 'post tag-fables'
author: gaki
---  



Combine을 통해 네트워킹 코드를 작성하던 도중 확실히 그 원리에 대해 정리하고 넘어가도록 하겠습니다.

RxSwift를 사용하다가 SwiftUI가 나오면서 이부분을 어떻게 Combine이 대신하고 있는지 또 어떻게 사용하는지 그리고 Combine의 궁극적인 목적(?)은 무엇인지 하나하나 공부해보겠습니다.



### Combine이란?

> Customize handling of asynchronous events by combining event-processing operators.

애플에서 기존의 UIKit에서 새로운 UI 프레임워크인 SwiftUI를 만들었고 따라서 기존의 데이터바인딩도 변경되었습니다.

이에 애플은 새로운 프레임워크인 **Combine** 이라는 프레임워크를 제공합니다.

비동기 리액티브 프로그래밍을 위한 프레임워크로 기존에 사용하던 RxSwift 와 매우 유사하게 보입니다.

RxCocoa에서는 RxSwift를 그리고 SwiftUI에서는 Combine이라고 생각하시면 될것같습니다.

어려운 말들이 많지만 아래의 한 문장으로 용도를 정의할 수 있습니다.

> "시간의 흐름에 따라 발생하는 이벤트를 처리하기 위한 API"





### Combine의 3가지 요소

Combine은 크게 3가지의 요소로 이루어져있습니다.

- Publisher
- Operator
- Subscriber

<img width="1046" alt="image" src="https://user-images.githubusercontent.com/33486820/80186596-33527d00-8649-11ea-8061-cbca247199f5.png">

위의 구조는 Rx에서 흔히 본 생성자-소비자 패턴과 유사한것을 알 수 있습니다. 이벤트에 관해 구독자(Subscriber)가 발행자(Publisher)에게 데이터를 요청(input)하면 이에 발행자는 그에 맞는 데이터를 전달(output)하게 됩니다.

중간에 Operator의 경우 데이터의 흐름을 제어하고 input/output 데이터를 가공하는 역할을 하게됩니다.





### Publisher & Subscriber

앞에서 말씀드렸듯이 Publisher와 Subscriber의 경우 데이터를 요청 하고 전달하는 흐름이 존재하기 때문에 아래와 같이 도식화 할 수 있습니다.

<img width="762" alt="image" src="https://user-images.githubusercontent.com/33486820/80187865-52520e80-864b-11ea-8eda-a4b168b7cecb.png">

Publisher의 경우 Subscriber의 요청에 Output 타입의 데이터를 주고 실패할 경우 Failure 타입을 반환하기 위해 가지고 있습니다. 이와 마찬가지로 Subscriber도 요청할 데이터의 Input 타입 그리고 Failure타입을 가지고 있습니다.

여기서 중요한 것은 Subscriber와 Publisher는 각각의 Input/Output, Failure에 대해 서로 동일한 타입을 가져야 합니다.



#### Publisher

- 타입이 시간에 따라 일련의 값을 전송(transmit)할 수 있음을 선언합니다. 

- Publisher는 하나 이상의 Subscriber인스턴스에게 element를 제공합니다.

```swift
protocol Publisher {
    associatedtype Output
    associatedtype Failure : Error
    func receive<S>(subscriber: S) where S : Subscriber, Self.Failure == S.Failure, Self.Output == S.Input
}
```

- Publisher는 `receive(subscribe:)` 메소드를 구현해 Subscriber를 accept하게 됩니다.

https://developer.apple.com/documentation/combine/publisher](https://developer.apple.com/documentation/combine/publisher)



#### Subscriber

- Publisher로부터 요청한 input 에 대해 output을 받게됩니다.

```swift
public protocol Subscriber : CustomCombineIdentifierConvertible {
    associatedtype Input
    associatedtype Failure : Error

    func receive(subscription: Subscription)
    func receive(_ input: Self.Input) -> Subscribers.Demand
    func receive(completion: Subscribers.Completion<Self.Failure>)
}
```

Rx에서의 생산자-소비자와 동일하게 성공하거나 실해할시에 둘 사이에 오고가는 타입은 동일해야 합니다.





### Publisher&Subscriber Example

간단한 예제를 통해 Publisher와 Subscriber를 구현해보도록 하겠습니다.

우선 Publisher를 생산하는 방법에는 어떤것이 있는지 알아보겠습니다.

![image](https://user-images.githubusercontent.com/33486820/80191321-7a903c00-8650-11ea-9bc0-3b34e833536b.png)

위의 6개를 통해 생성할 수 있고 빌트인 방식으로 가장 쉽게 생성할 수 있는 `Just` 를 통해 예제를 작성해 보겠습니다.



#### Just를 이용한 Publisher생성 예제

`Just` 는 오직 하나의 값만을 출력하고 끝나게 되는 가장 단순한 형태의 Publisher로 Combine에서 빌트인(Built-in)형태로 제공하는 Publisher입니다.

각 Subscriber에게 output을 **오직 한번만(just once)** 출력한 다음 완료하는 Publisher입니다.



```swift
// ex1)
let publisher = Just("Gaki")

let subscriber = publisher.sink { (value) in
    print(value)
}

// ex2)
Just("Gkai")
    .sink {
        print($0)
}

// 출력: Gaki

```

"Gaki" 라는 String 타입의 데이터를 `publisher` 가 `Just` 를 통해 발행을 했습니다. 아래와 같이 `subscriber` 가 구독을 하기 전까지는 데이터를 가져올 수 없기 때문에 `sink` 메소드를 통해 값을 가져옵니다. 



#### Publisher의 값을 구독하기 위한 sink 메소드

>This method **creates the subscriber** and immediately requests an unlimited number of values, prior to returning the subscriber.

sink 메소드는 **subscriber** 를 생성하고 subscriber를 리턴하기 전에 즉시 unlimited number of values를 요청합니다.

subscriber인스턴스를 sink가 하게 되고 이를 publisher에게 요청을 하게 됨으로써 데이터를 가져올 수 있습니다.



sink에는 `receiveComplietion` 을 통해 receive를 완료했을때를 구분지어 제어할 수 있는 메소드도 존재합니다.

![image](https://user-images.githubusercontent.com/33486820/80192348-fd65c680-8651-11ea-8cd8-416d4f03b6eb.png)

- 예제 1

```swift
let provider = (1...10).publisher

provider
    .sink(receiveCompletion: { _ in
        print("데이터 전달 전송완료")
    }, receiveValue: { data in
        print(data)
    })

/*
1
2
3
4
5
6
7
8
9
10
데이터 전달 전송완료
*/
```

- 예제 2

```swift
let publisher = Just("Gaki")

let subscriber = publisher.sink(receiveCompletion: { (result) in
    switch result {
    case .finished:
        print("데이터 전송완료")
    case .failure(let error):
        print(error.localizedDescription)
    }
}, receiveValue: { (value) in
    print("전달받은 데이터:", value)
})

/*
전달받은 데이터: Gaki
데이터 전송완료
*/
```

`receiveCompletion` 에서 Receive를 완료와 실패를 나누어 제어할 수 있습니다.





### Subscriber 구독을 통해 값을 받자!

위의 Just 예제에서 우리는 sink를 통해 publisher가 발행하는 값으 구독하여 값을 가져올수 있었습니다.

그 외에도 공식문서에는 기존의 sink 방법외에 두가지를 더 제시하고 있습니다.

1. `sink`
2. `subscribe` 메소드를 이용
3. `assign(to: on:)`



#### `subscribe` 를 통한 구독

```swift
class GakiSubscriber: Subscriber {
    typealias Input = String
    typealias Failure = Never
    // 1.
    func receive(subscription: Subscription) {
        print("구독 시작")
        subscription.request(.unlimited)
    }
    // 2.
    func receive(_ input: String) -> Subscribers.Demand {
        print("전달받은 데이터:",input)
        return .none
    }
    // 3.
    func receive(completion: Subscribers.Completion<Failure>) {
        print("전달 완료", completion)
    }
}

publisher.subscribe(GakiSubscriber())

/*
구독 시작
전달받은 데이터 Gaki
전달 완료 finished
*/
```

하나하나 살펴보겠습니다.

1. `receive(subscription:)` : 여기서 `Subscription` 타입의 프로토콜 파라메터 타입 입니다. 이 Subscription은 `publisher` 와 `subscriber` 간의 연결을 나타내는 **구독** 그 자체라고 볼 수 있습니다. 이  프로토콜 타입의 파라메터 `subscription` 를 이용하여 `publisher` 에게 item을 요청하게 됩니다.

   Subscription 프로토콜에는 `request(_ demand:)` 라는 메서드가 존재합니다.

   <img width="413" alt="image" src="https://user-images.githubusercontent.com/33486820/80194166-f12f3880-8654-11ea-99b8-0ff698b77a8d.png">

   이 메서드를 통해 `subscriber` 로 부터 `publisher`에게 요청 할 item의 수를 지정할 수 있습니다.

   `Subscribers.Demand` 에는 아래의 3가지의 수제한을 둘 수 있습니다.

   - `unlimited`: 무제한
   - `max`: 최대 개수 제한
   - `none`(no elements): `max(0)` 과 동일

   그러므로 위의 예제에서는 개수제한이 없이 무제한으로  구독할 수 있습니다.



#### 그럼 시퀀스를 Publishing 해보자

```swift
class GakiSubscriber: Subscriber {
    typealias Input = String
    typealias Failure = Never
    // 1.
    func receive(subscription: Subscription) {
        print("구독 시작")
        subscription.request(.max(2))
    }
    // 2.
    func receive(_ input: String) -> Subscribers.Demand {
        print("전달받은 데이터:",input)
        
        switch input {
        case "Gaki":
            return .none
        default:
            return .max(1)
        }
    }
    
    func receive(completion: Subscribers.Completion<Failure>) {
        print("전달 완료", completion)
    }
}


let publishers = ["Gaki", "Chris Martin", "Jonny Buckland", "Will Champion", "Guy Berryman"].publisher

publishers.subscribe(GakiSubscriber())

/*
구독 시작
전달받은 데이터: Gaki
전달받은 데이터: Chris Martin
전달받은 데이터: Jonny Buckland
전달받은 데이터: Will Champion
전달받은 데이터: Guy Berryman
전달 완료 finished
*/
```



처음 1번 에서 구독을 시작하고 request의 최대치를 2로 정했습니다. 그런다음 데이터를 전달 받아 하나씩 출력하는 2번에서

input이 "Gaki"이면 `.none` 을 통해 더이상 구독하지 않지만 이전에 1번에서 구독을 시작할때 max를 2로했기 때문에 그다음 item을 구독 그리고 default기 때문에 하나씩 계속해서 구독할 수 있게 되어 결과적으로 모두 출력할 수 있게 됩니다.



### Publisher 와 Subscriber 통신

![image](https://user-images.githubusercontent.com/33486820/80194479-6438af00-8655-11ea-899b-712939bed8f0.png)



오늘은 Combine의 Publisher와 Subscriber의 개념에 대해 알아보았습니다. 기본적으로 RxSwift 와 구조가 많이 비슷하고 그 원리 또한 많이 비슷한거 같습니다. 사용방법이나 활용도 많이 중요하지만 비동기 제어에 대해 애플이 궁극적으로 추구하는것이 어떤것인지 공부하면서 배워야겠습니다.

Combine에 대해서 계속해서 글을 작성하도록 하겠습니다!!





### Reference

- [ZeddiOS-Combine](https://zeddios.tistory.com/966)
- [https://kka7.tistory.com/163](https://kka7.tistory.com/163)
- [해리의유목코딩-Combine입문가하기1-구성요소](https://medium.com/harrythegreat/swift-combine-입문하기-가이드-1-525ccb94af57)

