---
layout: post
title: xib를 통해 CustomView 만들기 & xib와 nib의 차이
subtitle: ""
categories: ios
tags: basic
comments: false
---


Custom View를 작업하기 위해 xib 사용법과 그리고 nib xib에 관해 알게된 개념에 대해 정리해보겠습니다.

기본적으로 xib는 `UIViewController`,`UITableViewCell`, `UICollectionViewCell` 에만 xib 연결을 제공하고 있습니다. 하지만 `UIView` 에서도 이 xib연결을 어떻게 사용하는 지에 대해서 알아보겠습니다.



### nib 파일이란?

인터페이스 빌더에서 구성해놓은 여러 시각적 요소 및 비 시각적 요소에 대해서 개별적인 세팅값이나 뷰의 위치나 색상, 폰트 등의 설정을 시각적으로 구성해 놓고, 이들을 object graph로 만든 정보를 직렬화 한 파일이다



### xib 와 nib

- **NIB** = **N**xt **I**nterface **B**uilder (NXT = NextStep = NS)

- **XIB** = **X**ml **I**nterface **B**uilder

인터페이스 빌더에서 구성한 모든 정보는 XML 형태의 `.xib` 라는 확장자의 파일로 저장됩니다. 그리고 프로젝트를 컴파일 하게 되면 `.xib` 파일 역시 바이너리 파일로 컴파일 되고 그 컴파일된 결과가 `.nib` 파일이 됩니다.



## nib 로딩 - Bundle 클래스와 UINib 클래스

nib 파일을 로딩하는데는 두가지 방법이 존재합니다.  



### 1. Bundle 클래스의 loadNibNamed 사용

첫번째로 `Bundle` 클래스의 `loadNibNamed(_:owner:options:)` 메서드를 사용하는 것입니다.  이 메서드는 nib 파일 **이름**으로 찾아서 이를 로드하여 nib 파일 내의 탑 레벨 객채들을 `[Any]?` 타입으로 반환해줍니다.

```swift
extension UIView {
    func loadView(nibName: String) -> UIView? {
        let loadNib = Bundle.main.loadNibNamed(nibName, owner: self, options: nil)
        guard let view = loadNib?.first as? UIView else { return nil }
        return view
    }
}

// UIView에서 호출해서 연결
override init(frame: CGRect) {
    guard let view = loadView(nibName: String(describing: type(of: self))) else { return }
    view.frame = self.bounds
    view.autoresizingMask = [
        .flexibleWidth,
        .flexibleHeight
    ]
    
    addSubview(view)
}
```

저같은 경우에는 UIView를 extension 해서 사용하고 있습니다 xib를 여러군데서 사용할 수 있기때문입니다.

`loadNibNamed` 의 first view 를 반환하는 이유는 xib placeholder를 보게 되면 아래 여러개의 view를 둘 수 있습니다. 

