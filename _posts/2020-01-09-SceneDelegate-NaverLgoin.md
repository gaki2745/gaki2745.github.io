---
layout: post
current: post
navigation: True
title: iOS13 이상 프로젝트의 SceneDelegate를 AppDelegate로 이관 방법
date: 2020-01-09 13:18:00
tags: Swift iOS
class: post-template
subclass: 'post tag-fables'
author: gaki
---  

## iOS 13 SceneDelegate를 AppDelegate로 이관 방법

<br>



오류 경로는 네이버 로그인을 구현하면서 iOS13이상의 SceneDelegate관련한 문제가 발생하였습니다.

네이버 로그인 뿐만이 아니라 기존의 SNS연동 작업을 할때 iOS 13버전 이상에서 발생할 수 있는 이슈를 간단하게 기록하고 또 해결방법에는 어떤것이 있는지 알아보겠습니다.

Xcode에서 SwiftUI, iOS 13, iPadOs 13 등의 SDK가 포함되면서 **SceneDelegate** 의 개념이 추가 되었습니다.

 즉 Xcode11 버전이상에서 프로젝트를 만들게 되면 SceneDelegate가 추가가 됩니다.( iOS 13버전 부터 SceneDelegate가 자동으로 추가 됩니다) 



우선 iOS 네이버 로그인이 어떻게 동작하는지 간단하게 알아보고 가겠습니다.



<br>



### iOS 네이버 로그인 로직 순서  

<img width="1004" alt="image" src="https://user-images.githubusercontent.com/33486820/72035719-06699c80-32dc-11ea-9c01-448177d2ab37.png">



<br>

위에서 볼수 있듯이 대략적인 순서는 우선 앱이 백그라운드로 진입 후 네이버 앱 또는 네이버 사파리로 접근 후에 

[`application(_open:options:)` ](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623112-application) 가 호출이 되어 앱이 요청한 URL에서 리소스즉 네이버 로그인 인스턴스를 요청하게 됩니다.



그리고  **네이버 아이디로 로그인** 을 구현하기 위해 아래의 함수를 AppDelegate에 추가를 해주었습니다.

```swift
    func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
        // 네이버 로그인 인스턴스 가져오는 함수
        NaverThirdPartyLoginConnection.getSharedInstance()?.application(app, open: url, options: options)
        return true
    }
```

하지만 AppDelegate에 위 코드를 추가 후에 네이버 로그인을 테스트하면 앱이 백그라운드로 진입 후 위의 함수가 호출이 되지 않는 것을 확인할 수 있습니다.

<img width="584" alt="image" src="https://user-images.githubusercontent.com/33486820/72036569-88f35b80-32de-11ea-8e84-40b3d00cc75f.png">

> 이 문제는 iOS 13 이상부터 발생하는 이슈라고 하네요...



 iOS 13의 SceneDelegate가 있는 프로젝트의 경우 위의 [`application(_open:options:)` ](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623112-application) 함수가 호출되지 않는 것을 로그를 통해 확인할 수 있었습니다.

조금더 찾아본 결과 SceneDelegate의 [`scene(_:openURLContexts:)`](https://developer.apple.com/documentation/uikit/uiscenedelegate/3238059-scene) 에서 URL에 관련된 정보를 context로 관리하는 것을 알 수 있었습니다.



<br>



하지만 네이버 로그인의 경우 인스턴스를 가져오는 작업을 요청할때 아직까지는 SceneDelegate의 UIScene에 대응하는 함수가 제공되지 않는 것을 확인할 수 있었습니다.

그래서 해결책으로 iOS 13 이상의 프로젝트에서 SceneDelegate의 역할을 없애고 AppDelegate 이관하는 작업을 해보겠습니다.



우선 SceneDelegate의 경우 iPadOS의 Multiple window 기능때문에 추가된 개념으로 UI의 생명주기를 AppDelegate에서 분담하여 관리하도록 추가된 개념입니다.(추후에 자세하게 SceneDelegate 개념에 대해서 정리하겠습니다.)

<img width="811" alt="image" src="https://user-images.githubusercontent.com/33486820/72036823-631a8680-32df-11ea-92fa-1e040d7c01eb.png">



<img width="800" alt="image" src="https://user-images.githubusercontent.com/33486820/72036806-59911e80-32df-11ea-92bc-4aca66dbd7db.png">

그래서 함수도 iOS 13 이상에만 쓸수있게 노테이션이 붙어있죠.. 이것을 제거하는 방법은 상당히 간단합니다.



<br>



###  1. 일단 SceneDelegate.swift를 지우자

일단 SceneDelegate를 지우는 겁니다. 과감하게 지웁시다.

<img width="892" alt="image" src="https://user-images.githubusercontent.com/33486820/72037036-14b9b780-32e0-11ea-9787-daa89a2e7bd7.png">



<br>



### 2. 13미만의 AppDelegate 코드를 그대로 가져오자

이전 버전의 AppDelegate의 내용을 그대로 가져오시면됩니다.

SceneDelegate에 있던 UIWindow객체는 반드시 AppDelegate로 가져오셔야합니다.(그냥예전 프로젝트 복붙 추천합니다!)

```swift
import UIKit

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?
    
    func application(_ application: UIApplication, supportedInterfaceOrientationsFor window: UIWindow?) -> UIInterfaceOrientationMask {
        return [.portrait, .landscape]
    }


    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        return true
    }

    func applicationWillResignActive(_ application: UIApplication) {
      
    }

    func applicationDidEnterBackground(_ application: UIApplication) {
    
    }

    func applicationWillEnterForeground(_ application: UIApplication) {

    }

    func applicationDidBecomeActive(_ application: UIApplication) {

    }

    func applicationWillTerminate(_ application: UIApplication) {

    }
}
```



<br>



### 3. info.plist의 Application Scene Manifest를 삭제



<img width="863" alt="image" src="https://user-images.githubusercontent.com/33486820/72037266-dd97d600-32e0-11ea-8ea6-4e45e9a1cb82.png">


> Application Scene Manifest 에 관련된 내용은 이 블로그를 참고하시길 바랍니다.(https://jercy.tistory.com/11)



<br>



이제 테스트를 해보면 AppDelegate에서 정상적으로 함수가 호출되는 것을 알 수 있습니다. SceneDelegate 개념이 나오고 좀 많이 당황했지만 AppDelegate의 기능 자체가 사라진것은 아니기 때문에 조금더 철저하게 대비를 해야겠습니다.



<br>

## Reference

- https://jercy.tistory.com/11
- [zedd iOS - SceneDelegate](https://zeddios.tistory.com/811)
- [Naver Login iOS SDK Documents](https://developers.naver.com/docs/login/ios/)
- [이동건의 이유있는 코드 - 네아로 사용법](https://baked-corn.tistory.com/119)

