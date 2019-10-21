---
layout: post
current: post
navigation: True
title: iOS Static Home Quick Action
date: 2019-10-21 23:18:00
tags: Swift, iOS
class: post-template
subclass: 'post tag-fables'
author: gaki
---  

## 3DTouch? Haptic Touch?


![image](https://user-images.githubusercontent.com/33486820/67206943-8efb3e00-f44d-11e9-8f12-6535395fb986.png)

자료영상 : https://www.youtube.com/watch?v=ivWChvW2eCU

Apple은 iPhone11 과 동시에 발표된 새로운 모델들에 3DTouch 대신 **햅틱 터치** 를 탑재해서 출시가 되었다. 예전부터 3Dtouch가 일반 long touch와의 차이가 모호하다는 비판이 많았고 내부에 들어가는 압력 감지 센서때문에 디바이스의 크기가 두꺼워지고 가격도 늘어난다는 단점 때문에 뺀게아닌게 생각이 든다. 

아무튼 3D touch 가 Haptic Touch 로 변경이 되었고 소프트웨어 적으로도 iOS 13 버전 부터는 3D 터치가 제공되지 않고 Long Touch로 아래의 기능들을 수행할 수 있다.

<img width="983" alt="image" src="https://user-images.githubusercontent.com/33486820/67207420-89eabe80-f44e-11e9-8e21-044bce07dfd6.png">

그러면 오늘은 대체된 Haptic Touch로 **Custom Quick Action** 의 구현 방법에 대해서 익히도록 하겠습니다.



## 홈 화면 Static Quick Action

홈화면 에서 Long Touch 로 앱들의 ShortCut을 보여줘서 빠르게 앱의 원하는 기능으로 접근할 수 있는 기능을 **Static Home Screen Quick a Actions** 라고 한다.

기존에 Apple 에서 제공하고있는 앱들은 위와 같이 다양한 shortcut 기능을 제공하고 있다. 이것을 구현할려면 우선 프로젝트의 `info.plist` 에 **UIApplicationShortcutItems** 을 추가하여 Quick Action을 설정해주어야한다.

##### UIApplicationShortcutItems

<img width="784" alt="image" src="https://user-images.githubusercontent.com/33486820/67208441-82c4b000-f450-11e9-951d-019a52ae0c3b.png">

UIApplicationShortcutItems 배열 안에는 Dictionory 값이 들어가게 되는데 이것이 바로 Static Quick Action 하나에 설정값에 해당한다. 위 순서대로 item0, item1, item3 가 차례대로 Long Touch 시 홈 화면에 표시된다.

우선 여기서는 필수적으로 작성해야하는 것이 존재한다

##### 필수 작성 요소

- **UIApplicationShortcutItemType** : 사용자가 해당 Quick Action을 호출 할 때, 엡에 전달되는 문자열을 사용하여 Quick Action작업을 타입으로 분류한 다음, 수신한 작업 타입을 명확하게 구분 할 수 있다.
- **UIApplicationShortcutitemTitle** : Quick Action의 제목으로, 홈 화면에서 사용자에게 표시되는 문자열이다. 타이틀이 너무 길거나 UIApplicationShortcutItemSubtitle을 지정하지 않은 경우, 타이틀이 두줄로 표시된다. 

##### 그밖의 옵션값 4개

- **UIApplicationShortcutItemSubtitle** : 홈 화면에서, 사용자에게 해당 타이틀 문자열 바로 아래에 표시되는 선택적(optional) 문자열(있어도 되고 없어도 된다.) quick action을 위한 Subtitle을 지정하면 시스템은 타이틀의 길이에 관계없이 한 줄에 quick action 타이틀을 표시한다.

- **UIApplicationShortcutItemIconType** : 시스템 제공 라이브러리의 아이콘 타입을 지정하는 선택적(optional)문자열. UIApplicationShortcutIcon class Reference의 UIApplicationShortcutIconType enum을 참고. 아이콘은 quick action의 타이틀과 함께 홈 화면에서 표시된다. 

- **UIApplicationShortcutItemIconFile** : 앱 번들에서 사용할 아이콘, 이미지 또는 Asset 카탈로그에서 이미지 이름을 지정하는 선택적 문자열이다. 홈 화면에서 quick action 타이틀 앞에 아이콘이 표시된다. 아이콘은 OS Human Interface Guidelines에 따라, 정사각형, 단일색상 및 35x35pt여야 한다. 

- **UIApplicationShortcutItemUserInfo** : 선택적(optional)인 앱 정의 사전(dictionary). 이 사전의 한가지 용도는 UIApplicationShortcutItem class Reference에 있는 "빠른 실행을 위한 응용 프로그램 시작 및 응용 프로그램 업데이트 고려 사항" 섹션에서 설명한대로, 응용프로그램 버전 정보를 제공하는 것이다. 



자 이제 `info.plist` 에 필요한 작업은 모두 마쳤으니 Quick Action을 제어해보자.



![image](https://user-images.githubusercontent.com/33486820/67210395-fb793b80-f453-11e9-9f92-d5690b778a1c.png)



Naver 앱과 같이 보통 해당 Quick Action을 통해 앱에 접근하게 되면 해당 영역으로 가게된다. 예를 들어 내가 "음성검색" 을 터치했을 떄는 UIApplicationShortcutItemType에 대한 정보가 앱에 전달이 되어 실제 음성검색영역으로 시스템이 곧바로 앱을 시작하거나 다시시작한다.

이는 UIKit의 AppDelegate가 **`application:performActionForShortcutItem:completionHandler` 메서드를 호출한다** .

```swift
  func application(_ application: UIApplication, performActionFor shortcutItem: UIApplicationShortcutItem, completionHandler: @escaping (Bool) -> Void) {
			// code
    }
```

이 메서드가 Quick Action 을 하였을 때 호출되는 메서드임을 알고 간단한 실습을 통해 어떻게 구성을 하는지 알아보자



## Static Quick Action 구현

우선 Naver 앱의 경우를 살펴보면 Quick Action 으로 적지 않은 기능을 제공하기 때문에 이를 enum형으로 관리하는 것이 좋다.

```swift
enum ShortcutIdentifier: String {
    case First
    case Second
    case Third
    
    init?(fullType: String) {
        guard let last = fullType.components(separatedBy: ".").last else {
            return nil
        }
        self.init(rawValue: last)
    }
    
    var type: String {
        return Bundle.main.bundleIdentifier! + ".\(self.rawValue)"
    }
}
```

First, Second, Third 3개의 Quick Action Type이 있고 이 3개는 각각의 ViewController로 이동을 하기위해 enum으로 선언했다.

![image](https://user-images.githubusercontent.com/33486820/67212280-17caa780-f457-11e9-9c9e-16e65d7e6edb.png)

appDelegate.swift 에 아래의 코드를 추가하면 된다. `application:performActionForShortcutItem:completionHandler` 메서드가 escaping closure 에 Bool 타입을 매개변수로 받기 때문에 Bool 값 반환 함수 `handleShortCutItem` 메서드를 통해 내가 이동해야 할 ShortcuItem 에 대한 ViewController 로 Present 하는 코드를 작성했다.

```swift
 func application(_ application: UIApplication, performActionFor shortcutItem: UIApplicationShortcutItem, completionHandler: @escaping (Bool) -> Void) {
        let handledShorCutItem = handleShortCutItem(shortcutItem)
        completionHandler(handledShorCutItem)
    }
    
    // shortcut 제어  메서드
    func handleShortCutItem(_ shortcutItem: UIApplicationShortcutItem) -> Bool {
        var handled = false
        // 해당 shorcutItem 의 type이 내가 지정한 enum 타입에 있는지 체크
        guard let _ = ShortcutIdentifier(fullType: shortcutItem.type) else {
            return false
        }
        // shortCutType Type 가져온다 이는 위의 델리게이트 메서드가 알고 있다.
      	// info.plist 에 UIApplicationShortcutItemType 필수적으로 명시하는 이유도 이것 때문
        guard let shortCutType = shortcutItem.type as String? else {
            return false
        }
        
        let storyboard = UIStoryboard(name: "Main", bundle: nil)
        var vc: UIViewController
        // 전달 받은 shortCutType 에 맞는 ViewController 선정
        switch shortCutType {
        case ShortcutIdentifier.First.type:
            vc = storyboard.instantiateViewController(identifier: "AcsVC") as! AscViewController
            handled = true
        case ShortcutIdentifier.First.type:
            vc = storyboard.instantiateViewController(identifier: "BscVC") as! BscViewController
            handled = true
        case ShortcutIdentifier.First.type:
            vc = storyboard.instantiateViewController(identifier: "CscVC") as! CscViewController
            handled = true
        default:
            vc = UIViewController()
            break
        }
        
        // 선택된 view를 dispaly
        guard var presentedVC: UIViewController = window?.rootViewController else {
            return false
        }
        while presentedVC.presentedViewController != nil {
            presentedVC = presentedVC.presentedViewController!
        }
        
        if !presentedVC.isMember(of: vc.classForCoder) {
            presentedVC.present(vc, animated: true, completion: nil)
        }
        
        return handled
        
    }
}
```



<hr>



## Reference 

- [Apple Developer](https://developer.apple.com/documentation/uikit/uiapplicationdelegate)
- [allenwong/30DaysofSwift](https://github.com/allenwong/30DaysofSwift)



