---
layout: post
current: post
navigation: True
title: iOS LifeCycle, ViewController LifeCycle
date: 2019-10-14 21:18:00
tags: Swift iOS
class: post-template
subclass: 'post tag-fables'
author: gaki
---



앱 생명 주기는 크게 두가지 상태등을 정의해 놓은 일종의 "앱이 실행이 되었다가 사라지는" 하나의 과정이라고 할 수 있다.  

## Swift의 앱 실행 

1. Main 함수가 실행
2. Main 함수는 UIApplicationMain 함수를 호출
3. UIApplicationMain 함수는 앱의 본체에 해당하는 `UIApplication` 객체 생성 
   - 싱글톤 객체로 Event Loop에서 발생하는 여러 이벤트 들을 감지하고 Delegate에 전달하는 역할을 한다. 예를 들어 앱이 백그라운드로 갈때, 메모리 부족 경고를 할 때와 같은 상황들을 감지하여 Delegate에 전달한다.  
4. nib 파일을 사용하는 경우나, info.plist 파일을 읽어들여 파일에 기록된 정보를 참고하여 그 외에 필요한 데이터를 로드한다.

5. `@UIApplicationMain` 어노테이션이 있는 클래스를 찾아 `AppDelegate` 객체를 생성   

6. `Main Event Loop`를(touch, text input 등 유저의 액션을 받는 루프) 실행 및 기타 설정  



## The Main Function

`Objective-C` 는 C언어의 기반 프로그램으로써 시작점은 `main` 함수이다 아래와 같이 `main.m` 이란 파일이 기본적으로 생성 되어져 있다.

##### Objective-C 의 main function

```objective-c
#import <UIKit/UIKit.h>
#import "AppDelegate.h"
 
int main(int argc, char * argv[])
{
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```

Swift의 경우 C기반 언어가 아니라 main 파일이 존재하지 않으며 시작 포인트 역시 존재하지 않는다. 하지만 시작 진입점이 없으면 안되기 때문에 **anotation** 표기로 대체한다.

##### Swift 의 @UIApplicationMain 

```swift
import UIKit

@UIApplicationMain // 어노테이션으로 표기

class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?
    
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {

    }
}
```



## UIApplicationMain Function

`UIApplicationMain` 함수는 코코아 터치 CocoaTouch 프레임워크에서 앱의 Life Cycle을 시작하는 함수이다. UIApplication 객체의 인스턴스를 만들고, 해당 객체의 앱으로서 기능하기 위한 기반을 마련하는데 이 과정을 **앱 로딩 프로세스** 라고 한다.

```swift
import UIKit

UIApplicationMain(
    CommandLine.argc,
    UnsafeMutableRawPointer(CommandLine.unsafeArgv)
        .bindMemory(
            to: UnsafeMutablePointer<Int8>.self,
            capacity: Int(CommandLine.argc)),
    nil,
    NSStringFromClass(AppDelegate.self)
)

// UIApplicationMain 정의
public func UIApplicationMain(_ argc: Int32,
_ argv: UnsafeMutablePointer<UnsafeMutablePointer<Int8>>!,
_ principalClassName: String?,
_ delegateClassName: String?) -> Int32

```

#### UIApplicationMain 함수의 4개의 인자

- 첫번째, 두번째 인자는 argc, argv는 main 함수로 받는 것으로 shell 에서 프로그램을 실행 할때 프로그램 실행 명령어와 함께 인자로 들어오는 값을 넣어주는 것이다
- 세번째 `principalClassName`: UIApplication 클래스를 서브클래싱한 경우 해당 클래스 이름을 전달한다. nil을 쓰는 경우 이 값은 UIApplication 으로 고정된다.
- 네번째 `delegateClassName`: 앱 델리게이트 클래스 이름이다. 만약 nib 파일 내에 앱 델리게이트 객체가 정의 되어있다면 nil을 전달해야한다.

> 결론 :  iOS에서는 우선 프로젝트 언어마다 다르겠지만 main 이 되는 함수에서 UIKit 프레임워크에 권한을 위임한다는 사실이다. `UIApplicationMain` 함수는 앱의 핵심 객체들을 생성하고, 유효한 스토리보드 파일로 부터 UI를 불러오며 개발자가 작성한 코드들을 호출하여 초기 세팅 작업을 해준다. 또한 앱의 Run Loop 를 실행시켜 이러한 프로세스들을 처리한다. 즉 내부적으로 실행에 필요한 제어는 이 함수가 알아서 해주고 개발자는 스토리보드 파일과 직접 작성한 코드만 작성하면 된다.



