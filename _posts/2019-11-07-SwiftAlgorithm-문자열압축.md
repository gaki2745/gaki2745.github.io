---
layout: post
current: post
navigation: True
title: Swift 프로그래머스 문자열 압축
date: 2019-08-22 10:18:00
tags: Swift Algorithm
class: post-template
subclass: 'post tag-fables'
author: gaki
---

## 2020 카카오 블라인드 - 문자열 압축



###### 문제 설명

데이터 처리 전문가가 되고 싶은 **어피치**는 문자열을 압축하는 방법에 대해 공부를 하고 있습니다. 최근에 대량의 데이터 처리를 위한 간단한 비손실 압축 방법에 대해 공부를 하고 있는데, 문자열에서 같은 값이 연속해서 나타나는 것을 그 문자의 개수와 반복되는 값으로 표현하여 더 짧은 문자열로 줄여서 표현하는 알고리즘을 공부하고 있습니다.
간단한 예로 aabbaccc의 경우 2a2ba3c(문자가 반복되지 않아 한번만 나타난 경우 1은 생략함)와 같이 표현할 수 있는데, 이러한 방식은 반복되는 문자가 적은 경우 압축률이 낮다는 단점이 있습니다. 예를 들면, abcabcdede와 같은 문자열은 전혀 압축되지 않습니다. 어피치는 이러한 단점을 해결하기 위해 문자열을 1개 이상의 단위로 잘라서 압축하여 더 짧은 문자열로 표현할 수 있는지 방법을 찾아보려고 합니다.

예를 들어, ababcdcdababcdcd의 경우 문자를 1개 단위로 자르면 전혀 압축되지 않지만, 2개 단위로 잘라서 압축한다면 2ab2cd2ab2cd로 표현할 수 있습니다. 다른 방법으로 8개 단위로 잘라서 압축한다면 2ababcdcd로 표현할 수 있으며, 이때가 가장 짧게 압축하여 표현할 수 있는 방법입니다.

다른 예로, abcabcdede와 같은 경우, 문자를 2개 단위로 잘라서 압축하면 abcabc2de가 되지만, 3개 단위로 자른다면 2abcdede가 되어 3개 단위가 가장 짧은 압축 방법이 됩니다. 이때 3개 단위로 자르고 마지막에 남는 문자열은 그대로 붙여주면 됩니다.

압축할 문자열 s가 매개변수로 주어질 때, 위에 설명한 방법으로 1개 이상 단위로 문자열을 잘라 압축하여 표현한 문자열 중 가장 짧은 것의 길이를 return 하도록 solution 함수를 완성해주세요.

### 제한사항

- s의 길이는 1 이상 1,000 이하입니다.
- s는 알파벳 소문자로만 이루어져 있습니다.

##### 입출력 예

| s                            | result |
| ---------------------------- | ------ |
| `"aabbaccc"`                 | 7      |
| `"ababcdcdababcdcd"`         | 9      |
| `"abcabcdede"`               | 8      |
| `"abcabcabcabcdededededede"` | 14     |
| `"xababcdcdababcdcd"`        | 17     |

### 입출력 예에 대한 설명

**입출력 예 #1**

문자열을 1개 단위로 잘라 압축했을 때 가장 짧습니다.

**입출력 예 #2**

문자열을 8개 단위로 잘라 압축했을 때 가장 짧습니다.

**입출력 예 #3**

문자열을 3개 단위로 잘라 압축했을 때 가장 짧습니다.

**입출력 예 #4**

문자열을 2개 단위로 자르면 abcabcabcabc6de 가 됩니다.
문자열을 3개 단위로 자르면 4abcdededededede 가 됩니다.
문자열을 4개 단위로 자르면 abcabcabcabc3dede 가 됩니다.
문자열을 6개 단위로 자를 경우 2abcabc2dedede가 되며, 이때의 길이가 14로 가장 짧습니다.

**입출력 예 #5**

문자열은 제일 앞부터 정해진 길이만큼 잘라야 합니다.
따라서 주어진 문자열을 x / ababcdcd / ababcdcd 로 자르는 것은 불가능 합니다.
이 경우 어떻게 문자열을 잘라도 압축되지 않으므로 가장 짧은 길이는 17이 됩니다.



## 문제풀이

우선 문자열을 압축하는 경우의 수는 `1~입력문자열의 길이` 까지 압축을 해봐야한다. 예를 들어 "abcabcdede" 가 입력 값이면 1부터 길이 10 까지 압축하는 경우의 수가 10이라는 말이다. 그후면 문제가 간단해 진다.

swift에서 subString을 좀더 편하게 사용하기 위해 String 구조체를 extension 하여 subString을 반환하는 서브스크립트를 추가했다.

```swift
extension String {
    subscript(r: Range<Int>) -> String {
        let start = self.index(self.startIndex, offsetBy: r.lowerBound)
        let end = self.index(self.startIndex, offsetBy:  r.upperBound)
        
        return String(self[start..<end])
    }
}
// test
let str = "iOS Developer"

print(str[0..<3])   // iOS
print(str[4..<str.count]) // Developer
```

서브 스크립트를 활용하여 앞서 말한 `1~문자열길이` 만큼 압축을 해 나가면서 같은 것이 반복이 되면 `sameCnt` 카운트를 늘리다가 다른 것을 만나면 이전의 ` "\(sameCnt)\(beforeUnit)"` 를 최종 문자열에 추가해 나가는 방식으로 하면 된다.

이렇게 반복작업을 수행하다가 압축해야할 남은 문자열이 압축할려는 길이보다 작으면 그냥 붙이고 종료하면 된다.

- [소스코드 바로가기](https://github.com/gaki2745/Algorithm-with-Swift/blob/master/프로그래머스/Programmers_2020카카오_문자열압축/Programmers_2020카카오_문자열압축/main.swift)
- 문제 출처 : [2020 KAKAO BLIND RECRUITMENT 문자열 압축](https://programmers.co.kr/learn/courses/30/lessons/60057)

