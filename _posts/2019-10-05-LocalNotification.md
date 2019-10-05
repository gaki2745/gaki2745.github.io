---
layout: post
current: post
navigation: True
title: iOS Message(2/3) - UserNotification편
date: 2019-10-05 11:18:00
tags: Swift iOS
class: post-template
subclass: 'post tag-fables'
author: gaki
---  


## iOS 로컬알림

그 다음 공부할 내용은 로컬 알림입니다. 로컬 알림은 앱 내부에서 만든 특정 메시지를 iOS의 알림 센터를 통해 전달하는 방법입니다. 앞의 UIAlertController의 경우는 앱이 실행 되어있을때 사용자가 특정 이벤트를 통해서만 메시지를 보낼 수 있는 반면에 앱이 백그라운드 상태일 때에도 메시지를 전달할 수 있는 대표적인 방법중 하나입니다.  

로컬 알림은 iOS 스케줄러에 의해 발송되며, 앱 내부에서 미리 메시지를 구성한 후 발송될 시각을 iOS 스케줄러에 등록해 두면 해당 시각에 맞추어 자동으로 발송됩니다. 모든 발송은 iOS 스케줄러에 의해 관리되므로 앱이 실행 중이 아니더라도 메시지를 받을 수 있을 뿐만 아니라, 알림 센터에 표시된 메시지를 클릭하여 앱을 실행시키고 원하는 기능을 실행하거나 특정 화면으로 이동하게끔 처리할 수 있습니다.  

로컬 알림은 앱의 실행 여부와 상관 없이 사용할 수 있지만, 만약 앱이 실행 중이라면 로컬 알림 보디는 앞의 UIAlertController 를 이용한 알림이 더욱 확실하고 효과 적입니다. 로컬 알림은 **앱을** **종료할** **때** **또는** **백그라운드** **상태로** **진입할** **때** **등** **사용자의** **관심으로** **부터** **멀어지는** **상황의** **주의** **환기** **목적**  으로 사용하는 것이 좋습니다.   

이는 로컬 알림에 관련된 작업이 뷰컨트롤러가아닌 **앱** **델리게이트** **클래스에서** **이루어** **지는** **것** 과 관련이 되어있습니다. 앱의 생명주기와 관련하여 적절한 시점에 로컬 알림을 사용하면 **앱의** **홍보** **효과** 를 거둘 수 있습니다!  

경험) 사용하다가 어느 시점에서 AppStore 평점을 묻거나, 종료할때 정말 종료 할꺼냐는 등의 알림 등 앱 생명주기에 맞추에 적재적소에 사용을 해야한다! 



**UserNotification** **프레임** **워크를** **이용한** **로컬** **알림**  

UserNotification 프레임워크는 UIKit 프레임워크나 Foundation 프레임워크와 같은 수준의 **독립된** 프레임워크 입니다.

> ```swift
> import UserNotifications
> ```

위와 같이 프로젝트에서 반입 구문을 추가해주어야 사용할 수 있습니다. 

UserNotification 프레임워크에서 중요한 객체 4가지는 다음과 같습니다.

- UNMutableNotificationContent
- UNTimerIntervalNotificationTrigger
- UNNotificationRequest
- UNUserNotificationCenter

### UNMutableNotificationContent 

알림에 필요한 메시지와 같은 기본적인 속성을 담는 알림 콘텐츠의 역할을 합니다. 이 객체를 통해 **로컬 알림 타이틀, 서브 타이틀 및 알림 메시지를 설정** 할 수 있고 **앱 아이콘에 표시될 배지나 사운드 설정** 또한 이 객체를 통해 설정합니다. `UNNotificationContent` 는 수정이 불가능하고 객체로부터 속성을 읽을 수 만 있고 속성을 설정할려면 반드시 `UNMutableNotificationContent` 객체를 사용해야합니다.  



### UNTimerIntervalNotificationTrigger  

알림의 **발송 조건** 을 관리합니다. 설정할 수 있는 속성은 **발생 시간 과 반복 여부** 입니다. `UNTimerIntervalNotificationTrigger` 의 경우는 입력값의 단위가 **초 단위** 이기 때문에 수분 후의 알림을 설정하는 등 시간 간격을 설정하여 알림 메시지를 발송 할 수 있고 `UNCalanderNotificationTrigger` 객체를 사용하여 **하루 중 특정 시각**에 맞추어 알림 메시지를 전송 할 수 있습니다.



### UNNotificationRequest

 위의 두 객체를 통해 알림 콘텐츠와 알림 발생 조건이 준비가 되면 이들을 모아 **알림 요청 객체** 를 생성해야합니다. 이 때 사용되는 클래스가 `UNNotificationRequest` 입니다,



### UNUserNotificationCenter

실제 발송을 담당하는 Center 입니다. 등록된 알림 내용을 확인하고 정해진 시각에 발송하는 역할을 맡고 있습니다. 이 객체는 **싱글턴 방식** 으로 동작하기 때문에 따로 인스턴스를 생성하지 않고 `current()` 메소드를 통해 참조 정보만 가져올 수 있습니다. 앞에서 생성한 `UNNotificationRequest` 요청 객체를  `UNUserNotificationCenter::add(_:)` 메서드를 이용하여 추가 하기만 하면 알림 등록과정이 모두 완료 됩니다.



## UserNotification 실습  

