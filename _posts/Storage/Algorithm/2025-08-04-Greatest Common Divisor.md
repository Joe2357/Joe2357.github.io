---
title: "최대공약수 & 최소공배수"
author: Joe2357
categories: [Storage, Algorithm]
tags: [Algorithm]
date: 2025-08-04 12:00:00 +0900
last_modified_at: 2025-08-04 12:00:00 +0900
description: "- GCD & LCM"
math: true
---



## 최대공약수

- 두 수 $A$, $B$가 있을 때, $A$와 $B$의 *공약수*들 중 **최댓값**
  - 공약수 : $A$의 약수이면서 $B$의 약수인 수



### 구현 방법

[유클리드 호제법](https://en.wikipedia.org/wiki/Euclidean_algorithm)을 통해 구현한다.

> 두 양의 정수 $a, b~(a > b)$ 에 대하여, $a = bq + r~(0 \leq r < b)$이라 하면, $a,b$의 최대공약수는 $b,r$의 최대공약수와 같다. 즉, $\text{gcd}(a, b) = \text{gcd}(b, r)$.  
> 이 때 $r=0$이라면 $a,b$의 최대공약수는 $b$가 된다.



위의 정의를 통해, 유클리드 호제법은 아래와 같은 과정으로 반복할 수 있다. 예시로 $a=4212, b=2484$라고 하자.   


$$
\begin{aligned}
4212 &= 2484 \times 1 + 1728 \\
2484 &= 1728 \times 1 + 756 \\
1728 &= 756 \times 2 + 216 \\
756 &= 216 \times 3 + 108 \\
216 &= 108 \times 2 + 0
\end{aligned}
$$


$r=0$이 나올 때의 $b=108$이 두 수 $a=4212,~b=2484$의 최대공약수가 된다.



#### 구현 예시

```c
/* 재귀를 이용한 구현 */
int gcd(int a, int b) {
    return ((b > 0) ? (gcd(b, a % b)) : (a));
}
```

```c
/* 반복문을 이용한 구현 */
int gcd(int a, int b) {
    while(b > 0) {
        int c = a % b;
        a = b, b = c;
    }
    return a;
}
```



---



## 최소공배수

- 두 수 $A$, $B$가 있을 때, $A$와 $B$의 *공배수*들 중 **최솟값**
  - 공배수 : $A$의 배수이면서 $B$의 배수인 수



### 구현 방법

수의 성질 $a \times b = \text{gcd}(a, b) \times \text{lcm}(a,b)$ 로 구할 수 있다.

즉, 두 수의 최대공약수 $\text{gcd}$를 계산하면 최소공배수 $\text{lcm}$을 계산할 수 있다는 의미이다.