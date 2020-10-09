---
layout: post
title: Swift OOP) DIP - 의존관계역전원칙(Dependency Inversion Principle)
subtitle: ""
categories: swift
tags: oop
comments: false
---



## DIP - 의존관계역전원칙(Dependency Inversion Principle)



- 코드에서 통상적으로 발견되는 특별한 문제는 다음과 같습니다
  - "상위 수준의 모듈이 하윗 수준의 모듈에 의존성을 가지는 경향"
  - "정책이 구체적인 것에 의존하는 경향"
- DIP의 목적
  -  모듈간의 의존관계를 끊는 방법을 제시
  - 변경이 다른 코드에 영향을 최소화 시킬 방법 제시



### 모듈의 의존관계

- 코드의 의존관계는 무조건 적으로 가지고 작성됨
- 의존관계 자체를 없앨수는 없다
- 하지만 의존관계를 잘 정리하지 않으면 코드는 경직성을 뛰게됩니다.
- 무엇이 무엇에 의존하느냐에 따라 다른 결과



### 서로 의존관계를 가지는 모듈

<img width="803" alt="image" src="https://user-images.githubusercontent.com/33486820/95596013-ce200600-0a87-11eb-98b5-37f0d01bf6c7.png">

상호의존적인 관계로 존재 하게 되는 경우입니다. 이런경우가 빈번하게 발생하고 있는 경우이기도 합니다.

만약 서로를 프로퍼티로 갖게 된다면 이는 순환참조에 의해서 한방향의 경우 weak로 선언해야하기도 합니다. 



### 의존관계가 Cycle을 만드는 경우

<img width="770" alt="image" src="https://user-images.githubusercontent.com/33486820/95596490-6a4a0d00-0a88-11eb-8c1a-b07846be3542.png">

이런 경우에는 어느 한클래스도 독자적으로 사용할 수 없습니다. 위의 경우에서 ClassB만 사용하고자 했을 뿐인데 불필요한 A,C도 같이 가지고 와야하는 경우가 생기게 됩니다.



### 단방향으로 흘러가는 의존관계

<img width="1157" alt="image" src="https://user-images.githubusercontent.com/33486820/95596786-c90f8680-0a88-11eb-98c2-60d11cf01fbc.png">

Cycle이 발생하는 경우에는 B를 사용하고자 했을때는 A와 C 모두 필요했다면 지금과 같은 구조에서는 단방향으로 의존관계가 흘러가기 때문에 B를 사용하고자 했을때 C를 가져와야하긴하나 A를 가져올 필요는 없기때문에 의존관계를 낮출수 있다고 할 수 있습니다

사실상 A, B, C 모듈이 모두 독립적으로 존재하여 의존관계가 전혀 없는게 최선이지만 현실적으로 의존관계가 아예없는 것이 불가능하기 때문에 위와 같은 구조인 단방향으로 흘러가는 의존관계를 만드는 것이 불필요한 의존도를 낮출 수 있게 됩니다.



### 의존관계의 역전이 필요한 경우

<img width="1164" alt="image" src="https://user-images.githubusercontent.com/33486820/95597492-aaf65600-0a89-11eb-9e6c-54df003fbf71.png">

단방향이긴 하나 간혹 C의 모듈이 역으로 B와 A를 의존해야하는 경우가 발생합니다. 이 때 우리는 DIP를 이용해서 해결할 수 있습니다



### 의존의 방향

의존의 방향은 **단방향**의 형태인 경우가 가장 좋다고 했습니다. 그렇다면 단방향일 때 방향을 결정짓는 요소는 어떤게 있을까요?

- 구체적 -> 추상적

- 구체적 -> 정책(추상적)
- 하위(구체적) -> 상위(추상적)

> "구체적인 부분에서 추상적인 방향으로 의존관계를 가져야 한다"

우리가 구현하는 구체적인 것이 추상적인 어떤 정책에 의존하는 관계가 되어야한다는 것입니다. 추상적인 것 자체가 구체적인 것에 의존한다는 것이 모순이겠죠?(~~개발스럽지않은표현이네요ㅜ~~)



