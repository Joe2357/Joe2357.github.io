---
title: "Euclidean Algorithm"
author: Joe2357
categories: [Storage, Algorithm]
tags: [Algorithm]
uploaded_at: 2026-01-18 07:25:00 +0900
last_modified_at: 2026-01-18 07:25:00 +0900
description: "- 최대공약수와 확장 유클리드 호제법"
math: true
---



## 유클리드 호제법

**유클리드 호제법**은 두 자연수의 최대공약수(GCD, Greatest Common Divisor)를 구하는 가장 대표적인 알고리즘이다. '호제법'이란 두 수가 서로(互)를 나누어(除) 결국 원하는 수를 얻는다는 의미를 담고 있다. 이 방법은 정수론에서 소수 판별이나 서로소 판별 등 다양한 연산의 기초가 된다.

### 기본 원리

두 양의 정수 $a, b$ ($a > b$)에 대하여 $a$를 $b$로 나눈 나머지를 $r$이라고 하면, $a$와 $b$의 최대공약수는 $b$와 $r$의 최대공약수와 같다.


$$
\gcd(a, b) = \gcd(b, a \pmod{b})
$$




이 과정을 반복하여 나머지가 $0$이 되었을 때, 나누는 수(divisor)가 바로 $a$와 $b$의 최대공약수이다. 이 연산은 매우 큰 수에 대해서도 `BigInteger` 수준의 나머지 연산(Modular)을 통해 효율적으로 계산할 수 있다.



### 구현 방법

유클리드 호제법은 재귀 함수나 반복문을 사용하여 간단하게 구현할 수 있다.

#### 재귀를 이용한 방법

```c
int gcd(int a, int b) {
    if (b == 0) return a;
    return gcd(b, a % b);
}
```

#### 반복문을 이용한 방법

```c
int gcd(int a, int b) {
    while (b != 0) {
        int r = a % b;
        a = b;
        b = r;
    }
    return a;
}
```

- **시간 복잡도**: $O(\log(\min(a, b)))$
  - 숫자가 커지더라도 연산 횟수가 매우 느리게 증가하여 효율적이다.



## 확장 유클리드 호제법 (Extended Euclidean Algorithm)

**확장 유클리드 호제법**은 유클리드 호제법의 원리를 확장하여, 베주 항등식(Bézout's Identity)을 만족하는 정수해를 찾는 알고리즘이다.

### 정의

두 정수 $a, b$에 대하여 다음 식을 만족하는 정수 $x, y$가 반드시 존재한다.


$$
ax + by = \gcd(a, b)
$$


확장 유클리드 호제법은 $\gcd(a, b)$뿐만 아니라, 이를 만족하는 $x$와 $y$의 값도 함께 찾아낸다. 이는 특히 **모듈러 역원(Modular Multiplicative Inverse)**을 구하는 데 필수적인데, $p$가 소수가 아니더라도 $a$와 $m$이 서로소라면 $ax \equiv 1 \pmod{m}$을 만족하는 $x$를 찾을 수 있기 때문이다.



### 구현 방법

확장 유클리드 호제법은 재귀적으로 호출하면서 $x$와 $y$의 값을 갱신하는 방식으로 구현한다.

```c
#include <stdio.h>

typedef struct {
    int gcd;
    int x;
    int y;
} Result;

Result extended_gcd(int a, int b) {
    if (b == 0) {
        return (Result){a, 1, 0};
    }
    
    Result prev = extended_gcd(b, a % b);
    
    // x = y', y = x' - (a/b) * y'
    int x = prev.y;
    int y = prev.x - (a / b) * prev.y;
    
    return (Result){prev.gcd, x, y};
}

int main() {
    int a = 240, b = 46;
    Result res = extended_gcd(a, b);
    printf("GCD: %d, x: %d, y: %d\n", res.gcd, res.x, res.y);
    // 결과: 240*(x) + 46*(y) = gcd(240, 46)
    return 0;
}
```

### 알고리즘 활용

1. **모듈러 역원 구하기**: $ax \equiv 1 \pmod{m}$은 $ax + my = 1$로 변환 가능하므로 확장 유클리드 호제법으로 $x$를 구할 수 있다
2. **부정 방정식**: $ax + by = c$ 형태의 방정식에서 $c$가 $\gcd(a, b)$의 배수라면 정수해를 찾을 수 있다
3. **암호학**: RSA 암호화 등에서 개인키와 공개키를 생성할 때 핵심적으로 사용된다