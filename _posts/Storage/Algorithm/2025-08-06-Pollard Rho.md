---
title: "Pollard's Rho"
author: Joe2357
categories: [Storage, Algorithm]
tags: [Algorithm]
description: "- 폴라드 로 알고리즘"
math: true
---



## 폴라드 로

- 정수 $N$이 주어졌을 때, $N$의 **인수**를 찾는 알고리즘
  - <b style="color:red">주의</b> :: 알고리즘에서 반환된 인수는 <u>소인수가 아님에 유의할 것</u>
  - 소인수분해를 위한 폴라드 로 알고리즘을 위해서는 반환된 인수에 대해 **소수 판별**을 진행해야 함



### 알고리즘 증명

베이스 알고리즘으로 [플로이드 순환 찾기](https://en.wikipedia.org/wiki/Cycle_detection#Floyd's_tortoise_and_hare) 알고리즘을 활용한다. "토끼와 거북이" 알고리즘이라고도 알려져 있다.

우리의 목표를 확고히 하자. 정수 $N$이 주어졌을 때, $N$의 인수 $p$를 찾는 것이다. 찾은 인수 $p$를 또 다시 인수로 분해하는 과정을 거쳐가다보면 $N$의 모든 인수를 찾을 수 있을 것이다. 그러므로, 한 번의 step만 제대로 구현하면 될 것이다.

시작에 앞서 먼저 수식 하나를 정의하자.  


$$
f(x) = (x^2 + c) \pmod p
$$



그리고 아무 초기값 $x_0 \geq 2$를 정의한다. 이후 아래 과정을 반복한다.

- $x_1 = f(x_0)$
- $x_2 = f(x_1) = f(f(x_0))$
- $x_3 = f(x_2) = f(f(x_1)) = f(f(f(x_0)))$
- $\cdots$

이제 이 값들을 모아 수열 $\mathbf{x} = \lbrace x_1, x_2, x_3, \cdots \rbrace$를 만든다. 이렇게 하면 수열 $\mathbf{x}$는 [비둘기집 원리](https://joe2357.github.io/posts/Pigeonhole-Principle/)에 의해 **$p$번 이내에는 사이클을 형성**하기 시작한다. 즉, 언젠가는 $x_a = x_b \pmod p$인 경우가 생긴다는 것이다. 그렇게 되면 $\vert x_a - x_b \vert = 0 \pmod p$가 되므로 $p$로 나누어 떨어진다. 처음에 $N$도 $p$로 나누어 떨어진다고 가정하였으므로, $N$과 $\vert x_a - x_b \vert$는 공통된 인수 $p$를 갖게 된다!




### 구현 방법

우선 함수 $f(x)$와 초기값 $x_0$을 정의하자. 이 때 선택한 값에 따라 알고리즘의 성공 여부가 달라지기도 한다. 그래서 랜덤으로 정하는 것을 추천한다.

함수와 초기값을 정했다면, [플로이드 순환 찾기](https://en.wikipedia.org/wiki/Cycle_detection#Floyd's_tortoise_and_hare) 알고리즘을 활용하여 사이클을 구하도록 하자. 우리는 사이클의 시작지점이 아니라 **함수값이 같은 지점**만 찾으면 되므로 거북이를 1번, 토끼를 2번 옮기는 과정만 진행하면 된다. 이 과정을 반복하면서, 아래 조건을 계속 탐색한다.  



$$
\text{gcd}(\vert x_a - x_b \vert, N) > 1
$$


만약 이 조건이 참이라면, 우리는 $N$의 인수 중 하나를 $\text{gcd}$를 통해서 찾아낼 수 있다. 이 인수를 반환하면 된다. 다만 찾은 인수가 $N$이라면 인수를 찾은 것이 아니므로 탐색 실패로 판단한다. 이 때에는 함수나 초기값을 새로 설정하여 알고리즘을 다시 시작하면 된다.

다만 알고리즘 시작 전에 한가지 생각해보아야할 것이 있다. 알고리즘이 "$N$은 인수 $p$를 가지고 있다"는 가정 내에서만 진행되고 있기 때문. 즉, $N$이 소수라면 이 알고리즘이 진행되지 않는다. 그래서 시작 전에 $N$**이 소수인지** 먼저 확인하고 폴라드 로 알고리즘을 진행해야 한다. 폴라드 로 알고리즘은 **큰 수**의 인수분해가 목적이므로, 입력되는 큰 수에 대해서 소수 판별을 위해서는 $O(\sqrt{N})$ 알고리즘으로는 부족할 수 있다. 일반적으로는 [밀러-라빈 소수판별법](https://joe2357.github.io/posts/Prime-Number/#%EB%B0%80%EB%9F%AC-%EB%9D%BC%EB%B9%88-%EC%86%8C%EC%88%98%ED%8C%90%EB%B3%84%EB%B2%95)을 사용한다.



#### 구현 예시

```c
/*
폴라드 로 알고리즘
  - 입력 : 인수분해할 수 n
  - 출력 : n의 어떤 인수
*/
ll pollard_rho(ll n) {
    if (isPrime(n) == true) { // 입력된 수가 소수인지 확인
        return n;
    }

    // 변수 설정 (함수 y = x^2 + c)
    // x값의 범위 : 2 ~ n-1
    ll x = rand() % (n - 2) + 2;
    ll y = x;
    ll c = rand() % 10 + 1;
    ll g = 1;

    // 폴라드 로 알고리즘 수행
    while (g == 1) {  // 서로소가 아닐 때까지 함수 반복수행
        // 플로이드 순환 찾기 알고리즘 수행
        x = (multiply_modular(x, x, n) + c) % n;
        y = (multiply_modular(y, y, n) + c) % n;
        y = (multiply_modular(y, y, n) + c) % n;

        // 최대공약수 계산
        g = get_gcd(get_abs(x - y), n);
    }

    return g;
}
```



### 알고리즘 활용 방안

정수 $N$의 소인수분해를 위해 주로 쓰인다. 찾은 인수 $g$가 소수인지 아닌지만 추가로 판단하면 된다.

```c
/*
폴라드 로 알고리즘 (소수만 반환)
  - 입력 : 인수분해할 수 n
  - 출력 : n의 어떤 인수 (소수만 반환)
*/
ll pollard_rho_prime(ll n) {
    if (n % 2 == 0) { // 입력된 수가 짝수라면 2를 인수로 갖는다
        return 2;
    } else if (isPrime(n) == true) { // 입력된 수가 소수인지 확인
        return n;
    }

    // 변수 설정 (함수 y = x^2 + c)
    // x값의 범위 : 2 ~ n-1
    ll x = rand() % (n - 2) + 2;
    ll y = x;
    ll c = rand() % 10 + 1;
    ll g = 1;

    // 폴라드 로 알고리즘 수행
    while (g == 1) {  // 서로소가 아닐 때까지 함수 반복수행
        // 플로이드 순환 찾기 알고리즘 수행
        x = (multiply_modular(x, x, n) + c) % n;
        y = (multiply_modular(y, y, n) + c) % n;
        y = (multiply_modular(y, y, n) + c) % n;

        // 최대공약수 계산
        g = get_gcd(get_abs(x - y), n);

        if (g == n) { // 찾은 인수가 n임 => 다른 함수로 실행해야 함
            return pollard_rho_prime(n);
        }
    }

    if (isPrime(g) == true) { // 찾은 인수가 소수라면 반환
        return g;
    } else { // 아니라면 찾은 g가 합성수이므로, g의 소인수를 찾기
        return pollard_rho_prime(g);
    }
}
```



