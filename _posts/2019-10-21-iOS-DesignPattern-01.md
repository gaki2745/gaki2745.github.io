---
layout: post
title: Delegate Pattern
subtitle: ""
categories: ios
tags: designPattern
comments: false
---

## iOS Delegate Pattern


Delegate Pattern은 iOS에서만 사용되는 개념이 아니다. 하지만 iOS에서 가장 많이 사용되는 디자인 패턴 중에 하나이므로 그 중요성은 익히알고있다. delegate 라는 용어의 정의 부터 짚고 넘어가보자.

> Delegate : **"대리자", "위임자"** 

Delegate Pattern은 객체지향 프로그래밍에서 하나의 객체가 모든 일을 처리하는 것이 아니라 처리해야할 일 중 일부를 다른 객체에 **위임** 하는 것이다.

객체가 해야할 일 들의 위임 즉, 어떤 대화를 하는 방법 중 하나라고 생각하면된다.

기존의 iOS 에서는 객체들간의 대화를 하는 방법에는 여러가지 방법이 있다.

1. Notification
2. KVO(Key-Value Observing)
3. Completion Handlers / CallBack (클로저를 사용)
4. Target-Action

이런 방법 들 가운데 Delegate Pattern은 어떤 형식으로 되는지 기본부터 응용까지 알아보자.



## Delegate Pattern의 구성

Delegate Pattern은 **하나의 프로토콜 과 두가지의 객체** 로 구성이된다.

- Protocol : **해야할 작업(task)의 목록**
- Sender : **task 처리를 요청(위임)할 객체**
- Receiver : **위임받은 task를 처리하는 객체**

### Delegation 적용 과정

1. **요구 사항을 파악**하여  Delegate Protocol을 구성한다. 아래의 구조와 같이 Delegate Protocol을 작성하여 해야할 작업을 함수로 구성한다.

   ```swift
   protocol SomeDelegate {
   		// task 목록
   		func task1()
   		func task2()
   		...
   }
   ```

2. **Delegation 채택**을 위해 Sender의 내부에서는 `SomeDelegate` 타입의 프로퍼티 즉 delegate 객체를 선언한다.

   ```swift
   var delegate: SomeDelegate?
   ```

3. Receiver 에서 Sender 타입의 객체를 선언한다. 그리고 **Delegate 객체(일을 시키는 객체)와 연결**한다

   ```swift
   // Receiver 에서
   sender.delegate = self
   ```

4. **요구사항 task 를 구현** 한다. Receiver 에서 SomeDelegate 프로토콜을 채택하여 내부의 해야할 task 함수들을 구현해야한다.

> 참고)  sender.delegate = self 의 의미
>
> 의미 적으로는 Sender객체의 delegate 즉 할일 을 Receiver 의 self(자기자신) 에게 위임 한다는 의미 이다.  이렇게 self 로 가능한 것은 프로토콜을 Receiver 에서 상속받고 구현하게 되면 상속받은 프로토콜로 형변환이 가능해진다. 그렇기 떄문에 Sender의 delegate 프로퍼티 즉 `SomeDelegate` 프로토콜 에 Receiver 의 self 를 대입하면서 위임이 시작 되는 것이다.



##### Delegate Pattern 의 Delegation 적용 모식도 

<img width="975" alt="image" src="https://user-images.githubusercontent.com/33486820/67159651-4dea2780-f382-11e9-96fd-1386bd5af16b.png">



## Delegate Pattern 구현 



자 예를 들어 웹툰 앱이나 쇼핑 앱에서 댓글이나 평가를 할수 있게 하는 UIView를 구현한다고 생각해보자. 우선 요구 사항이 댓글을 입력하고 버튼을 눌렀을때 댓글이 입력이 되거나 서버에 기록이 되어야 하는데 과연 이 댓글 뷰(CommnetView)는 어떤 ViewController 에서 입력이 되었는가? 에 대해 알 수 있어야한다

우선 댓글 입력 기능은 한 영역에서 제공되는 기능이 아니라 웹툰 앱을 예를 들어 생각해보면 요일별 웹툰, 유저 도전작, 금일 베트스, 금주 베스트,,, 등등 다양한 영역에서 댓글과 평가를 줄 수 있는 기능이 제공 되어야한다

그럼 여기서 우리는 **중복 되는 기능이 있고 이는 프로토콜로 명시해서 처리해보자** 라고 생각할 수 있다. 하지만 앞에서 언급 했듯이 요구사항에 **어떤 ViewController 에서 댓글을 입력했는지 알고 싶다** 가 있다. 이에 우리는 Delegate 패턴을 이용해서 해결 할 수 있다

여기서 **각 ViewController 가 대신 처리해줄 객체(Receiver)** , **CommentView(댓글 뷰) 가 댓글 입력을 처리하라고 시키는 객체(Sender)** 가 된다.

다음 단계로 Delegate Pattern 을 주입해서 간단하게 예제를 작성해겠습니다.



##### Delegate Protocol 선언

```swift
protocol CommentViewDelegate: class {
  	// task1
    func touchUpCommentButton()		// 댓글 입력 버튼을 눌렀을때 처리해야할 task
  	// task2
  	func touchUpStarRateButtom()	// 별점을 눌렀을 때 처리해야할 task
}

```