![image](https://user-images.githubusercontent.com/33486820/85969331-4c3d2d00-ba02-11ea-9776-3310f0477440.png)

그리고 `UIView` 에 서 `String(describing: type(of: self)` 를 통해 nibName을 넘겨주어 연결하게 됩니다. 보통 `UIView` 와 xib의 이름을 같게 하고 위의 File's Owner 에서 `UIView` 클래싱 했기 때문에 연결이 되게 됩니다



####  nib속 CustomView 찾기

`loadNibNamed(_:owner:options:)` 가 `[Any]?` 를 반환하고 여기서 원하는 `CustomView` 를 찾을 수 있습니다.

```swift
func loadCustomView(nibName: String) -> CustomView? {
    let loadNib = Bundle.main.loadNibNamed(nibName, owner: self, options: nil)
    return objectsInNib
        .lazy
        .filter { $0 is CustomView }.first
}
```





### 2. UINIb클래스 사용

```swift
extension UIView {
    func loadView(nibName: String) -> UIView? {
        let nib = UINib(nibName: nibName, bundle: Bundle.main)
        return nib.instantiate(withOwner: self, options: nil).first as? UIView
    }
}

// UIView에서 호출해서 연결
override init(frame: CGRect) {
    guard let view = loadView(nibName: String(describing: type(of: self))) else { return }
    view.frame = self.bounds
    view.autoresizingMask = [
        .flexibleWidth,
        .flexibleHeight
    ]
    
    addSubview(view)
}
```



사용 방법은 동일합니다. `UINib` 의 `instantiate` 의 첫번째 `UIView` 를 반환하게 해주었습니다.



### 성능적인 차이

공식문서에는 이렇게 설명이 되어있습니다

> "A UINib object caches the contents of a nib file in memory, ready for unarchiving and instantiation. "
>
> UINib object는 메모리에 있는 nib 파일의내용을 캐시에 저장하고 있어서 보관하지 않고(unarchiving) 인스턴스 화할 준비가 되어있다



`UINib` 클래스를 사용하는 방법은 하나의 nib 파일의 컨텐츠에 대해서 여러개의 복사본을 만들어야할때 성능적인 측면에서 더 효율적이다고 합니다.  nib 파일을 로딩하는 프로세스는 파일 자체를 읽어오는 것 부터 시작하지만, `UINib` 인스턴스의 경우 **이미 읽어온 파일의 내용을 캐시하고 있기 때문에 디스크로 부터 파일을 읽어 들이는 오버헤드를 줄일 수 있습니다**



### 관련 메소드



#### `initWithCoder`

스토리보드 내에 정의된 VC들은 보통 코드상에서 직접 인스턴스화 하지 않습니다. 대신에 해당 Scene이 필요한 경우에 스토리보드에 의해서 자동으로 혹은 `instantiateViewController(withIdentifier:)` 를 스토리보드에게 보내서 VC의 인스턴스를 호출하게 됩니다.

> 참고: 스토리 보드는 1개 이상의 nib 파일을 담고 있는 bundle 이며, 스토리보드는 이렇게 초기화한 VC의 `nibName` 프로퍼티를 실제 nib 파일 이름으로 설정한다.



### `initWithNibName:bindle:`

`UIViewContoller` 들은 `init(nibName:bundle:)` 이라는 초기화 메소드를 지원합니다. 이 과정은 VC 인스턴스를 초기화 하고, 여기에 `nibName`속성을 부여하게 됩니다. 실질적으로 nib 파일은 View를 액세스하기 위해서 로드 하는 시점에서 읽어들이게 됩니다.

nib 파일을 읽어들이는 과정은 파일로부터 직렬화된 데이터를 읽고 이를 object graph로 복원하는 과정입니다. 이 과정에서 객체들이 온전한 인스턴스로 복원이 되고, 이를 위해서 모든 객체들이 `init(coder:)` 의 메시지를 받게 됩니다.



### `awakeFromNib`

`awakeFromNib()` 메시지는 nib 파일의 내용을 읽어서 object graph를 복원할때, **모든 복원이 완료된 후에 nib 파일 내의 모든 객체가 받게되는 메시지** 입니다. 이 메시지가 호출되는 것은 각 객체가 인스턴스화는 물론 nib 파일 내에 정의되었던 모든 속성값과 `IBOutlet` 등의 커넥션이 복원 완료되었다는 것을 의미 합니다.



1. `init(nibName:bundle:)` 은 nib 파일로부터 뷰 컨트롤러를 생성할 때 사용하며, 이 때 nib 파일은 아직 로딩되지 않는다.
2. `init(coder:)`는 nib 파일을 읽어들여서 그 데이터로부터 뷰 컨트롤러 (및 내부 뷰 들)를 생성할 때 사용되는 이니셜라이저이다.
3. `awakeFromNib()`은 모든 nib 로딩 과정이 끝난 후에 nib 파일로부터 깨어난 모든 객체들이 받게 된다.



### Refernece

- [Apple Developer - UINib](https://developer.apple.com/documentation/uikit/uinib)

- [Wireframe-Nib 파일로부터 UI 관련 객체를 로딩하기](https://soooprmx.com/archives/7226)
- https://stackoverflow.com/questions/3726400/what-is-the-difference-between-nib-and-xib-interface-builder-file-formats
- [XIB를 사용한 UIView Custom 제대로 이해하기](https://medium.com/@whitelips/xib를-사용한-uiview-custom-제대로-이해하기-348a9b789496)