### iOS의 대표적인 예



iOS에서 각 UIComponent를 생각해보겠습니다. UITableView, UICollectionView, UITextView, UIScrollView.. 등등 우리는 각 컴포넌트들을 구현할때를 따라가보겠습니다

UITableView에 대한 동작과 기능 그리고 UITableViewCell과 더불어 TableView에 담길 데이터 등의 구체적인 코드를 작성하게 되는데 이는 모두 **Delegate, DataSource** 를 통해 추상적인 부분이 구체화된 영역에 동작등을 위임하게 됩니다.

즉 UITableViewDelegate, UITableViewDataSource라는 인터페이스에 의해 의존하게 됩니다.

```swift
import UIKit

internal class ViewController: UIViewController {
    
    @IBOutlet private weak var tableView: UITableView?

    override func viewDidLoad() {
        super.viewDidLoad()
        
        // UITableView(추상화)에 관한 구체적인 부분에 동작을 위임
        tableView?.delegate = self
        tableView?.dataSource = self
    }
}

extension ViewController: UITableViewDelegate {
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        // 셀이 선택 되었을때의 동작 구체화
    }
}

extension ViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        // 셀의 Section의 개수에 관한 구체화
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        // 셀을 생성해서 반환하는 구체화
    }
}
```

마찬가지로 UITableView(추상적)은 내가 구현한 CustomCell(구체적)에 참조하지 않게됩니다. UITableViewDataSource라는 인터페이스를 통해 의존하고 있을 뿐입니다.



또 한가지 예는 CustomMapViewController라는 VC를 만들자고 가정해보겠습니다.

```swift

internal class CustomMapViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
    }
    
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
    }
    
    ...
}
```

CustomMapVC는 지도에 관련된 기능등을 구체화할 목적으로 생성했지만 기본적으로 UIViewController에 의존하고 있습니다.

그 이유는 구체적인 구현을 담당하는 CustomMapVC는 UIViewController의 **View Life Cycle**에 대한 정책결정을 따르고자 하기 위해 추상적인 개념인 정책결정을 하는 CustomMapViewController -> UIViewController의 의존관계를 가지게 됩니다. 



### DIP: 의존관계역전원칙

의존관계가 단방향인 경우에서 역으로 의존이 필요한 경우에 DIP를 사용한다고 했습니다. DIP의 개념과 목적에 대해서 한번 알아보겠습니다.

- DIP는 이처럼 소프트웨어  모듈들을 분리하는 원칙으로써, **상위계층(정책 결정) 이 하위계층(세부 사항 - 구체화)에 의존하는 전통적인 의존관계를 반전(역전)시커 상위계층이 하위계층의 구현으로 부터 독립되게 하는 것이 주 목적이라고 할 수 있습니다.**

- 상위 모듈은 하위모듈에 의존해서는 안되고, **상위모듈과 하위모듈 모두 추상화에 의존**해야한다.
- 추상화는 세부사항에 의존해서는 안된다. **세부사항이 추상화에 의존해야한다**



### 상속에서의 DIP

그럼 DIP는 앞에서 말한 인터페이스(프로토콜)을 사용하면 되는것인가? 그렇지는 않습니다.

상속에서는 **super class는 추상적인 개념, sub class는 구체적인 개념** 이라고 할 수 있습니다. 그렇다면 DIP원칙에 의해 super class 의 추상적인 로직은 sub class에 의존하지 않고 작성되어야 합니다. 즉 구체화를 하기 위해서는 sub class는 super class의 메소드를 override를 통해 구체화를 진행하게됩니다.



### Hollywood Principle(헐리우드 원칙)

