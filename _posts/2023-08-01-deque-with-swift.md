---
layout: post
title: Swift에서 Deque 구현하기
subtitle: 코딩테스트를 위한 O(1) 구현
author: Metishonora
categories: 문제풀이
tags: [Swift, Leetcode, 자료구조]
---

## 문제

[Leetcode #200. Number of Islands](https://leetcode.com/problems/number-of-islands)

Given an m x n 2D binary grid grid which represents a map of '1's (land) and '0's (water), return the number of islands.

An island is surrounded by water and is formed by connecting adjacent lands horizontally or vertically. You may assume all four edges of the grid are all surrounded by water.

BFS 또는 DFS를 통해 연결된 land(연결된 1들)의 개수를 찾아내면 되는 문제로, 굉장히 단순해 보이는 문제다.

## Swift에는 Deque이 없다.

그러나 C++, Java, Python등의 다른 언어와는 다르게,
Swift는 Stack, Queue 또는 Deque를 자체 지원하지 않는다. <br>
Swift의 Array 자료구조에서 지원하는 앞뒤쪽 insert(), remove()를 사용할 수 있지만,
Array의 reference를 자세히 살펴보면 문제점을 알 수 있다.

> insert(_:at:)
>> Complexity <br>
>> O(n), where n is the length of the array. [[1]]

> removeFirst()
>> Complexity <br>
>> O(n), where n is the length of the collection. [[2]]

맨 뒤쪽의 element를 삽입/제거하는 것은 O(1)으로 가능하지만, 맨 앞쪽의 element를 삽입/제거할 경우엔 O(N)이 걸리는 것을 확인할 수 있다.<br>
두 메서드를 O(1)로 최적화해야 PS에서 정상적으로 사용할 수 있을 것이다.

## Deque의 구현
이에 관해 조사하면서 얻은 결론을 정리한다. <br>
removeLast(), append(), reversed()의 시간 복잡도가 O(1)인 점에서 힌트를 얻을 수 있다. <br>

아래처럼 생긴 Queue (Array)를 생각해 보자. <Br>
![img](/assets/posts/230801-1.png) <br>

이 배열을 반으로 쪼개서, 앞쪽을 뒤집는다. 그러면 back만 2개인 배열을 만들 수 있다. <br>
뒤집는 것은 O(1)의 reversed()를 활용한다.
![img2](/assets/posts/230801-2.png) <br>

### append, appendLeft
![img3](/assets/posts/230801-3.png) <br>
left.append()와 right.append()를 사용하면
앞쪽 배열과 뒤집힌 배열에 O(1)로 원소를 추가할 수 있다.

>*(위)* right에 원소 추가<br>
>*(아래)* left에 원소 추가

두 개의 배열을 합쳐 하나의 큰 Queue로 본다면,
전체 Queue의 앞/뒤에 O(1)로 원소를 추가한 것으로 볼 수 있다.

### pop, popLeft
![img4](/assets/posts/230801-4.png) <br>
위 상황에서 전체 Queue의 앞/뒤쪽 원소를 꺼내려면 *left.removeLast()*, *right.removeLast()* 를 같은 방식으로 사용할 수 있다.

![img5](/assets/posts/230801-5.png) <br>
다만 위쪽 그림처럼 한쪽이 비어있는 경우엔 다르게 접근해야 한다.

먼저 left = right.reversed()를 진행하면 아래쪽 그림처럼 왼쪽 배열로 모든 원소를 옮길 수 있다. <br>
이후 right를 비우고, *left.removeLast()* 를 실행하면 왼쪽 배열의 마지막 원소, 즉 전체 Queue의 첫 번째 원소를 꺼낼 수 있을 것이다.

## 풀이 코드
이런 방식으로 Deque를 구현하고 BFS 및 DFS를 효율적으로 수행할 수 있다.<br>

구체적인 구현에는 generic 및 computed property를 사용하였고,
다른 문제에서도 사용할 수 있도록 일반화했다. <br>

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

## Reference
\[1] https://developer.apple.com/documentation/swift/array/insert(_:at:)-3erb3#discussion

\[2] https://developer.apple.com/documentation/swift/array/removefirst()

[1]: https://developer.apple.com/documentation/swift/array/insert(_:at:)-3erb3#discussion
[2]: https://developer.apple.com/documentation/swift/array/removefirst()