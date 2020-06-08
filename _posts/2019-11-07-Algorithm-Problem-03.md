---
layout: post
title: 프로그래머스) 자물쇠와열쇠
subtitle: ""
categories: algorithm
tags: solving
comments: false
---

## 2020 카카오 블라인드 - 자물쇠와 열쇠



###### 문제 설명

고고학자인 **튜브**는 고대 유적지에서 보물과 유적이 가득할 것으로 추정되는 비밀의 문을 발견하였습니다. 그런데 문을 열려고 살펴보니 특이한 형태의 **자물쇠**로 잠겨 있었고 문 앞에는 특이한 형태의 **열쇠**와 함께 자물쇠를 푸는 방법에 대해 다음과 같이 설명해 주는 종이가 발견되었습니다.

잠겨있는 자물쇠는 격자 한 칸의 크기가 **`1 x 1`**인 **`N x N`** 크기의 정사각 격자 형태이고 특이한 모양의 열쇠는 **`M x M`** 크기인 정사각 격자 형태로 되어 있습니다.

자물쇠에는 홈이 파여 있고 열쇠 또한 홈과 돌기 부분이 있습니다. 열쇠는 회전과 이동이 가능하며 열쇠의 돌기 부분을 자물쇠의 홈 부분에 딱 맞게 채우면 자물쇠가 열리게 되는 구조입니다. 자물쇠 영역을 벗어난 부분에 있는 열쇠의 홈과 돌기는 자물쇠를 여는 데 영향을 주지 않지만, 자물쇠 영역 내에서는 열쇠의 돌기 부분과 자물쇠의 홈 부분이 정확히 일치해야 하며 열쇠의 돌기와 자물쇠의 돌기가 만나서는 안됩니다. 또한 자물쇠의 모든 홈을 채워 비어있는 곳이 없어야 자물쇠를 열 수 있습니다.

열쇠를 나타내는 2차원 배열 key와 자물쇠를 나타내는 2차원 배열 lock이 매개변수로 주어질 때, 열쇠로 자물쇠를 열수 있으면 true를, 열 수 없으면 false를 return 하도록 solution 함수를 완성해주세요.

### 제한사항

- key는 M x M(3 ≤ M ≤ 20, M은 자연수)크기 2차원 배열입니다.
- lock은 N x N(3 ≤ N ≤ 20, N은 자연수)크기 2차원 배열입니다.
- M은 항상 N 이하입니다.
- key와 lock의 원소는 0 또는 1로 이루어져 있습니다.
  - 0은 홈 부분, 1은 돌기 부분을 나타냅니다.

------

### 입출력 예

| key                               | lock                              | result |
| --------------------------------- | --------------------------------- | ------ |
| [[0, 0, 0], [1, 0, 0], [0, 1, 1]] | [[1, 1, 1], [1, 1, 0], [1, 0, 1]] | true   |

### 입출력 예에 대한 설명

![자물쇠.jpg](https://grepp-programmers.s3.amazonaws.com/files/production/469703690b/79f2f473-5d13-47b9-96e0-a10e17b7d49a.jpg)

key를 시계 방향으로 90도 회전하고, 오른쪽으로 한 칸, 아래로 한 칸 이동하면 lock의 홈 부분을 정확히 모두 채울 수 있습니다.



## 문제풀이

문제를 이해하는데 조금 어려움이 있었던 문제였다.  결론 부터 말하면 이 문제의 풀이 방법은 **key 배열을 적절하게 회전, 이동(동,서,남,북) 을 시켜 lock 배열과 일치가 되는가**  이다.

일치 라는 말이 조금 애매 하긴 key 와 lock 은 말그대로 열쇠 와 자물쇠 이다. key 배열의 1 이 돌기 부분에 해당하고 lock 배열의 0 은 홈 부분에 해당해서 이 위취가 서로 맞아야 자물쇠가 풀린다는 것이다.

그러면 문제에서도 명시 되어 있긋이 key 배열을 아래의 2가지 작업을 통해 lock 배열의 홈에 key배열의 돌기를 맞추는 것이다.

- **key 배열을 90 도 회전**
- **key 배열을 동 서 남 북 으로 한칸 밀기**



### 배열 회전 함수

````swift
// 회전한 배열을 반환
func rotate(_ key:[[Int]]) -> [[Int]] {
    var temp = [[Int]](repeating: [Int](repeating: 0, count: key.count), count: key.count)
    
    for i in 0..<key.count {
        for j in 0..<key.count {
            temp[i][j] = key[key.count - j - 1][i]
        }
    }
    return temp
}
````



### 배열 밀기 함수

```swift
// 동, 서, 남, 북 으로 한칸씩 밀어낸 배열을 반환
func move(_ key: [[Int]], _ N: Int, _ row: Int, col: Int) -> [[Int]] {
    var temp = [[Int]](repeating: [Int](repeating: 0, count: N), count: N)
    
    for i in 0..<key.count {
        for j in 0..<key.count where i + row >= 0 && i + row < N && j + col >= 0 && j + col < N {
            temp[i + row][j + col] = key[i][j]
        }
    }
    return temp
}
```

그리고 이제 탐색을 어떻게 할건지에 대해서 고민을 했다. 처음에는 한칸씩 move 하고 rotate 돌려서 문제를 풀려고 했는데 시간 초과가 발생했다. 재귀로 문제를 풀어서 실패를 한것도 있고 종료 조건이 조금 모호했다.

그래서 우선 적으로 현재 배열에서 배열 이동을 아래와 같이 반복하여 우선 모든 경우의 수로 이동을 다해 본 다음 check 하고  만약 unlock이 되지 않으면 배열을 rotate 하여 다시 check 하도록 구현했다.

```swift
func check(_ key: [[Int]], _ lock: [[Int]]) -> Bool {
    // -N ~ +N 만큼 배열을 우선 이동 시키면서 unLock이 되는지 판단한다.
    for i in 1-key.count..<lock.count {
        for j in 1-key.count..<lock.count {
            let moveMap = move(key, lock.count, i, col: j)
            if isUnlock(moveMap, lock) { return true }
        }
    }
    return false
}

func solution(_ key:[[Int]], _ lock:[[Int]]) -> Bool {
    var keyMap = key
    // 회전을 위한 반복
    for _ in 0..<4 {
        if check(keyMap, lock) {
            return true
        }
        // 0 90 180 270도 회전
        keyMap = rotate(keyMap)
    }
    
    return false
}
```

배열의 회전은 0, 90, 180, 270 네가지 경우만 있기 때문에 4번만 반복을 수행하면 된다.

시간초과가 났던 이유는 **이동->회전** 을 반복했던 탓이였다. 회전은 전체적으로 4번만 하면 된다는 것을 직시하고 문제를 풀어야한다.

<hr>

- [소스코드 바로가기](https://github.com/gaki2745/Algorithm-with-Swift/blob/master/프로그래머스/Programmers_2020카카오_자물쇠와열쇠/Programmers_2020카카오_자물쇠와열쇠/main.swift)
- 문제 출처 : [2020 KAKAO BLIND RECRUITMENT 자물쇠와 열쇠](https://programmers.co.kr/learn/courses/30/lessons/60059)
