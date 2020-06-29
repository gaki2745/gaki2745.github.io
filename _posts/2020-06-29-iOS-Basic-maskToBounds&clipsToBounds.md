---
layout: post
title: masksToBounds 와 clipsToBounds의 차이점
subtitle: ""
categories: ios
tags: basic
comments: false
---


`CALayer`의 프로퍼티 중 `cornerRadius` 를 설정하여 UIView의 모서리를 둥글게 하는 작업 도중에 subLayer 영역이 잘리는 현상을 해결하는 방법 중 하나가 masksToBounds 와 clipsToBounds 입니다. 그럼 어떤 차이가 있을가요?



### masksToBounds

- `CALayer` 의 프로퍼티
- layer의 bound에 sublayer가 잘릴지(clipping)에 대한 bool 값

  - true: sublayer가 잘리게 되어 보이지 않게 된다
  - false: sublayer영역 밖이여도 표시되게 한다



### clipsToBounds

- `UIView`의 프로퍼티 값
- subView의 내용이 잘리는 여부를 나타내는 bool값
  - true: subView가 테두리를 기준으로 잘리게된다.
  - false: 잘리지 않고 표현이된다.



결국 차이는 `CALayer`의 영역인지 `UIView` 의 영역에서의 sublayer,view 의 테두리에 대한 표시를 나타내는 bool 값입니다.

`UIView`와 `CALayer` 의 차이에 대한 설명은 아래 제 블로그 포스팅을 참고하시길 바랍니다!

[Gaki-CoreAnimation & UIView와 CALayer 관계](https://gaki2745.github.io/ios/2019/09/29/iOS-Basic-06/)



