- application(_:didFinishLaunchingWithOptions:)` 메소드에 다음과 같은 코드를 추가합니다.  

```swift
    // 앱이 가장 처음 실행될때 호출 되는 메서드
    // 애플리케이션에서 사용할 클래스와 리소스들이 모두 메모리에 로드되고 아직 애플리케이션의 첫 화면을 모바일 디바이스에 띄우기 직전
    // 시작화면이 스크린에 표시되고 있는 동안에 호출이 된다.
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        if #available(iOS 10.0, *) {
            // 경고창, 배지, 사운드를 사용하는 알림 환경 정보를 생성하고, 사용자 동의 여부 창을 실행
            let notiCenter = UNUserNotificationCenter.current()
          	// requestAuthorization("1. 알림 메시지에 포함될 항목", "2. 클로저")
            notiCenter.requestAuthorization(options: [.alert, .badge, .sound]) { (didAllow, e) in }
        
        } else {
            
        }
        
        return true
 }

```



`application(_:didFinishLaunchingWithOptions:)` 메서드는 앱이 처음 실행될 때 호출이 됩니다. 이와 관련 하여 더 자세한 내용은 [gaki: iOS-LifeCycle](https://gaki2745.github.io/Second-Post) 글을 참고하시길 바랍니다. 

위 코드에서 확인 할 수 있듯이 앱이 처음 실행 될때 `notiCenter = UNUserNotificationCenter.current()` 를 통해 객체를 생성합니다. 앞서 말씀 드렸듯이 **싱글턴 패턴**으로 정의되어 있기 때문에 인스턴스 객체를 마음대로 생성해서는 안됩니다. 위와 같이 `UNUserNotificationCenter.current()` 의 방식은 시스템에서 제공하는 인스턴스를 받아오는 방법이기에 가능합니다.  

`UNUserNotificationCenter` 의 싱글턴 인스턴스를 생성 해서 받아왔다면 `requestAuthorization()` 메서드를 호출하여 사용자에게 알림 설정에 대한 동의를 받아야합니다. 

1. 첫번째 매개변수로 알림 메시지에 포함 될 항목을 `UNAuthorizationOptions` 의 열거형 객체에 정의되어있는 값들을  받는다. 위의 코드에서는 **경고창, 배지, 사운드** 를 사용할 수 있도록 알림 환경을 설정한다.
2. 두번째 매개변수로는 사용자의 알림 동의 여부를 ture/false 형태로 전달받는 클로저이다. `didAllow: 사용자 동의 여부, e: error`   



- `applicationWillResignActive(_:)` 메소드에 아래와 같은 코드를 추가합니다. (iOS 13 이상부터는 AppDelegate에 미리 작성이 되어있지 않아서 추가해주어야 함)

```swift
// 앱이 활성화 상태를 잃었을때 실행되는 메서드
func applicationWillResignActive(_ application: UIApplication) {
        
        if #available(iOS 10.0, *) {
            // 알림 동의 여부를 확인
            UNUserNotificationCenter.current().getNotificationSettings { (settings) in
                if settings.authorizationStatus == UNAuthorizationStatus.authorized {
                    // 알림 콘텐츠 객체
                    // 속성을 수정 가능한 mutable 객체로 생성
                    let notiContents = UNMutableNotificationContent()
                    notiContents.badge = 1
                    notiContents.title = "로컬 알림 메시지"
                    notiContents.subtitle = "앱을 다시 실행시켜보세요. 많은 메시지들이 도착했습니다."
                    notiContents.body = "읽지 않는 메시지들이 9241개 있습니다."
                    notiContents.sound = UNNotificationSound.default
                    notiContents.userInfo = ["name": "구영준"]
                    
                    // 알림 발송 조건 객체
                    // 발송 시간과 반복 조건 설정
                    let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 5, repeats: false)
                    
                    // 알림 요청 객체
                    // 매개변수로 알림 콘텐츠 객체와 알림 발송 조건 객체를 넘겨주는 것을 확인 가능
                    let request = UNNotificationRequest(identifier: "wakeup", content: notiContents, trigger: trigger)
                    
                    // 노티피케이션 Center에 추가
                    UNUserNotificationCenter.current().add(request, withCompletionHandler: nil)
                } else {
                    print("사용자가 동의하지 않았습니다.")
                }
            }
        } else {
            // iOS 9 이하 UILocationNotification 객채를 이용한 로컬 알림을 수행해야하는 코드 작성
        }
    }

extension ViewController : UNUserNotificationCenterDelegate{
    //To display notifications when app is running  inforeground
    
    //앱이 foreground에 있을 때. 즉 앱안에 있어도 push알림을 받게 해준다.
    func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        completionHandler([.alert, .sound, .badge])
    }
    
    func userNotificationCenter(_ center: UNUserNotificationCenter, openSettingsFor notification: UNNotification?) {
        let settingsViewController = UIViewController()
        settingsViewController.view.backgroundColor = .gray
        self.present(settingsViewController, animated: true, completion: nil)
        
    }
    
    
}


```



`applicationWillResignActive(_:)` 메서드는 앱이 **활성화 상태를 잃었을 때 실행되는 메서드** 입니다. 앱이 백그라운드로 내려가 비활성화가 되었을때 이 메서드가 호출이 됩니다. 그렇기 때문에 앱이 종료 되었을때 알림을 띄워 주기 위해선 해당 메서드에서 작업을 해주어야합니다.

![image-20191005231009743](/Users/youngjungoo/Library/Application Support/typora-user-images/image-20191005231009743.png)





<hr>

## Reference

- [꼼꼼한 재은씨의 Swift 기본편](https://book.naver.com/bookdb/book_detail.nhn?bid=12320111)
- [Zedd iOS - Notification](https://zeddios.tistory.com/589)
