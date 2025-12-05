---
title: "Meet in the Middle"
author: Joe2357
categories: [Storage, Algorithm]
tags: [Algorithm]
uploaded_at: 2022-09-06 12:00:00 +0900
last_modified_at: 2022-09-06 12:00:00 +0900
description: "- 중간에서 만나기"
math: true
---



## Meet in the middle 알고리즘

  - 많은 양의 정보를 처리해야하는 경우에는 시간복잡도가 매우 커지므로, 그 양을 **반으로 나누어** 각각 처리한 후 결과만을 합치는 방법
  - $n$이 커지면 커질수록 복잡도가 획기적으로 감소하는 것을 볼 수 있음



### 시간복잡도 계산

- 문제 상황 : 총 $n$개의 수 중에서 부분수열을 구해 결과를 계산하려고 함
  - 기존 브루트포스나 탐색 알고리즘을 사용하는 경우 : $O(2^n)$
  - 배열을 반으로 나누어 탐색하고 결과를 합치는 경우 : $2 \times O(2^{\frac{n}{2}}) + O(\text{결과 합치기})$
    - 대부분의 경우 결과를 합치는 복잡도는 $O(1)$ 내지 $O(n)$, 많으면 $O(n^2)$이므로 앞의 수치에 비해 무시할 수 있는 수준
- ex. $n = 40$이라면
  - 기존 방법 : $2^{40} =$ 약 1조번 연산
  - meet in the middle 방법 사용 : $2 \times 2^{\frac{40}{2}} + x \approx 2 \times 10^{6} + x$



### 구현 방법

```pseudocode
// 처리 함수
def processing(arr, start, end):
    ... get result from arr[start] to arr[end]
    return result

/* 값의 범위 */
define N -> 40
define Array[N] -> integer 1D array

// 기존 방법
processing(Array, 0, N)

// meet in the middle
processing(Array, 0, N / 2)
processing(Array, N / 2, N)
... merge result of 2 functions
```