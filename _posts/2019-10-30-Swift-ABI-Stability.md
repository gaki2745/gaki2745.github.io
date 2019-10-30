---
layout: post
current: post
navigation: True
title: Swift ABI Stability
date: 2019-10-30 22:18:00
tags: Swift
class: post-template
subclass: 'post tag-fables'
author: gaki
---  

## ABI Stability(안정화)

Swift는 매 버전 마다 강조되는 이슈나 기능들이 새롭게 추가 되고있다. Swift 5.0 에서 가장 강조가 되고 있는 것이 바로 **ABI Stability(안정화)** 이다. 우선 ABI 가 무엇인지 에 대해서 부터 알아보자

> ABI : Application Binary Interface, 응용 프로그램과 운영체제, 응용 프로그램과 라이브러리 사이에 필요한 **저 수준 인터페이스** 를 정의한다.

API와 ABI를 많이 비교하는데 차이는 이렇다

- API(Appilcation Programming Interface) : 소스코드에서 사용
- ABI(Application Binary Interface) :  Binary(이진 파일)에서 사용

즉 API는 일반적으로 우리가 응용프로그램을 만들기 위해 소스코드에서 사용하는 인터페이스이고 ABI 의 경우 소스파일 보다 더 low level **이진(Binary) 파일** 로 0, 1로 만 되어진 파일들 사이에서 호환되는 인터페이스 이다. 

API가 소스코드상에서 다양한 구성요소 같은 통신 규격을 정하는 매개체이면 ABI는 기계코드에 대한 통신 규칙을 정한다고 보면 된다.

그러면 이런 low level 인 이진파일에서 호환이 이루어지는 ABI가 안정화 되는 것에 어떤 이점이 있을까?



## ABI 안정화의 목적

ABI안정화의 대표적인 예시가 윈도우 98 버전에서 윈도우XP 로 버전 업을 해도 기존의 파일이 잘 돌아 간다는 것이다. 이는 MicroSoft 에서 윈도우에 대한 ABI를 지원하기 때문이다.

Apple 의 궁극적인 목적은 이런 ABI 안정화를 지원하여 **Swift 버전간의 호환성(Compatibility)**을 제공하기 위함이란것을 알 수 있다.

Swift는 지속적인 버전업데이트가 이루어 지고 있고 그런 버전이 업데이트 될 때마다 문제점이 발생하는데 ABI를 완벽하게 지원한다면 이런 문제점이 없을 것이다. 

Swift 의 호환성을 해결하기 위해서는 두가지 과제가 존재한다.

##### **소스 호환성(Source Compatibility)**  

최신 컴파일러가 이전 버전의 Swift로 작성된 코드를 컴파일 할 수 있는 것을 말한다. 예를들어 iOS 개발자가 4.1 환경에서 프로젝트를 진행하고 있을때 Swift 5.0이 업데이트 되어도  새 버전으로 Migration 해야할 때 좀더 편의를 제공한다는 것이다. 만약 소스 호환성이 없다면 **버전 잠금(version-lock)** 에 직면하게 되는 이는 프로젝트 패키지 내부의 모든 소스코드가 동일한 Swift 버전으로 작성되어야 한다는 것을 의미한다.

소스 호환성이 있게 되면 여러 Swift 버전에서 새로운 Swift 에 대해 Migration 과 같은 추가적인 작업이 필요 없이 서로 호환이 된다.

##### **바이너리 프레임워크 & 런타임 호환성**(Binary Framework & Runtime Compatibility)

다양한 Swift 버전에서 동작할 수 있는 바이너리 형태의 프레임워크 배포를 가능하게 한다. 예를들어 GakiKit 이라는 써드 파티 프레임워크를 프로젝트에 추가했다. 이 프레임워크는 4.2 기반으로만들어 진 프레임워크고 현재 내프로젝트는 5.0 버전으로 마이그레이션 한 상태이다. 그렇게 되면 5.0 기반의 프로젝트과 기존의 4.2 버전의 프레임워크의 버전이 서로 맞지 않아 문제가 발생할 것이다.

이런 문제점을 해결하기 위해서는 바이너리 프레임워크 & 런타임 호환성가 제공되어야한다.

특히 Binary Framework에는 프레임워크 API의 소스레벨 정보를 전달하는 **Swift Module**과 런타임에 로드되는 컴파일된 구현을 제공하는 **공유라이브러리**가 모두 포함되어있다.

즉,**Binary Framework = Swift Module + 공유 라이브러리** 이다. 따라서 Binary Framework 의 호환성은 두 가지 목적을 가지고 있다.

