---
layout: post
current: post
navigation: True
title: Facade Pattern
date: 2020-04-01 11:20:00
tags: iOS DesignPattern
class: post-template
subclass: 'post tag-fables'
author: gaki
---  


## Facade Pattern 



디자인 패턴에는 여러가지들이 존재합니다. 이전에는 Delegate pattern에 대해서 정리를 해보았는데 이번엔 **퍼사드(Facade)패턴** 에대해서 공부해볼려고 합니다.

#### Facade란?

<img width="495" alt="image" src="https://user-images.githubusercontent.com/33486820/78085066-d1f61000-73f4-11ea-99ca-d6b94af7f9f2.png">

예를 들어 스마트 티비를 시청하는 경우를 생각해봅시다.

여러분들은 스마트티비를 보기위해 스마트 리모컨을 들고 쇼파에 앉는다고 상상해보세요

티비를 보면서 같이 먹을 치킨을 시키기 위해 주문을 할려고합니다.

1. 배달어플에 접속
2. 원하는 치킨 주문
3. 방의 불빛 조절
4. 티비 전원을 켠다
5. 셋톱 박스 전원을 켠다
6. 셋톱 박스 컴포넌트로 연결한다
7. 넷플릭스, 왓챠플레이 등 스트리밍 서비스에 접근한다.
8. 시청할 영상콘텐츠를 고른다
9. 음향을 서라운드  음향모드로 전환한다.
10. 시청할 영상콘텐츠를 재생한다. 

...

 이 밖에도 더 있습니다. 

- 만약 원하는 영상을 다보고 나면 어떻게 종료를 할지? 종료는 방금의 순서의 역순으로 진행해야하는건지?
- 스트리밍서비스말고 그냥 공중파 방송을 볼때도 이렇게 복잡한지?
- 만약 시스템이 업그레이드가 되어서 작동방법이 변경이 되는 경우는 어떻게 해야하는지?



그럼 어떻게 이런 복잡한 일을 간단하게 처리하고 편안하게 티비를 보게 해줄 수 있는지 **퍼사드**  패턴을 통해 티비시청을 즐길 수 있을지 알아보겠습니다.



이렇게 복잡한 시스템의 경우 퍼사드 패턴을 사용하면 쉽게 해결할 수 있습니다. 퍼사드 패턴을 쓰면 훨씬 쓰기 쉬운 인터페이스를 제공하는 퍼사드 클래스를 구현함으로써 복잡한 시스템을 훨씬 쉽게 사용할 수 있게 합니다.



#### Facade Pattern 정의  



위키피디아에는 이렇게 설명이 되어있습니다. 

> "퍼사드 패턴은 객체 지향 프로그래밍에서 흔히 사용되는 소프트웨어 디자인 패턴이다. 건축물의 외형과 유사하게 퍼사드는 복잡한 시스템 뒤단은 숨기고 간단한 인터페이스를 제공하는 객체이다"



건축물에 비유하였는데 쉽게 말해 우리는 건축물의 복잡한 **"내부 구조는 알 필요 없다"** 입니다. 즉 우리는 새로운 건물을 들어갈때마다 건축물의 내부 도면과 내부 구조를 파악하고 들어가지 않아도 엘리베이터나 계단을 통해 건축물을 오를 수 있다는 것 입니다.



<img width="568" alt="image" src="https://user-images.githubusercontent.com/33486820/78087761-0ff73200-73fd-11ea-881f-c9b6e4a5ae29.png">

위의 다이어그램에서 볼 수 있듯이 LogicA,B,C 클래스와 로직을 구분하는 Facade 클래스가 있습니다. 결과적으로 클라이언트는 다른 클래스(A,B,C)에서 구현되는 몇가지 메서드를 사용하기위해 단순하게 파사드 클래의 메서드만 호출하면 된다는 것입니다.

