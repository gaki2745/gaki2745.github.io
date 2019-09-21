---
published: false
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














