1. **모듈 포맷 안정성(Module format stability)** : 컴파일러가 프레임워크의 공개 인터페이스를 나타내는 모듈 파일을 안정화 시킨다. 이는 API 선언과 inlineable code(프로그램 body에 추가될 수 잇는 코드)를 포함한다. 이 모듈 파일은 컴파일러가 프레임워크를 사용하는 클라이언트 코드를 컴파일 할 때 타입 검사, 코드 생성 등과 같은 필수 작업을 진행하는데 사용된다.
2. **ABI 안전성** : 다른 Swift 버전으로 컴파일 된 앱과 라이브러리 간의 Binary Compatibility(이진 호환성)을 가능하게 한다.

## ABI Stability 란?

프레임워크 또는 라이브러리들의 구성요소들은 **이진 언어** 즉 0 과 1 로 이루어진 binary language를 기반으로 통신하기 때문에 이 규칙을 서로 호환이 되어있어야 통신이 가능하다.

위의 GakiKit이라는 프레임워크 예시를 조금더 살펴 보면 Swift4.2로 binary를 생성하고 이를 Swift5.0에 연결하려고 하면 실패가 나는 이유는 그 사이에 바이너리 코드 간의 호환이 되질 않는 문제 즉 ABI가 호환이 안되어서 나는 문제이다.

결과적으로 ABI Stability 가 지원되면 ABI가 지원된 버전의 Swift 컴파일러로 컴파일한 앱과 이후 업데이트 된 Swift 컴파일러로 컴파일된 라이브러리의 바이너리 간 호환이 가능하다. 

ABI가 안정화 됐다는 말은 앞으로 ABI가 변하지 않을 것이라는 보장이다. 즉 , 바이너리 내부 처리방식은 변경이 가능하지만 인터페이스는 유지된다는 것이다. 이는 **향후 컴파일러 버전이 안정적인 ABI를 준수하는 바이너리를 생성할수 있도록, ABI를 잠그는 것(locking)을 의미** 한다.

> ABI 안정화 : 버전이 바뀌어도 최신버전의 컴파일러는 공개되어있는 **public interface**를 유지하는 한 내부 함수 호출의 호출 규칙을 자유롭게 변경이 가능하다.



## Module Stability

ABI 안정화는 **런타임** 중의 Swift버전을 혼용(Mixing)하는 것이다.  

컴파일 타임의 경우 Swift 는 "swiftmodule"이라는 불투명한 아카이브 포맷을 사용해 수동으로 작성된 헤더파일이 아닌 "MagicKit"라는 프레임워크와 같은 라이브러리 인터페이스를 나타낸다.

그러나 "swiftmodule" 포맷 역시 현재 컴파일러 버전과 맞물려 있어, 다른 버전의 Swift로 "MagicKit"를 구축하면, 개발자는 MagicKit를 import할 수 없게된다. **즉 앱 개발자와 라이브러리 작성자는 같은 버전의 컴파일러를 사용해야한다.**

>  위의 GakiKit의 개발자의 GakiKit 개발환경에 맞추어 iOS 개발자는 이 프레임워크를 사용하려면 앱 프로젝트의 버전을 프레임워크와 동일하게 해야한다는 말

이러한 제한을 제거하기 위해, 라이브러리 작성자는 "module stability"라고 불리는 기능을 구현해야한다.

![image](https://user-images.githubusercontent.com/33486820/67859510-2c95f200-fb5f-11e9-85c3-855f2a8d0bd6.png)

위의 예시 처럼 Swift 6을 사용하여 프레임워크를 만들어도 이 프레임워크 인터페이스는 Swift 6과 향후 Swift 7의 컴파일러에서도 모두 읽을 수 있도록 대비하여 구현해야한다.



## Library Evolution

현재 Swift라이브러리가 변경되면, 해당 라이브러리를 사용하는 모든 앱을 **다시 컴파일** 해야한다. 이런 방식은 컴파일러가 앱에서 사용하는 라이브러리의 정확한 버전을 알고 있기때문에 코드크기를 줄이고 앱을 더 빨리 실행 할 수 있는 추가 가정(assumptions)를 할 수 있다는 장점이 있지만 한계는 바로 **다음 버전의 라이브러리에서는 보장 받지 않을 수 있다** 는 것이다.

>  **Library Evolution** 기능은 앱을 다시 컴파일 할 필요 없이 새로운 버전의 라이브러리의 기능을 사용할 수 있도록 하는 것이다.

![](https://swift.org/assets/images/abi-stability-blog/library-evolution.png)





<hr>

## Reference

- [ABI Stability and More](https://swift.org/blog/abi-stability-and-more/)
- [Swift 5 ABI Stability](https://medium.com/swift-india/swift-5-abi-stability-769ccb986d79)
- [Zedd님의 ABI Stability](https://zeddios.tistory.com/654)
- https://pspdfkit.com/blog/2018/binary-frameworks-swift/