> "먼저 연락하지 마세요. 저희가 연락드리겠습니다."(Don't call us, we will call you)

DIP를 설명할때 등장하는 원칙이 바로 헐리우드원칙입니다.

영화에 출연하 싶은 배우는 이력서를 기획사에 낼뿐입니다. 배우는 서류통과 이후 헐리우드 영화사에 요청하는데로 반응하여 행동할 수 밖에 없습니다.(먼저 연락해도 소용이 없다!)

이 처럼 저수준의 구성요소(배우)에서 고수준 구성요소(헐리우드 영화사)를 직접 호출(먼저 연락) 할 수 없게 하고, 고수준 구성요소가 저수준 구성요소를 직접 호출하는 것을 허용하게한다는 의미가 됩니다.

앞선 예로 UTableView의 경우도 우리가 동작들을 dataSource와 delegate를 통해 구체적인 부분이 추상적인 부분을 참조하게 하여 사이의 의존관계를 단방향으로 만들어 의존성 부패를 막게 됩니다. 이는 추상화인 부분 UITableView의 재활용성을 증가시킨 것입니다. 

우리가 개발하는 앱에서 이곳저곳에서 상황에 맞는 UITableView를 만들수 있는게 바로 DIP가 지켜지기 때문에 가능한 것입니다.



### 추상적인 개념, 구체적인 개념

기존의 프레임워크에서 지켜지는 DIP에 대해서 살펴보았습니다. 그럼 이제 우리가 직접 작성하는 코드에서 DIP원칙을 사용해야하는데 과연 추상적인 개념과 구체적인 개념이 각각 어떤 것을 말하는 건지를 파악해야합니다.

- 추상적인 부분(정책)
  - 자주 변경되지 않는 부분
  - 해당 로직의 뼈대
- 구체적인 부분(세부)
  - 자주변경되는 부분
  - 자꾸 추가되는 부분



### 의존성의 이행

의존성의 Cycle이 생기는 것을 이행이라고 할 수 있습니다. 불필요한 의존성이 같이 따라오는 것을 말합니다. 이는 결국 코드가 **스파게티**가 되어버릴 수 밖에 없습니다. 우리는 DIP를 통해 이런 Cycle을 없애고 의존의 흐름을 단방향을 바꾸어야할 필요가 있습니다.



### 각 레이어와 인터페이스

- 상위레이어(추상화, 정책, 클라이언트)
  - 인터페이스를 제공하는 쪽(인터페이스의 변화가 가장 적어야한다)
  - 변화가 비교적 적다
  - 재활용성이 높다
- 하위레이어
  - 자주변경이된다.

레이어라는 개념이 추가가 되었지만 더 넓은 범주의 개념이라 생각하시면 될거같습니다. 계속해서 강조하고 있는 것이 **구체화->추상화** 로의 의존관계를 유지하라 입니다. 이는 결국에는 코드의 재활용성 까지 이어질 수 있는 부분이라고 앞서 UITableView의 예제를 통해 확인을 했었습니다. 



### 인터페이스의 소유

그러면 인터페이스는 누가 소유(제공)을 하게 되는걸까요? [ISP(인터페이스 분리 원칙)](https://gaki2745.github.io/swift/2020/07/20/Swift-OOP-ISP/) 에서 설명한 클라이언트, 서버의 개념을 한번더 살펴보면

- 클라이언트: 객체를 사용하는 쪽
- 서버: 사용되는 쪽

이렇게 됩니다. 즉 구체화된 내용의 제공은 서버가 하게 되고 이를 클라이언트는 사용하는 쪽이 되는 것입니다

UITableViewController와 UITableView의 관계를 보면 TableView에 관련된 구체화된 내용을 UITableView에서 제공을 하고 있고 이를 UITableView가 사용하기 때문에 정리하자면 다음과 같습니다

- UITableViewController (서버): TableView의 구체화된 내용을 **제공**
- UITableView (클라이언트): TableView의 구체화를 **사용**

구체화된 내용을 사용하는 쪽인 클라이언트가 인터페이스를 소유하는 것이 됩니다.



### Reference

- [베어코드 - DIP](https://www.youtube.com/watch?v=cAhBEmpCNVQ)

