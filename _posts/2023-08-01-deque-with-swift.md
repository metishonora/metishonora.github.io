---
layout: post
title: Swift에서 Deque 구현하기
subtitle: 코딩테스트를 위한 O(1) 구현
author: Metishonora
categories: 문제풀이
tags: [Swift, Leetcode, 자료구조]
---

저는 최근에 Leetcode를 사용한 문제 풀이를 진행하고 있는데요. <br>
PS 사이트에서 새로운 언어를 활용하다 보면 시간 복잡도, 공간 복잡도를 고려하지 않을 수가 없습니다.<br>
그런 점들을 고려하다 보면 자연스럽게 라이브러리나 언어 내부 구현 등을 생각하게 되는 것 같네요.<br>
이번에는 Swift로 문제 풀이를 진행하다가 마주친 문제를 소개하려고 합니다.

## 문제

[Leetcode #200. Number of Islands](https://leetcode.com/problems/number-of-islands)

Given an m x n 2D binary grid grid which represents a map of '1's (land) and '0's (water), return the number of islands.

An island is surrounded by water and is formed by connecting adjacent lands horizontally or vertically. You may assume all four edges of the grid are all surrounded by water.

문제 자체는 굉장히 단순합니다.<br>
BFS 또는 DFS를 통해 연결된 land(연결된 1들)의 개수를 찾아내면 되는 문제.

## 진짜 문제

그러나 C++, Python과 다르게, Swift는 Stack, Queue 또는 Deque를 자체 지원하지 않습니다. <br>
Swift의 Array를 사용하면 쉽게 구현할 수 있긴 하지만, reference를 자세히 봅시다.

> insert(_:at:)
>> Complexity <br>
>> O(n), where n is the length of the array. [[1]]

> removeFirst()
>> Complexity <br>
>> O(n), where n is the length of the collection. [[2]]

맨 뒤쪽의 element를 삽입/제거하는 것은 O(1)으로 가능하지만, 맨 앞쪽의 element를 삽입/제거할 경우엔 O(N)이 걸리는 것을 확인할 수 있습니다.<br>
두 메서드를 O(1)로 최적화하려면 어떻게 구현해야 할까요?

## Deque의 구현
removeLast(), append(), reversed()의 시간 복잡도가 O(1)인 점에서 힌트를 얻을 수 있습니다. <br>
아래처럼 생긴 Queue (Array)를 생각해 봅시다. <Br>
![img](/assets/posts/230801-1.png) <br>
이 배열을 반으로 쪼개서, 앞쪽을 뒤집습니다. 그러면 back 부분만 2개로 만들 수 있습니다. <br>
![img2](/assets/posts/230801-2.png) <br>

### append, appendLeft
![img3](/assets/posts/230801-3.png) <br>
left.append()와 right.append()를 통해 전체 Queue의 앞/뒤에 O(1)로 원소를 추가할 수 있습니다.

### pop, popLeft
![img4](/assets/posts/230801-4.png) <br>
원소가 위처럼 left, right 양쪽에 차있는 경우엔 left.removeLast(), right.removeLast() 연산을 통해 전체 Queue의 앞/뒤쪽 원소를 꺼낼 수 있습니다. <br>
한쪽이 비어있을 때는 어떻게 할까요? <br>
![img5](/assets/posts/230801-5.png) <br>
reversed()가 O(1)인 점을 이용하여, left = right.reversed() 후 left.removeLast() 연산을 진행하면 O(1)의 시간복잡도로 popLeft()를 구현할 수 있습니다. <br>

## 풀이 코드
이렇게 구현한 Deque를 이용하면 BFS 및 DFS를 효율적으로 구현할 수 있습니다.<br>
저는 아래와 같이 generic 및 computed property를 사용하여 아래와 같이 구현하였고, 이 구현을 다른 문제에도 동일하게 사용하고 있습니다.<br>
원리를 이해하니 구현 코드를 달달 외우지 않아도 쉽게 구현하여 사용할 수 있었습니다.<br>
문제에 맞춰 추가적인 메서드를 구현하면 좋을 것 같네요.
```Swift
class Solution {
    class Deque<T> {
        var left: [T] = []
        var right: [T] = []
        var count: Int {
            get {
                return left.count + right.count
            }
        }
        func append(_ element: T) {
            right.append(element)
        }
        func appendLeft(_ element: T) {
            left.append(element)
        }
        func pop() -> T {
            if right.isEmpty {
                right = left.reversed()
                left = []
            }
            return right.popLast()!
        }
        func popLeft() -> T {
            if left.isEmpty {
                left = right.reversed()
                right = []
            }
            return left.popLast()!
        }
    }

    func numIslands(_ grid: [[Character]]) -> Int {
        let m = grid.count
        let n = grid[0].count
        var visited = [[Int]](repeating: [Int](repeating: 0, count: n), count: m)
        var count = 0
        var queue = Deque<[Int]>()
        let dr = [-1, 1, 0, 0]
        let dc = [0, 0, -1, 1]
        for i in 0..<m {
            for j in 0..<n {
                if grid[i][j] == "1" && visited[i][j] == 0 {
                    count += 1
                    visited[i][j] = 1
                    queue.append([i, j])
                    while queue.count > 0 {
                        var here = queue.popLeft()
                        for k in 0...3 {
                            let rr = here[0] + dr[k]
                            let cc = here[1] + dc[k]
                            if 0 <= rr && rr < m && 0 <= cc && cc < n && grid[rr][cc] == "1" && visited[rr][cc] == 0 {
                                visited[rr][cc] = 1
                                queue.append([rr, cc])
                            }
                        }
                    }
                }
            }
        }
        return count
    }
}
```

## 결과
![img6](/assets/posts/230801-6.png)

[1]: https://developer.apple.com/documentation/swift/array/insert(_:at:)-3erb3#discussion
[2]: https://developer.apple.com/documentation/swift/array/insert(_:at:)-3erb3#discussion