클라이언트가 만약 LogicA,B,C의 시스템에 접근하게 되면 구현, 변경, 테스트 그리고 재사용성이 굉장이 힘들어집니다. 이러한 복잡한 내부 구조는 퍼사드 클래스에 숨기고 클라이언트는 단순하게 퍼사드만 가지고 있고 그 퍼사드가 제공하는 인터페이스를 사용하기만 하면 됩니다.





![image](https://user-images.githubusercontent.com/33486820/78087268-7e3af500-73fb-11ea-96aa-f0eaeb6b9d93.png)

 

1. HomeTheater 시스템용 퍼사드를 작성합니다. `watchSteaming()` 같이 기본적인 메소드만 들어있는 클래스를 새로 작성합니다.
2. 퍼사드 클래스에서는 HomeTheater 구성요소들을 하나의 서브시스템 요소로 간주하고 , `watchSteaming()` 메소드에서는 서브시스템의 메소드들을 호출하여 필요한 작업을 처리합니다.
3. 클라이언트 코드에서는 버시스템이 아닌 HomeTheaterFacade에 있는 메서드를 호출합니다. 즉 `watchStreaming()` 메서드만 호출하면 알아서 방의 전등, 셋톱박스, 배달앱 치킨 주문 등이 준비가 됩니다.



이 처럼 사용자(client)는 단순하게 "스트리밍볼꺼야!" 하면 내부적으로 어떤 구조인지 몰라도 `watchStreaming()` 를 호출만 하면 알아서 그에 맞는 세팅을 해주는 것이라 볼 수 있습니다. 만약 퍼스드를 쓰더라도 서브시스템에는 여전히 접근하여 클래스의 고급기능이 필요하여 추가가 할 경우 마음대로 변경할 수 있는 이점이 있습니다.



> Q) 퍼사드에서 서브



이제 퍼사드 패턴의 기본적인 정의를 알아 보았습니다. 그럼 iOS에서는 퍼사드 패턴을 어떻게 사용하는지 알아 보겠습니다.



### iOS의 Facade Pattern



사실 iOS에서 파사드 패턴은 쉽게 볼 수 있는 것 같습니다. 기본적으로 Cocoapod 라이브러리 자체가 어떻게 보면 일종의 퍼사드가 아닐까 생각이 듭니다.



공유하기 기능을 예를 들어 보겠습니다. **UIActivityViewController** 통해 공유할 수 있는 기능은 클라이언트의 입장에서 단순하게 로컬에 저장, 에어드랍, 복사, 프린트 등 Share에서 제공하는 Facde 클래스의 메서드를 통해 기능을 실행한다고 볼 수 있습니다.



<img width='300' src='https://user-images.githubusercontent.com/33486820/78090902-69b02a00-7406-11ea-84a7-71d8f807891b.png'>



### Share Facade 구현

```swift
class ShareFacade {
    public unowned var inputDrawing: UIView
    public unowned var parentViewController: UIViewController
    public var imageRenderer: ImageRenderer()
    
    init(entireDrawing: UIView, inputDrawing: UIView, parentViewController: UIViewController) { 
    	// ...

    }

    func presentShareActivity() { 
        // ...
        parentViewController.present( ... )
    }
    
    private func convert() -> UIImage { 
        // ...
        let image = imageRenderer.convert( ... )
        // ...
        
        return image
    }
}
```

<img width='300' src='https://user-images.githubusercontent.com/33486820/78092376-95cdaa00-740a-11ea-9411-0c2707c3b820.png'>

```swift
public class ViewController: UIViewController {
    lazy var shareFacade: ShareFacade = ShareFacade(entireDrawing: drawViewContainer, inputDrawing: inputDrawView, parentViewController: self)
    
    ...
    
	@IBAction public func sharePressed(_ sender: Any) {
		shareFacade.presentShareActivity()
	}
}
```

<img width='300' src='https://user-images.githubusercontent.com/33486820/78092407-a9791080-740a-11ea-8bb2-7c146c009d75.png'>



## Reference

- [Head First Design Patterns](https://book.naver.com/bookdb/book_detail.nhn?bid=1882446)
- https://ehdrjsdlzzzz.github.io/2019/04/09/Design-Pattern-Facade/
