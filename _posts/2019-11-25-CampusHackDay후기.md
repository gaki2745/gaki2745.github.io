---
layout: post
current: post
navigation: True
title: 2019 Naver Campus HackDay Winter 후기
date: 2019-11-25 10:18:00
tags: 기록
class: post-template
subclass: 'post tag-fables'
author: gaki
---  

## 2019 Naver Campus HackDay Winter를 마치며

<img width="720" alt="image" src="https://user-images.githubusercontent.com/33486820/69505671-672c5800-0f6e-11ea-888c-f868eb997ae7.png">


2019년 Summer를 1회차로 Winter도 정말 좋은 기회로 참여하게 되었다. 주제는 Naver Maps 를 활용한 프로젝트였다.

선정된 프로젝트 팀 내부에서 iOS와 Android로 나뉘었고 나는 혼자서 iOS개발을 맡았다. 전적으로 iOS 개발을 맡은 것은 처음이라 긴장이 많이 되었지만 하나하나 해결해 나가는데 있어서 상당히 즐기면서 프로젝트를 임했다.



Naver Maps iOS SDK를 활용하여 내부 SDK를 분석하고 Naver Direction API를 요청하여 데이터를 밑단 작업부터 파싱해서 가져오는 작업까지, 개인공부만을 통해서는 얻을 수 없는 경험을 했다는 것에 상당히 뜻 깊었다.



그리고 내가 작성한 코드에 대해 네이버 멘토분들이 섬세하게 하나하나 코드리뷰와 피드백을 통해 **네이버는 이렇게 개발한다** 를 느낄 수 있는 정말 좋은 기회였다. 작게는 코딩스타일 부터 크게는 디자인 패턴, 아키텍쳐 패턴등 전체 프로젝트의 구조까지 배울 수 있었고 또 내가 미쳐 생각하지 못했던 부분과 커리큘럼상으로는 배울 수 없었던 **현업에서의 기술** 또한 배울 수 있어서 좋았다.



코드리뷰를 통해 받았던 리팩터링 항목 기록해 보았다.

### 리펙터링 항목

- DataModel
  - Model 구조체의 명을 좀 명확히 할 필요가 있다.
  - 불필요한 구조체 프로퍼티를 선언하지 말자-> 항상 구조체를 선언할때는 필요한 프로퍼티가 어떤 것이 있는지 명확하게 한 다음에 선언하는 것이 필요

- Request

  - 상수는 구조체 final station으로 관리 : ClientKey 값 등을 관리
  - createURLComponents : request의 urlComponent의 헤더와 파라미터를 미리 설정하여 완성된 urlComponent를 반환 
  - 고차함수 간단하게 표현하는것을 고려
  - API String Key 값도 마찬가지로
  - guard 구문시 early exit에 맞게 내부에서 추가적인 작업 할필요 없다 -> Completion으로 escaping 핸들링 필수

- MainViewController

  - 프로퍼티는 최대한 줄이자 -> 협업 시에 불필요한 프로퍼티는 혼돈을 야기한다!

  - 외부라이브러리 등을 사용할때 현재 작업이 어떤 스레드에서 실행되는지 확인하고 비동기 프로그래밍을 해야한다

  - IBOutlet didSet을 통해 설정을 할시에 해당 IBOutlet 이 nil 이 될경우 앱 크래쉬가 난다 그렇기 떄문에 함수로 따로 빼서 설정을 해주고 -> ViewDidLoad 에서 실행 시켜 세팅해주어야한다.

  - 주석을 최대한 줄이고 함수명,변수명으로 이야기하자!

  - Extension을 통해 불필요한 함수의 중복과 Extension내부에 변수를 추가하여 해당 변수로 핸들링하는 것이 효과적

  - ViewModel을 통해 View에 담길 데이터를 모두 요청하고 세팅하자 -> 이를 통해 VC의 크기와 Model 사이의 종속성을 줄일 수 있다 -> **TODO: MVVM, MVP 정리**

  - 델리게이트가 항상 겹칠수 있기 때문에 라이브러리는 필수적으로 내부에 들어가서 확인을 해야 불필요한 오류로 고생하지 않는다

    

## 결론 

항상 오류가 발생하고 문제가 날 시에는 시간이 조금 걸리더도 문제를 기록하고 해결한 뒤에 스스로 피드팩 하는 습관을 기르자

지금까지 문제가 발생했을때는 해결하기 급급했지만 이제는 되돌아보는 습관도 중요한것 같다. 개인 프로젝트와 핵데이때 한 프로젝트를 아키텍쳐 패턴등을 적용해보며 리팩터링을 통해 공부해보자!