<hr>



## 앱의  Structure

앱이 시작되면서 `UIApplicationMain` 함수는 몇가지 중요한 객체를 생성하고 앱을 실행시킨다. 앱의 심장 역할을 하는 객체 `UIApplication` 객체로 이는 시스템과 앱의 다른 객체들간의 상호작용을 원할하게 하는 역할을 한다.

iOS앱은 기본적으로 **MVC(Model-View-Controller)** 구조를 사용한다. 이 다자인 패턴의 핵심은 앱의 Data와 비지니스 로직을 UI요소(View)로부터 분리를 시켜준다. 이 구조는 화면의 크기가 다른 여러 디바이스에서 실행되야 하는 앱을 만드는데 중요한 역할을 담당한다.

![image](https://user-images.githubusercontent.com/33486820/66727340-73a29880-ee79-11e9-82cb-328d0ce3cd8f.png)

- UIApplication 객체: `UIApplication` 객체는 event loop 와 고수준의 앱 동작 들을 관리한다. 이 객체는 또한 주요 앱 전환과 일부 특별한 이벤트(푸시 알림) 발생을 개발자가 정의한 delegate객체에 전달한다. `UIApplication` 객체는 상속없이 그대로 사용해야한다.

- App delegate 객체: App Delegate는 개발자가 작성한 코드 중 핵심이 된다. 이 객체는 `UIApplication` 객체와 연결되어 앱의 초기화, 상태 전환 그리고 많은 고 수준의 앱의 이벤트들을 처리하는 일을 한다. 또한 이 객체는 모든 앱에 존재하는 유일한 객체이기 때문에 앱의 초기 데이터 구조 설정을 하는데 자주 사용되곤한다.

- Documents and Data Model 객체: Data Model 객체들은 앱의 컨텐츠를 저장하고 있으며 해당 앱에만 한정되어 저장된다. 금융 관련 앱은 금융 거래등과 관련된 데이터베이스를 저장하고 있을 수 있고 메모 앱은 이미지 객체나 이를 그리고 작성하는데 필요한 명령어들을 포함할 수 있다. 앱은 또한 Document 객체(`UIDocument` 의 사용자 정의 서브클래스)를 사용하여 일부 혹은 전체 데이터 객체들을 관리할 수 있다. Document 객체는 반드시 필요한 객체는 아니지만 단일 파일이나 패키지에 속하는 데이터를 그룹으로 묶어 관리하는데 편리한 기능을 제공한다.

- View Controller 객체: 뷰 컨트롤러 객체는 화면 위 컨텐츠의 시각화를 담당한다. 뷰 컨트롤러는 단일 뷰와 해당 뷰의 자식 뷰들 집합을 관리한다. 컨텐츠가 보요질 때 뷰 컨트롤러는 뷰들을 앱의 윈도우에 설치하여 이들이 화면상에서 보여질 수 있게 해준다. `UIViewController` 클래스는 모든 뷰 컨트롤러 객체의 기본 클래스이다. 이는 뷰들을 불러오고 보여주며 디버이스의 방향에 따라 회전시키는 기본적인 기능들을 제공하며 몇몇 시스템의 기본적인 행위들도 제공한다. UIKit 과 다른 프레임워크들은 ImagePicker, Tap bar Interface, Navigation Interface 와 같은 기본 시스템 인터페이스를 구현하기 위해 뷰 컨트롤러 클래스들을 추가적으로 정의하여 제공하고 있다.

- UIWindow 객체 : `UIWindow`  객체는  화면상의 하나 이상의 뷰들의 표시를 조정한다. 대부분의 앱은 메인 화면에 컨텐츠들을 표시하는 **하나의 윈도우** 만을 가지고 있다. 하지만 외부 디스플레이에서 컨텐츠를 표시한다면 추가적은 윈도우를 가질 수 있다. 앱의 컨텐츠를 바꾸기 위해선 뷰 컨트롤러를 사용하여 해당 윈도우에 존재하는 뷰들을 변경해주어야 한다. 절대 윈도우 그 자체를 변경해선 안된다. 뷰를 위치시는 것 외에도 윈도우는 `UIApplication` 과 상호작용하여 부와 뷰 컨트롤러에 이벤트를 전달하기도 한다.

- View 객체, Control 객체, Layer 객체: 뷰와 컨트롤들은 앱의 컨텐츠의 시각화를 담당한다. 뷰는 하나의 객체로 지정된 사각형의 영역에 컨텐츠를 그리며 해당 영역에서 발생하는 이벤트를 처리해야 하는 역할을 갖고있다. 컨트롤은 특별한 타입의 뷰로 Button, TextField, Toggle 등을 구현하는 역할을 갖고있다.  

  UIKit 프레임워크는 여러다른 타입의 컨텐츠들을 표시하기 위한 기본적인 뷰들을 제공한다. 직접적으로 `UIView` (혹은 `UIView` 의 SubClass)를 상속하여 자신만의 사용자 정의 뷰를 정의할 수 있다. 게다가 View 와 Controller 를 통합하는것 외에도 앱은 **Core Animcation** 레이어를 View와 Controller의 계층에 통합시킬 수 있다. 레이어 객체는 시각적인 컨텐츠의 표시를 담당하는 데이터 객체이다. View는 Layer 객체를 집중적으로 사용하여 컨텐츠들을 화면에 보여주기 위해 렌더링 작업을 진행한다. 개발자는 사용자 정의 레이어 객체를 추가하여 복잡한 애니메이션과 다른 타입의 정교한 시각적인 효과를 구현할 수 있다.



## The Main Run Loop

앱의 메인 런 루프(run loop)는 사용자와 관련된 모든 이벤트들을 처리한다. `UIApplication` 객체는 런치 시점에 메인 런 루프를 실행시키고 이를 이벤트나 뷰에 기반한 인터페이스 갱신을 처리하는데 사용한다. 이름에서 알 수 있듯이 메인 런루프는 **앱의 메인 스레드** 에서 동작한다. 이런 행위는 사용자의 이벤트가 도착한 순서대로 처리되는 것을 보장한다.(ex UI의 변화 등)

![image](https://user-images.githubusercontent.com/33486820/66747981-eedb6d00-eec0-11e9-91ab-438ab601c719.png)

사용자가 디바이스에서 특정 액션을 취하게 되면, 그 액션에 해당하는 이벤트가 시스템에 의해 생성되어 UIKit에서 생성한 port를 통해 앱에 전달된다. 전달된 이벤트들은 Event Queue 에 보관되고 하나씩 Main Run Loop로 전달되어 처리된다( Main Thread로 처리하기 때문에 Queue의 FIFO 형식으로 순서대로 처리 )

다양한 형태의 이벤트들이 iOS 앱에 전달된다. 아래는 가장 기본이 되는 이벤트 들이다. 어리헌 타입의 이벤트들은 메인 런 루프를 통해 전달된다. 하짐나 그중 몇몇은 델리게이트 객체에 전달 되거나 사용자 정의 블록에 전달될 수 있다.



## 앱의 상태 변화

어느 순간에도 앱은 다음 상태 목록 중 하나의 상태를 갖는다. 시스템은 시스템의 전반에 걸쳐 일어나는 특정 순간에 대한 대응으로 앱의 상태를 변경한다. 예를 들어 사용자가 홈 버튼을 누르거나, 전화가 들어오거나 혹은 다른 기타 방해가 생겨났을 떄 그에 대한 대응으로 앱은 상태를 변경한다.

![image](https://user-images.githubusercontent.com/33486820/66749236-4a5b2a00-eec4-11e9-8fad-d6dea1c794d7.png)

1. **Not running** - 앱이 아직 실행되지 않았거나 혹은 실행되고 있었지만 시스템에 의해 종료된 상태

2. **Foreground** - 앱이 실행되어 사용자에게 보여지고 있는 상태
   - **Inactive** - 앱이 foreground에서 실행되고 있지만 현재 어느 이벤트도 받고 있지 않는 상태(다른 코드를 실행하고 있을 수도 있다.) 앱은 대개 **현재 상태에서 다른 상태로 전환 될때 잠깐 동안만 이 상태(Inactive)에 머문다**.
   - **Active** - 앱이 foreground에서 실행되고 있고 이벤트를 전달 받고 있는 상태. **foreground에서 실행되고 있는 앱이 갖는 상태**이다.

3. **Background** - 앱이 background에 있으면서 코드를 실행시키고 있는 상태. 대부분의 앱이 Suspended 상태로 들어가기 전 background 상태로 먼저 진입한다. 하지만 추가 실행 시간을 요구하는 앱은 일정 시간 동안 이 상태를 유지할 수 있다. 추가적으로 background 상태로 직접 실행되는 앱은 Inactive 상태 대신 바로 background 상태로 진입한다.

4. **Suspended** - 앱이 background 상태에 있지만 코드를 실행시키고 있지 않은 상태. 앱이 background 상태로 전환된 후 더이상 작업을 수행하지 않으면 시스템에서 앱을 Suspend 상태로 바꾼다. 앱은 여전히 메모리에 존재하며 Suspend 상태가 될 당시의 상태를 저장하고 있지만, CPU나 배터리를 소모하지 않는다. Suspend 상태의 앱은 foreground 상태의 앱을 위해 메모리 부족 등의 이유로 시스템에 의해 언제든지 종료된다. 이후 앱을 실행하면 이전 상태의 화면은 나오지 않고 앱이 재시작 된다.



##### 앱을 전면으로 실행

앱이 시작되는 부분에서 시스템은 프로세스와 앱의 메인 스레드, 그리고 메인스레드에서 main 함수를 생성한다. main 함수에서 UIKit 프레임워크를 즉시 다룰 수 있고, UIKit 프레임워크는 앱의 초기화와 실행 준비를 한다.

![image](https://user-images.githubusercontent.com/33486820/66750405-5694b680-eec7-11e9-99c2-af1625f95bf4.png)



##### 앱을 background로 실행하기

참고 : [Background 에서 실행 ](https://medium.com/cashwalk/ios-background-mode-9bf921f1c55b)

![image](https://user-images.githubusercontent.com/33486820/66750524-a4112380-eec7-11e9-8261-9f5c330843d9.png)



## AppDelegate.swift 파일에는 어떤 것이?

appDelegate  파일이 존재하고 여기에 UIApplicationMain Function이 어노테이션으로 표기가 되어있다. 그리고 앱의 background 나 foreground의 진입시에 호출되는 델리게이트 함수등이 존재하고 CoreData 등 다양한 작업을 할 수 있는 영역이다.

### App Delegate 객체

- AppDelegate객체  
	AppDelegate 객체는 UIApplication 객체로 부터 메시지를 받았을 때, 해당 상황에서 실행 될 함수들을 정의한다. 

app delegate는 **AppDelegate클래스의 인스턴스**, app delegate는 앱의 상태에 따라 응답하는 콘테츠가 그려지는 **창(window)** 를 만든다.  

> AppDelegate.swift 있으므로 AppDelegate클래스가 만들어지고, 이 AppDelegate 클래스에 인스턴스인 app delegate가 앱 내용이 그려질 창(window)를 만든다.  


- AppDelegate.swift는 entry point와, 앱의 입력 이벤트를 전달하는 run loop를 생성한다.  

<img width="321" alt="image" src="https://user-images.githubusercontent.com/33486820/58061834-d9347680-7bb2-11e9-875a-862b1820b9eb.png">  

`@UIApplicationMain` 어노테이션에 의해 AppDeleage.swift 파일 자체가 객체가 된다 즉 UIApllicationMain함수를 호출하고 AppDelegate클래스의 이름을 delegate 클래스에 전달하는 것과 동일하다.  

그 후 시스템은 이에대한 응답으로 **응용프로그램 객체(application object)** 를 생성한다. application object는 **App의 Life Cycle**을 담당한다.  

또한 시스템은 AppDelegate클래스의 객체를 생성하고 이를 application object에 할당한다.  

최종적으로, 시스템은 앱을 실행한다.  

AppDelegate클래스는 새 프로젝트를 만들 때 마다 자동으로 생성된다. 

<img width="473" alt="image" src="https://user-images.githubusercontent.com/33486820/58094257-2fcd9f00-7c0b-11e9-9ef5-4ab47b192cec.png">

- `UIApplicationDelegate`  

AppDelegate클래스는 UIApplicationDelegate 프로토콜을 체택한다.  
이 프로토콜은 앱을 세팅하고, 앱 상태 변화에 응답하며 다른 app-level이벤트를 처리하는 데 사용하는 여러가지 방법을 정의한다.  

- AppDelegate의 window 프로퍼티  

<img width="733" alt="image" src="https://user-images.githubusercontent.com/33486820/58094491-cac67900-7c0b-11e9-95a2-3378dd29ba94.png">  

이 프로퍼티는 앱의 창(window)에 대한 **참조**를 저장한다.  
이 window는 앱의 view계층구조의 **루트**를 나타낸다. 이는 앱 콘텐츠가 모두 그려지는 곳이다. window프로퍼티는 **Optional 프로퍼티**이다.  
즉, nil 값이 될수있는 것을 허용한다는 것이다.  

- AppDelegate 클래스의 delegate 메서드  

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool

func applicationWillResignActive(_ application: UIApplication)

func applicationDidEnterBackground(_ application: UIApplication)

func applicationWillEnterForeground(_ application: UIApplication)

func applicationDidBecomeActive(_ application: UIApplication)

func applicationWillTerminate(_ application: UIApplication)
```

위의 delegate 메서드를 사용하면, **응용프로그램 객체(application object)가 app delegate(객체)와 통신이 가능하다**.  
앱상태가 전환되는 동안(background ,foreground) 응용프로그램 객체는 위 메서드들 중 해당하는 delegate 메서드를 호출하여 앱이 응답 할 수 있는 기회를 제공한다.  
자동으로 반드시 수행되기 때문에 따로 호출에 대한 작업을 할 필요는 없고, 응용프로그램 객체가 해당 작업을 알아서 처리한다.  

 

### AppDelegate 클래스의 Delegate 메서드  

```swift
// 앱이 처음 시작될 때 실행
func application(_:didFinishLaunchingWithOptions:)

// 앱이 Active 에서 Inactive로 이동될 때 실행
func applicationWillResignActive(_ application: UIApplication)

// 앱이 Background 상태일 때 실행
func applicationDidEnterBackground(_ application: UIApplication)

// 앱이 Background에서 Foreground로 이동 될때 실행(아직 Foreground에서 실행중이진 않는다)
func applicationWillEnterForeground(_ application: UIApplication)

// 앱이 Active 상태가 되어 실행 중일 때
func applicationDidBecomeActive(_ application: UIApplication)

// 앱이 종료될 때 실행
func applicationWillTerminate(_ application: UIApplication)
```

  


# ViewController-Life-Cycle  

<img width="845" alt="image" src="https://user-images.githubusercontent.com/33486820/58097611-c94c7f00-7c12-11e9-984c-6bda46cb03cd.png">


## viewDidLoad  


> "Called after the controller's view is loaded into memory"  

뷰의 컨트롤러가 메모리에 로드되고 난 후에 호출된다.  
해당 메서드는 뷰의 로딩이 완료 되었을때 **시스템에 의해 자동으로 호출**되기 때문에 일반적으로 리소스를 초기화하거나 초기 화면을 구성하는 용도로 주로 사용한다.  
화면이 처음 만들어질 때 한번 실행하므로 **처음 한번 만 실행해야하는 코드의 초기화 기능**을 이메서드내에 코드를 작성하면 된다.  



## viewWillAppear  

<img width="937" alt="image" src="https://user-images.githubusercontent.com/33486820/58098344-4fb59080-7c14-11e9-8573-96fd605ccc35.png">  

위의 그림 처럼 초기에 A view는 `viewDidLoad()` 와 `viewWillAppear()`이 모두 호출된다. A에서 B로 view이동이 일어날때도 마찬가지로 B view의 `viewDidLoad()` 와 `viewWillAppear()`이 호출된다. 하지만 B에서 A로 다시 돌아갈 경우 `viewDidLoad()`의 경우 화면이 처음 만들어질 때 한번만 실행하므로 `viewWillAppear()`만 호출이 되는 것을 볼수있다.  
**다른 뷰에서 갔다가 다시 돌아오는 상황에 해주고 싶은 처리를 viewWillAppear에서한다.**  

![image](https://user-images.githubusercontent.com/33486820/58109039-d6c03400-7c27-11e9-87ff-083283dc60f7.png)  


> 참고: 네비게이션 컨트롤러의 경우 view가 stack 처럼 쌓이기 때문에 A에서 B로 갔다가 B 에서 다시 A로 올경우 A의 viewDidLoad()함수는 호출이 되지 않는다. A의 경우 메모리 스택에 남아 있고 B의 경우 A로 back 을 하게 될경우 pop이 수행이 되어 메모리 스택에서 사라지게 되고 결국 다시 B를 이동하게 될때 처음 로드 되는 것처럼 메모리스택에 push 되고 viewDidLoad()가 호출되게 된다.  




## viewDidAppear  

뷰가 나타났다는 것을 Controller에게 알리는 역할을 한다. 또한 화면에 적용될 애니메이션을 그려준다.  
뷰가 화면에 나타난 직후에 실행되기 때문에 이것을 제외하고는 `viewWillAppear()`와 거의 흡사하다.  

 

## viewWillDisappear  

뷰가 사라지기 직전에 호출되는 함수로써, 뷰가 삭제되려고 하는 것을 ViewController에게 통지한다.  

## viewDidDisappear 

ViewController가 뷰가 제거되었음을 알려준다.  

 
<hr>

## Reference  

- https://zeddios.tistory.com/43
- https://medium.com/ios-development-with-swift/앱-생명주기-app-lifecycle-vs-뷰-생명주기-view-lifecycle-in-ios-336ae00d1855 
- https://www.edwith.org
