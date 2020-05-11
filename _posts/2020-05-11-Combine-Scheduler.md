---
layout: post
current: post
navigation: True
title: SwiftUI Combine(3) - Scheduler
date: 2020-05-11 00:25:00
tags: SwiftUI
class: post-template
subclass: 'post tag-fables'
author: gaki
---  

## Combine Scheduler



오늘은 Combine의 Scheduler를 정리해보려고 합니다. 운영체제에서 Scheduler의 개념은 **실행의 순서를 정하는** 역할로 많이 소개가 되어있습니다. Combine에서도 Scheduler의 역할은 비슷하다고 할 수 있습니다.



### Scheduler란?

Scheduler는 Publisher의 작업들이 **언제 그리고 어떻게** 수행될지를 결정하기 위한 객체입니다. 작업들을 적절한 시간에 수행이 되어야 하고 또 어 떤 Thread에서 수행되어야 하는지를 제어할 수 있습니다. 

비동기 처리에 있어서 적절한 시기에 스레드 사용을 하는 것은 필수적입니다. 그래서 Apple에서는 GCD(Grand Central Dispatch)를 제공하고 있습니다. 

```swift
DispatchQueue.global().async {
    //Code
}

DispatchQueue.main.async {
    //Code
}
```

이 밖에도 GCD를 추상화 시켜서 wrapping 시켜서 제공하는 OpearionQueue 도 존재합니다.

Apple에서 제공하는 Scheduler 타입객체는 DispatchQueue, OperationQueue, RunLoop등으로 이것들을 Combine에서도 그대로 사용가능합니다.



### Combine Scheduler 의 종류

Combine에는 Schduler를 switching 하여 명시적으로 지정할 수 있는 방법이 두가지 있습니다.

`receive(on:)` 과 `subscribe(on:)` 두가지 방식으로 지정할 수 있습니다.



<img width="660" alt="image" src="https://user-images.githubusercontent.com/33486820/81501224-26ed4600-9312-11ea-9e72-52b685fb3ef0.png">

<img width="659" alt="image" src="https://user-images.githubusercontent.com/33486820/81501295-9cf1ad00-9312-11ea-8747-99a9fe028662.png">

`receive(on:)` 의 정의를 잠깐 보면 매개변수로 Scheduler 프로토콜 타입을 받고 있습니다. 이 Scheduler 프로토콜을 iOS13이상 부터는 DispatchQueue, OperationQueue, RunLoop, ImmediateSchdeuler들이 채택하고 있기 때문에 매개변수로서 사용할 수 있습니다.





#### `receive(on:)` 으로 제어

publisher로 부터 element를 수신할 scheduler를 지정하는 역할을 합니다. downstream의 실행 컨텍스트를 변경하는 역할을 합니다.

아래의 예제를 우선 보겠습니다.

```swift
let publisher = ["Gaki"].publisher

publisher
    .map { _ in print(Thread.isMainThread) }    // true
    .receive(on: DispatchQueue.global())
    .map { print(Thread.isMainThread) }         // false
    .sink { print(Thread.isMainThread) }        // false

```



우선 Scheduler의 경우 element가 최초로 생성된 스레드와 동일한 스레드를 사용하게 됩니다. 처음 element가 생성된 스레드 가 MainThread에서 생성되었기 때문에 true가 나오게 됩니다. 그 후 `receive` 를 통해 `DispatchQueue.global()`로 변경한 후에는 백그라운드 스레드에서 동작하기 때문에 false가 나오는 것을 확인 할 수 있습니다.



#### `subscribe(on:)` 으로 제어

receive와 반대로 **upstream의 실행 컨텍스트를 변경합니다**. 즉 element의 소스를 구독하는 시점에서 실행 컨텍스를 변경 할 수 있습니다.

아래의 예제의 경우 MainThread에서 실행되기 때문에 True가 나오게 됩니다. 초기의 Upstream의 실행컨텍스트가 MainThread에서 아무런 변경사항을 주지 않았기 때문입니다.

```swift
let publisher = CurrentValueSubject<String, Never>("Gaki")

publisher
    .sink { _ in print("Sink \(Thread.isMainThread)") } // Sink true
```

위 예제를 아래와 같이 변경 시켜 보겠습니다.

```swift
let publisher = CurrentValueSubject<String, Never>("Gaki")

publisher
    .subscribe(on: DispatchQueue.global())
    .sink { _ in print("Sink \(Thread.isMainThread)") } // Sink false

```

위에서 추가된 구문은 `DispatchQueue.global()` 를 `subscribe(on:)` 에 추가를 했습니다. 그러면 구독 자체가 `DispatchQueue.global()` 에서 일어나게 되고, 즉 upstream을 백그라운드에서 실행하게 됩니다. 그런 다음 output downstream이 동일한 queue에 전달이 되어 sink에서 false를 리턴하게 됩니다.

이 모든 전제는 위에서도 언급한 **Scheduler는 element가 생성된 스레드와 동일한 스레드를 사용** 하기 때문입니다.



다른 예제를 통해 조금더 알아 보겠습니다.

```swift
func printValue(_ value: String) {
    Just(value)
        .handleEvents(receiveOutput: { _ in print(Thread.isMainThread) }) // 1
        .receive(on: DispatchQueue.global())
        .handleEvents(receiveOutput: { _ in print(Thread.isMainThread) }) // 2
        .receive(on: DispatchQueue.main)
        .handleEvents(receiveOutput: { _ in print(Thread.isMainThread) }) // 3
        .receive(on: DispatchQueue.global())
        .handleEvents(receiveOutput: { _ in print(Thread.isMainThread) }) // 4
        .subscribe(on: DispatchQueue.main)
        .handleEvents(receiveOutput: { _ in print(Thread.isMainThread) }) // 5
        .sink { value in
            print(value) // 출력
    }
}
DispatchQueue.global().async {
    printValue("Gaki")
}
/*
true
false
true
false
false
Gaki
*/
```

handleEvent를 통해 이벤트를 제어하는 구간을 5가지로 나누었습니다.

우선 `printValue` 함수를 `gloabal()`, 즉 백그라운드 스레드에서 실행을 시켰는데 첫번째 출력이 MainThread에서 실행이 되었다고 true가 반환이 되었습니다.

이 경우 `subscribe(on:)` 에 의해 이렇게 출력이 되었다고 볼 수 있습니다. "왜? subscribe는 4번 이후에 호출이 되었는데?"라고 생각할 수 있습니다. 하지만 이는 호출 시점이 상관이 없이 호출이 되면 우선적으로 upstream에 영향을 주게 됩니다.

그렇게 때문에 main에서 실행이 되고 그다음 `receive(on:)` 에서 (1) global -> (2) main -> (3) global 이 되고 4번 이후 subscribe에서 main으로 컨텍스트를 변경 시켰지만 이는 다시 main queue로 돌아오지 않기 때문에 여전히 global 에 의해 false를 출력하게 됩니다. 그리고 최종적으로 5번에서도 main이 아니기 때문에 false를 출력합니다.



#### 적절하게 사용하는 것이 중요

`receive(on:)` 의 경우 Publisher의 특정 작업의 스케쥴러를 변경할 수 있기 때문에 여러번 사용하여 downstream을 제어하게 됩니다. 하지만 `subscribe(on:)` 의 경우 upstream의 스케쥴러를 바꿔버리기 때문에 한번만 사용을 하여서 쓰레드를 관리하는 것이 좋습니다.