`CommentViewDelegate` 프로토콜을 선언했고 내부에는 추후 Receiver 가 구현해야할 task를 명시한다.

##### Sender : 댓글뷰(CommentView) 선언

```swift
class CommentView: UIView {
    // Delegate 객체 선언 -> Receiver에서 위임 받은 객체를 저장해 둘 프로퍼티
    weak var delegate: CommentViewDelegate?	
  	
    var commentButton: UIButton?
  	var starRateButton: UIButton?
    var commentTextField: UITextField?

    public override init(frame: CGRect) {
        super.init(frame: frame)
        configure()
    }
  
    public required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
        configure()
    }
  	// CommentView 구성
    func configure() {
        commentButton = UIButton(type: .system)
        if let commentButton = commentButton {
          // button 값 설정
          commentButton.addTarget(self, action: #selector(tapCommentButton), for: .touchUpInside)
          self.addSubview(commentButton)
        }
      	starRateButton = UIButton(type: .system)
      	if let starRateButton = starRateButton {
           // button 값 설정
          starRateButton.addTarget(self, action: #selector(tapStarRateButton), for: .touchUpInside)
            self.addSubview(starRateButton)
        }
        commentTextField = UITextField()
        if let textField = commentTextField {
    			// textField 값 설정          
            self.addSubview(textField)
        }
    }
    // 각 button을 터치 시 delegate의 메서드가 호출되게 한다.
    @objc func tapCommentButton() {
        delegate?.touchUpCommentButton()
    }
  
  	@objc func tapStarRateButton() {
      delegate?.touchUpStarRateButton()
    }
  
}
```

여기서 핵심인 것은 CommentView 내부에 CommentViewDelegate 타입의 프로퍼티가 존재한다는 것이다. 이 프로퍼티로 나중에 Receiver 즉 ViewController 측에서 대신해서 처리해줄 객체를 저장할 수 있도록 하는 프로퍼티다.

```swift
weak var delegate: CommentViewDelegate?	
```



##### Receiver: BestWebToonViewController 구현 

베스트 웹툰을 모아서 보여주는 ViewController 라고 가정하자! 여기서는 앞에서 선언한 CommentView 와 CommentViewDelegate 를 사용하여 Receiver 의 기능을 구현해 보겠습니다.

```swift
import UIKit

class BestWebToonViewController: UIViewController {
    // Sender의 객체 즉 CommentView 타입 객체 선언
    var commentView: CommentView?

    override func viewDidLoad() {
        super.viewDidLoad()
      	
        commentView = CommentView(frame: CGRect(origin: .zero, size: CGSize(width: 300, height: 200)))
        if let commentView = commentView {
						// commentView 값(위치, 옵션, 속성..) 설정
          
          	// 핵심 : Delegate 객체 연결
            commentView.delegate = self
        }
    }


}
// CommentViewDelegate 프로토콜을 상속하여 명시된 task 를 구체적으로 구현 하는 작업
extension BestWebToonViewController: CommentViewDelegate {
    func touchUpCommentButton() {
        print("commentButton Touched!")
    }
    func touchUpStarRateButton() {
        print("starRateButton Touched!")
    }

}
```

commentView를 해당 ViewController 에서 띄워주기 위해 객체를 선언했고 또 선언 과 동시에 내부에 delegate 프러퍼티에 "내가(BestWebToonViewController) 이제 이 객체를 위임 받아서 작업하겠다" 하고 Delegate 객체 연결 작업을 해준다.

```swift
commentView.delegate = self
```

그리고 CommentViewDelegate 프로토콜을 상속하여 이제 위임 받은 delegate 객체가 할 일들에 대해 상세하게 구현을 해준다.



## Delegate 와 Retain Cycle 

우선 결론부터 말하면 Delegate Pattern은  Retain Cycle 이 발생할 수 있다.

iOS의 메모리관리는 ARC가 전적으로 관리하고 있다. 기본적으로 객체의 참조가 더이상 존재하지 않으면 ARC가 메모리에서 해지한다.(ARC에 관한 내용은 [gaki블로그 - iOS ARC](https://gaki2745.github.io/Swift-ARC) 를 참고) 

Delegate Pattern 에서는 항상 참조를 할때 weak 키워드를 사용해야한다.

위의 예제에서도 weak 키워드를 붙여 delegate 객체를 선언했다 이 delegate 객체를 CommentView(Sender) 내부 객체도 참조를 하지만 ViewController(Receiver) 도 self 를 통해 참조를 하기 때문에 strong 하게 선언을 하게 되면 Retain Cycle 이 발생하게 되고 이는 메모리의누수를 초례할 수 있다.

> 결론 : Delegate Pattern 은 Retain Cycle 이 발생할 수 있기 때문에 weak 키워드르 붙여 약한 참조를 발생 시키게 해야한다!



## Reference

- [마기의 개발블로그 - iOS Delegate 패턴에 대해서 알아보기](https://magi82.github.io/ios-delegate/)
- [Swift 5.1 Delegate Pattern](https://medium.com/@nimjea/delegation-pattern-in-swift-4-2-f6aca61f4bf5)
- [Avoiding Reatin Cycle  in iOS](https://medium.com/meshstudio/avoiding-retain-cycles-in-ios-301fd522b6f8)



