---
layout: post
title: Naver Maps SDK에 Delegate Proxy 적용시켜보자
subtitle: ""
categories: swift
tags: rxSwift
comments: false
---


NaverMaps 와 같이 외부 프레임워크를 사용할때 delegate사용은 필수적이다. RxSwift를 활용한 MVVM을 학습하던 도중에 Delegate Proxy 에 관련 된 개념이 나와서 적용해보기로 했다. 



> 이 글은 [NaverMaps iOS SDK](https://navermaps.github.io/ios-map-sdk/guide-ko/1.html) , [NaverMaps API Reference](https://navermaps.github.io/ios-map-sdk/reference/)를 참고하여 작성하였습니다.



### Delegate Proxy란?

RxSwift에서 **DelegateProxy.swift** 와 **DelegateProxyType.swift** 가 존재하는것을 확인할 수 있다.

- delegateProxy.swift

<img width="528" alt="스크린샷 2020-01-02 오전 11 06 36" src="https://user-images.githubusercontent.com/33486820/71649401-cfcad980-2d51-11ea-8927-84755becd80d.png">

- delegateProxyType.swift

<img width="530" alt="스크린샷 2020-01-02 오전 11 07 07" src="https://user-images.githubusercontent.com/33486820/71649400-cfcad980-2d51-11ea-9c6f-6ad7f221dd5a.png">



위의 두 파일은 **delegate를 사용하는 외부 프레임워크와 RxSwift의 매개체 같은 역할을 해주는 파일** 이라고 생각하면된다. NaverMaps도 `NMFMapView` 에 관련된 상호작용이나 지도 좌표를 제어하기 위해서는 `NMFMapViewDelegate` 나 `NMFLocationManagerDelegate` 와 같이 delegate를 활용해야한다. Delegate Proxy의 두 파일을 이용하면 RxSwift와의 결합을 통한 작업이 가능해 진다는 것이다.



```swift
// NMFNaverMapView는 NMFMapViewDelegate의 delegate 객체를 가지고 있다.
@interface NMFNaverMapView : UIView
@property(nonatomic, weak, nullable) IBOutlet id<NMFMapViewDelegate> delegate;
```



<br>

지금 까지는 Delegate안의 메서드를 사용할려면 다음과 같이 직접 delegate안의 함수들을 사용하여서 코드를 작성하였다.

```swift
// MARK: - NMFMapViewDelegate
extension MainViewController: NMFMapViewDelegate {
		// 네이버 지도 위 터치 된 곳의 좌표 반환
    func didTapMapView(_ point: CGPoint, latLng latlng: NMGLatLng) {
        print("\(latlng.lat), \(latlng.lng)")
    }
}

// MARK: - NMFLocationManagerDelegate
extension MainViewController: NMFLocationManagerDelegate {
    // 현재 위치 반환
    func locationManager(_ locationManager: NMFLocationManager!, didUpdateLocations locations: [Any]!) {
        guard let curLocation = locations.last as? CLLocation else { return }
				print(curLocation)
    }
}
```

<br>

우리는 이제 RxSwift를 사용하기 때문에 이를 RxSwift로 아래와 같이 사용하고 싶은 것이다.

```swift
class NMViewController: UIViewController {
  
	@IBOutlet weak var naverMapView: NMFNaverMapView!
  
  let disposeBag = DisposeBag()
  
  override func viewDidLoad() {
    super.viewDidLoad()
  	initialize()  
  }
  
  func bind() {
    naverMapView.rx.didTapMapView
    	.asObservable()
    	.subscribe(onNext: { (point) in
      	print("\(latlng.lat), \(latlng.lng)")
      })
    	.disposed(by: disposeBag)
    
		naverMapView.rx.locationManager
    	.asObservable()
    	.subscribe(onNext: { (locations) in
        guard let curLocation = locations.last as? CLLocation else { return }
        print(curLocation)
      })
    	.disposed(by: disposeBag)
  }
  
  ...
}
```



그럼 어떻게 외부 프레임워크의 Delegate를 Rx와 연결하는지 DelegateProxy의 구조를 먼저 보자



<br>

#### delegateProxy.swift

```swift
open class DelegateProxy<P: AnyObject, D>: _RXDelegateProxy {
        public typealias ParentObject = P
        public typealias Delegate = D

        private var _sentMessageForSelector = [Selector: MessageDispatcher]()
        private var _methodInvokedForSelector = [Selector: MessageDispatcher]()

        /// Parent object associated with delegate proxy.
        private weak var _parentObject: ParentObject?

        fileprivate let _currentDelegateFor: (ParentObject) -> AnyObject?
        fileprivate let _setCurrentDelegateTo: (AnyObject?, ParentObject) -> Void

        /// Initializes new instance.
        ///
        /// - parameter parentObject: Optional parent object that owns `DelegateProxy` as associated object.
        public init<Proxy: DelegateProxyType>(parentObject: ParentObject, delegateProxy: Proxy.Type)
            where Proxy: DelegateProxy<ParentObject, Delegate>, Proxy.ParentObject == ParentObject, Proxy.Delegate == Delegate {
            self._parentObject = parentObject
            self._currentDelegateFor = delegateProxy._currentDelegate
            self._setCurrentDelegateTo = delegateProxy._setCurrentDelegate

            MainScheduler.ensureRunningOnMainThread()
            #if TRACE_RESOURCES
                _ = Resources.incrementTotal()
            #endif
            super.init()
        }
  ...
```



`DelegateProxy` 클래스는 위와 같이 **Object와 Delegate**를 세팅값으로 받고 있다. 

- Object : `NMFNaverMapView`
- Delegate: `NMFMapViewDelegate`



<br>

#### delegateProxyType.swift

```swift
public protocol DelegateProxyType: class {
    associatedtype ParentObject: AnyObject
    associatedtype Delegate
    
    /// It is require that enumerate call `register` of the extended DelegateProxy subclasses here.
    static func registerKnownImplementations()

    /// Unique identifier for delegate
    static var identifier: UnsafeRawPointer { get }
    static func currentDelegate(for object: ParentObject) -> Delegate?
  ...

```

delegateProxyType의 경우 프로토콜로 정의 되어 있고 필수적으로 구현해야하는 함수들의 정의 되어있다.

대략적인 구조는 보고 직접 구현하면서 익혀보자. **CustomDelegateProxy** 를 구현하기 위해서는 아래와 같이 3가지가 필요하다.

- `DelegateProxy<NMFNaverView , NMFMapViewDelegate>`
- `DelegateProxyType`
- `NMFMapViewDelegate` 

이제 이 것들을 채택하고 상속한 `RxNMFMapViewDelegateProxy`를 선언해 보자



<br>

### RxNMFMapViewDelegateProxy

```swift
import RxSwift
import RxCocoa
import NMapsMap

class RxNMFMapViewDelegateProxy: DelegateProxy<NMFNaverMapView, NMFMapViewDelegate>, DelegateProxyType, NMFMapViewDelegate {
    static func registerKnownImplementations() {
        self.register { (naverMapView) -> RxNMFMapViewDelegateProxy in
            RxNMFMapViewDelegateProxy(parentObject: naverMapView, delegateProxy: self)
        }
    }
    // 현재 obeject 즉 NMFNaverMapView의 delegate 객체 반환
    static func currentDelegate(for object: NMFNaverMapView) -> NMFMapViewDelegate? {
        return object.delegate
    }
    // delegate 객체 설정 -> NMFNaverMapView.delegate = self 와 같은 역할
    static func setCurrentDelegate(_ delegate: NMFMapViewDelegate?, to object: NMFNaverMapView) {
        object.delegate = delegate
    }
}
```

위와 같이 DelegateProxy 프로토콜에 필요한 메서드 세개를 반드시 구현해주어야한다.

- `registerKnownImplementation()` 
- `currentDelegate`
- `setCurrentDelegate`

<br>



이제는 `NMFLocationManager` 에 관련된 DelegateProxy를 구현해줄려고한다.

그런데 한가지 문제가 있었다. `NMFLocationManagerDelegate` 의 경우 `NMFLocationManager` 가 delegate 객체를 가지고 있지 않고 `sharedInstance()` 메서드를 통해 아래의 함수들로 delegate 를 설정해주고 있었다.

![image](https://user-images.githubusercontent.com/33486820/71702320-11bb5480-2e12-11ea-9a4a-882ee39aded6.png)



대신에  `NMFLocationManager` 도 결국 `CLLocationManagerDelegate` 를 채택하여 사용하기 때문에 `CLLocationManager` 를 이용해서 현재 위치의 경위도 좌표를 가져와 보자



<br>



### 	CLLocationManagerDelegateProxy

실시간으로 사용자의 현재 경위도 좌표를 반환하는 CoreLocation에 `CLLocationManageDelegate` 의 CallBack 함수를 사용한다. 우선적으로 위와 같이 동일하게 delegate proxy를 설정한다.

```swift
import RxCocoa
import RxSwift
import CoreLocation

class RxCLLocationManagerDelegateProxy: DelegateProxy<CLLocationManager, CLLocationManagerDelegate>, DelegateProxyType, CLLocationManagerDelegate {
    static func registerKnownImplementations() {
        self.register { (manager) -> RxCLLocationManagerDelegateProxy in
            RxCLLocationManagerDelegateProxy(parentObject: manager, delegateProxy: self)
        }
    }
    
    static func currentDelegate(for object: CLLocationManager) -> CLLocationManagerDelegate? {
        return object.delegate
    }
    
    static func setCurrentDelegate(_ delegate: CLLocationManagerDelegate?, to object: CLLocationManager) {
        object.delegate = delegate
    }
}

extension Reactive where Base: CLLocationManager {
    var delegateCLLocationManager: DelegateProxy<CLLocationManager, CLLocationManagerDelegate> {
        return RxCLLocationManagerDelegateProxy.proxy(for: self.base)
    }

    var didUpdateLocations: Observable<[CLLocation]> {
        return delegateCLLocationManager.methodInvoked(#selector(CLLocationManagerDelegate.locationManager(_:didUpdateLocations:)))
            .map { return $0[1] as? [CLLocation] ?? [CLLocation]() }
    }
    
}
```


아래 두개의 delegate를 rx와 연동을 하였고 Delegate proxy 작업은 모두 끝났다

- `NMFMapViewDelegate`
- `CLLocationManagerDelegate`

이제 bind를 통해 어떻게 호출하는지 확인해보자.



<br>



### Custom DelegateProxy 사용



```swift
import UIKit
import RxSwift
import RxCocoa
import NMapsMap

class NMViewController: UIViewController {
    
    @IBOutlet weak var naverMapView: NMFNaverMapView!
    @IBOutlet weak var navigationButton: UIButton!
    @IBOutlet weak var latLabel: UILabel!
    @IBOutlet weak var lngLabel: UILabel!
    let locationManager = CLLocationManager()
    let disposeBag = DisposeBag()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setUpNaverMapView()
        locationManager.requestWhenInUseAuthorization()
     		// 사용자의 현재 위치 업데이트 시작 -> 이 함수를 호출해야 callback함수가 호출된다.
        locationManager.startUpdatingLocation()
        bind()
    }
    
    private func setUpNaverMapView() {
        naverMapView.positionMode = .direction
        naverMapView.mapView.zoomLevel = 15
        naverMapView.showLocationButton = true
        naverMapView.showIndoorLevelPicker = true
    }
    
    func bind() {
      	// 지도를 터치했을때 해당 위치의 경위도 좌표를 반환하는 콜백 메서드
        naverMapView.rx.didTapMapView
            .asObservable()
            .subscribe(onNext: {
                print($0)
            })
            .disposed(by: disposeBag)
      
        // 지도가 표시하고 있는 영역이 변경되었을 때 호출되는 콜백 메서드
        naverMapView.rx.regionDidChangeAnimated
            .asObservable()
            .subscribe(onNext: {
                print($0)
            })
            .disposed(by: disposeBag)
      
      	// latLabel  
        locationManager.rx.didUpdateLocations
            .asObservable()
            .map {
                return "경도 : \($0.lat)"
        }
        .bind(to: latLabel.rx.text)
        .disposed(by: disposeBag)
      
        // lngLabel
        locationManager.rx.didUpdateLocations
            .asObservable()
            .map {
                return "위도 : \($0.lng)"
        }
        .bind(to: lngLabel.rx.text)
        .disposed(by: disposeBag)
        
    }
}

```

총 3개의 delegate proxy를 활용하여 delegate 내부의 메서드를 Rx와 연결했다.



<img width="400" alt="스크린샷 2020-01-02 오전 11 06 36" src="https://user-images.githubusercontent.com/33486820/71703777-e9cfef00-2e19-11ea-9e52-016fe7906e64.png">



이렇게 굳이 extension을 통해 ViewController라 massive 하는 것을 막고 또 가장 중요한 Rx와 delegate 메서드를 통해 제어 할 수 있다는 것이 가능해졌다.



delegate proxy가 가능한지 우선 알아보는것이 중요한거 같다 내부에 delegate 객체가 우선 존재해야한다. delegateProxyType 프로토콜의 함수에서 delegate를 설정하고 반환하는 함수가 있기때문에 이를 위해서 객체 우선 있는지 파악해야한다.



`NMFLocationManager` 의 경우 이것이 불가능해서 혹시 다른 방법이 있는지 공부하면서 찾아봐야겠다.



<br>



## Reference

- [NaverMaps iOS API 레퍼런스](https://navermaps.github.io/ios-map-sdk/reference/Protocols/NMFMapViewDelegate.html#/c:objc(pl)NMFMapViewDelegate(im)mapView:regionDidChangeAnimated:byReason:)
- [RxSwift: How to make your favorite Delegate-based-APIs Reactive](https://blog.ippon.fr/2018/11/13/rxswift-how-to-make-your-favorite-delegate-based-apis-reactive/)





