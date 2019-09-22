---
layout: post
current: post
navigation: true
title: 'Bounds vs Frame, CGPoint, CGSize, CGRect의 개념'
date: {}
tags: iOS
class: post-template
subclass: post tag-fables
author: gaki
---
  

  

# iOS Bounds vs Frame , CGRect? CGSize? CGPoint? 



- bounds  

![image](https://user-images.githubusercontent.com/33486820/65371028-2ed07b00-dc9a-11e9-90b4-329788c70375.png)

- frame  

![image](https://user-images.githubusercontent.com/33486820/65371048-840c8c80-dc9a-11e9-9682-3a163ebd7d5a.png)

위와 같이 레이아웃 관련 코드에서 자주 등장하는 bounds와 frame을 각각 상황에 맞게  적절하게 사용하는 방법을 설명하고 정리하도록 하겠습니다.  


Frame과 Bounds는 **UIView**의 인스턴스 프로퍼티다.

![image](https://user-images.githubusercontent.com/33486820/65373529-a235b500-dcb9-11e9-8151-11be59deb8e2.png)

두개의 프로퍼티 모두 `CGRect` 타입의 값입니다. 여기서 우선 `CGRect`의 개념을 짚고 넘어가겠습니다. `CGRect`를 포함하여   
`CGSize` 와 `CGpoint`또한 필수적으로 알고 넘어가야하는 부분이기 때문에 정리하도록 하겠습니다.  


<br>

## CGPoint  

2차원 좌표계의 **점**을 포함하는 구조체이다. iOS상에서 2차원 평면의 좌표계 즉 x,y를 나타낼때 사용하는 타입이다.  

- CGPoint의 정의

```swift  
public struct CGPoint {

    
    public var x: CGFloat

    public var y: CGFloat

    public init()

    public init(x: CGFloat, y: CGFloat)
}
```  

<br>

## CGSize  

너비(Width)와 높이(Height) 값을 포함하는 구조체이다.  
<br>


![image](https://user-images.githubusercontent.com/33486820/65373638-191f7d80-dcbb-11e9-8c8d-61bb276d9be7.png)

CGSize의 경우 위와 같이 사각형으로 나타내어 설명하지만 CGSize는 **넓이와 높이의 값**이다. 그렇기 때문에 CGRect와 헷갈리면 안된다. 


- CGSize 정의

```swift  
public struct CGSize {

    
    public var width: CGFloat

    public var height: CGFloat

    public init()

    public init(width: CGFloat, height: CGFloat)
}
```  

<br>

## CGRect  

사각형의 위치(Origin) 과 크기(Size)를 포함하는 구조체이다.  

![image](https://user-images.githubusercontent.com/33486820/65373691-8f23e480-dcbb-11e9-9321-f62d00a9e2fb.png)

CGRect는 이름에서 알 수 있듯이 사각형과 관련된 값을 가지고 있는 구조체이다. 그렇기 때문에 사각형의 크기를 알고 있어야 하기 때문에 CGSize를 프로퍼티로 가지고 있고 사각형이 어떤 곳에 위치 하고 있는지 CGPoint 형의 origin(위치) 값도 가지고 있다.  


- CGRect 정의  

```swift
public struct CGRect {

    public var origin: CGPoint

    public var size: CGSize

    public init()

    public init(origin: CGPoint, size: CGSize)
}
```

<br>

## CGPoint, CGSize, CGRect의 관계  


![image](https://user-images.githubusercontent.com/33486820/65373833-90eea780-dcbd-11e9-83fd-cb79c3dde23c.png)  


<br>

CGPoint, CGSize, CGRect의 개념을 바탕으로 Bounds 와 Frame의 차이점을 정리하겠습니다.  

<br>

## Bounds, Frame의 공통점  


![image](https://user-images.githubusercontent.com/33486820/65373529-a235b500-dcb9-11e9-8151-11be59deb8e2.png)

Bounds와 Frame 모두 CGRect 구조체형 인스턴스 이기 때문에 이는 사각형으로 그려진다는 것을 의미한다.
둘다 모두 origin(CGPoint)이라는 사각형의 시작점과 size(CGSize)값의 사각형 정보를 가지고 있다는 공통점이 있다.  

<br>

## Frame  

**SuperView의 좌표시스템**안에서 View의 위치와 크기를 나타낸다.  

## Bounds  

View의 위치와 크기를 **자신만의 좌표시스템**안에서 나타낸다.  

<br>

## Frame : SuperView와의 관계  


![image](https://user-images.githubusercontent.com/33486820/65382409-7826d600-dd3f-11e9-991b-d1e4095e1bfa.png)

위의 View 상황을 가정하여 설명하겠다.  

- YellowView: SuperView
- RedView: SubView
- BlueView: RedView의 SubView  

이렇게 상황을 가정하고 Frame의 값 `origin: CGPoint` 와 `size: CGRect`의 값을 출력한 것이다.
앞서 Frame의 개념은 **SuperView의 좌표시스템 안에서**라는 말이 핵심이다.  

![image](https://user-images.githubusercontent.com/33486820/65382496-ce951400-dd41-11e9-9a35-f5aa409cb1ec.png)  

즉 위의 그림설명에서 알 수 있듯이 RedView의 경우 SuperView인 YellowView의 좌표계에서 origin좌표가 **x: 60, y: 100**만큼 떨어져있고 BlueView의 경우 YellowView에서 **x:25, y:200**이 아닌 RedView로 부터 해당 값 만큼 origin이 떨어져있다는 것을 출력을 통해 확인 할 수 있었다.  

<br>

```swift
   redView.frame.origin.x = 0
   redView.frame.origin.y = 0
        
   blueView.frame.origin.x = 0
   blueView.frame.origin.y = 0
``` 

<br>

만약 여기서 모든 origin 의 값을 0으로 바꾸게 되면 RedView의 origin  은 YellowView로 부터 0,0 만큼 마찬가지로 변화가 된 RedView의 origin에 맞추어 BlueView의 origin도 0,0 이 되어 아래와 같은 모양이 된다.  


<img width="578" alt="image" src="https://user-images.githubusercontent.com/33486820/65382529-888c8000-dd42-11e9-96db-4486f337120c.png">  


<br>


## Bounds : 자기자신만의 좌표 

마찬가지로위의 상황에서 RedView와 BludeView의 bounds의 origing을 출력해보았다.

<img width="426" alt="image" src="https://user-images.githubusercontent.com/33486820/65386258-411ee780-dd74-11e9-9ffb-102a3dae53a5.png">  


결과는 위와 같이 0,0으로 두 View 모두 동일하게 나왔다. 그 이유는 **Bounds는 자신만의 좌표시스템을 갖기 때문이다.**   
위의 상황에서 RedView의 bounds 값을 각각 50, 100 을 주면 아래와 같이 변경이된다.  

<img width="578" alt="image" src="https://user-images.githubusercontent.com/33486820/65386427-93f99e80-dd76-11e9-9d3f-497f3b801906.png">  


**bounds의 값을 변경한다는 것은 해당 origin에서 View를 다시 그린다는 의미가 된다.** 상위의 View와는 아무런 관계가 없기 때문에 RedView의 origin 만 변경이 되어 위와 같이 BlueView가 움직인 거 처럼 보인다 사실은 아래와 같이 RedView가 움직인 것이다. 우리가 보는 디바이스의View는 고정이 되어있기 때문에 위와 같이 보이는 것이다.  

![image](https://user-images.githubusercontent.com/33486820/65386602-d45a1c00-dd78-11e9-9d2f-0d803050a34c.png)  

<br>

## Frame 과 Bounds의 사용  

- Frame의 경우 UIView의 크기와 위치를 변경시켜야할때 사용하여 값을 변경 시켜준다. 

- Bounds의 경우 View 내부에 View 내부에 그림을 그릴때(DrawRect), Trasnformation 후에 View의 크기를 알고싶을때, SubView를 정렬하는 것과 같이 내부적으로 변경하는 경우에 사용한다.  


<hr>

## Reference 

- [ZediOS](https://zeddios.tistory.com/203)
- [Apple Developer](https://developer.apple.com)










