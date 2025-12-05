---
title: "Big Integer"
author: Joe2357
categories: [Storage, Data Structure]
tags: [Data Structure]
uploaded_at: 2025-10-11 15:37:56 +0900
last_modified_at: 2025-10-11 15:37:56 +0900
description: "- 큰 수 연산을 위한 구현체"
math: true
---

> 구현 언어 : C



C언어에서는 정수를 담을 수 있는 여러 자료형이 존재한다. 아래는 그 목록이다.

|        자료형        | 크기 (byte) |                             범위                             |
| :------------------: | :---------: | :----------------------------------------------------------: |
|        `char`        |     $1$     |                        $-128$ ~ $127$                        |
|   `unsigned char`    |     $1$     |                         $0$ ~ $255$                          |
|       `short`        |     $2$     |                    $-32\,768$ ~ $32\,767$                    |
|   `unsigned short`   |     $2$     |                       $0$ ~ $65\,535$                        |
|        `int`         |     $4$     |           $-2\,147\,483\,648$ ~ $2\,147\,483\,647$           |
|        `long`        |     $4$     |           $-2\,147\,483\,648$ ~ $2\,147\,483\,647$           |
|    `unsigned int`    |     $4$     |                   $0$ ~ $4\,294\,967\,295$                   |
|   `unsigned long`    |     $4$     |                   $0$ ~ $4\,294\,967\,295$                   |
|     `long long`      |     $8$     | $-9\,223\,372\,036\,854\,775\,808$ ~ $9\,223\,372\,036\,854\,775\,807$ |
| `unsigned long long` |     $8$     |           $0$ ~ $18\,446\,774\,073\,709\,551\,615$           |

이 표를 보면 음수 양수를 포함하여 정수를 담을 수 있는 가장 큰 자료형은 `long long`이다. `long long`은 $8$byte 자료형이며, $2^{63} \approx 9.2 \times 10^{18}$까지의 절댓값을 저장할 수 있다. 이는 대략 $900$경 정도 되는 값이다. 우리가 흔히 크다고 생각하는 `억`이나 `조` 단위보다도 한참 크다. 그래서 문제풀이에 사용되는 정수들의 경우, 대부분 `int` 이내에서 해결되며 숫자가 좀 커진다 싶어도 `long long` 이내에서 해결되는 경우가 많다.

하지만 이런 문제들에서는 어떨까?

- [큰 수 A + B](https://www.acmicpc.net/problem/10757) :: $0 < A, B < 10^{10\,000}$ 범위 내의 두 정수 $A$, $B$의 합을 출력하는 문제
- [긴 자리 계산](https://www.acmicpc.net/problem/2338) :: $-10^{1\,000} \leq A, B \leq 10^{1\,000}$ 범위 내의 두 정수 $A$, $B$를 입력받고, $A + B$, $A - B$, $A \times B$를 출력하는 문제
- [엄청난 부자2](https://www.acmicpc.net/problem/1271) :: $1 \leq m \leq n \leq 10^{1\,000}$ 범위 내의 두 정수 $n$, $m$을 입력받고 $\lfloor \frac{n}{m} \rfloor$, $A \mod B$를 출력하는 문제

보면 알겠지만, 여기에서 사용되는 정수들의 범위는 `long long`의 범위를 아득히 뛰어넘는다! 해서 이 문제의 해결 방법이 없을 것 같지만, 의외로 이 문제의 난이도는 가장 쉬운 난이도인 **Bronze 5**다. `Python`이라는 언어가 자체적으로 큰 수를 지원하기 때문. `Python3`로 넘어오면서 <u>숫자를 특정 블록으로 나눠 배열로 저장</u>하는 방법을 채택하여 **무제한의 범위의 숫자를 저장 가능**하다고 한다. 그렇다면, `C`에서도 숫자를 배열로 저장하는 방식을 취하면 아득히 큰 수에 대한 연산을 구현할 수 있지 않을까? 정답은 "그렇다" 이다. 저런 문제들이라도 우리는 `C`로 해결할 방법이 있다.

그렇다면 저 문제들을 어떻게 해결할까? 이 포스트에서는 `long long` 범위를 아득히 뛰어넘는 큰 수를 저장하고 연산할 수 있는 구조인 `BigInteger`를 `C`에서 구현해보려한다.



### 자료형 선언

우선 큰 수를 직접적으로 저장할 수 있도록 하는 구조를 선언해야한다. 앞서 얘기했듯이 `Python`은 수를 블록 단위로 쪼개 저장한다고 하였다. 마찬가지로, `C`에서도 숫자를 특정 크기의 블록으로 분할하여 저장하도록 하면, 배열의 길이를 늘리는 것으로 더더욱 큰 수를 저장할 수 있을 것이다. 그럼 블록 크기를 어떻게 정해야할까?

- `Python`처럼 블록 1개의 크기를 $2^{30}$으로 설정한다 :: 각 배열을 `int`로 선언할 수 있으며, **여러 자릿수의 연산을 `int` 연산 한 번으로 계산해낼 수 있다**
- 현재 수 체계처럼 블록 1개의 크기를 $10$으로 설정한다 :: 각 칸이 `char`로 표현될 수 있으며, **10진법을 사용하므로 직관적이며 이해가 쉽다**

이 포스트에서는 첫 구현이기도 하고 이해를 쉽게 하기 위해, 각 자릿수를 문자로 표현하는 방식을 채택하여 진행한다. 그럼 수를 저장하는 방법은 "문자열"이 될 것이다.

```c
typedef struct BigInteger {
    char digits[MAX_LENGTH + 1];
} BigInteger
```

이렇게 하면 `digits` 배열에 숫자를 문자열 형태로 저장하면 값의 저장과 출력에는 문제없다. 다만, 이후 연산을 구현할 때 편의를 좀 더 챙기기 위해 2가지를 더 선언하자.

- 현재 수의 길이 `length` :: 두 수의 값을 비교할 때에 사용하며, **배열의 시작점**을 쉽게 찾기 위해 저장
- 현재 수의 음수 여부 `isNegative` :: 현재 수가 음수인지 아닌지를 `bool` 형태로 저장한다. 이렇게 하면 `digits` 배열은 **숫자만 저장하면 된다**

최종적으로 선언된 구조체는 아래와 같다.

```c
typedef struct BigInteger {
    char digits[MAX_DIGITS + MORE_SPACE_FOR_INPUT];
    int length;
    bool isNegative;
} BigInteger;
```



### 함수 구현

그럼 본격적으로 큰 수를 어떻게 활용할지 구현해보자.

#### 큰 수 객체 생성

우선 실제 수를 담고 있는 변수를 생성해야한다. 우리는 문자열을 하나의 숫자로 활용하겠다고 선언하였으므로, `문자열` -> `BigInteger`로 변환하는 작업을 해주는 함수를 구현해주자. 여기서 2가지 선택지가 있는데, 어느 것이 편할지는 이후 구현할 때 세울 계획에 따라 결정될 것이다.

- 문자열 그대로 `digits[]` 배열에 저장 :: `digits[0]`에 가장 큰 자릿수, `digits[len]`에 1의 자리 숫자를 저장
  - **출력은 매우 쉽게 가능** (`printf("%s")`로 출력 가능)
  - 그러나 자릿수 연산에 어려움이 있다 (덧셈의 경우, 같은 자리의 숫자를 더해야하는데, <u>자릿수가 다른 두 수의 덧셈</u>에서는 자릿수가 같은 지점을 찾기 어렵다)
- 문자열을 뒤집어 `digits[]` 배열에 저장 :: `digits[i]`에 $10^i$번째 자릿수를 저장
  - 출력이 약간 까다롭다. 문자열을 뒤집어 <u>문자 하나하나 출력</u>해야한다
  - 대신 **자릿수 연산이 간단**하다 (같은 index라면 같은 자릿수에 있는 숫자임을 알 수 있다)

우리는 덧셈 이외에도 모든 사칙연산, 추가로 다른 연산들도 고려 중이므로 출력이 조금 어려워지더라도 **문자열을 뒤집어 저장하는 방법**을 채택하려한다. 아래는 `문자열`을 `BigInteger`로 변환하는 코드.

```c
// 문자열을 BigInteger로 변환하는 함수
BigInteger createBigInteger(const char* str) {
    BigInteger bi;
    bi.length = 0;
    bi.isNegative = false;

    int start = 0;
    int len = strlen(str);

    // 부호 판별
    if (str[0] == '-') {
        bi.isNegative = true;
        start = 1;
    }

    // 뒤에서부터 역순 저장 (1의 자리 → 10의 자리)
    for (int i = len - 1; i >= start; --i) {
        if (str[i] >= '0' && str[i] <= '9') {
            bi.digits[bi.length++] = str[i] - '0';
        }
    }
    
    return bi;
}
```

PS할 때는 가능하면 `<stdio.h>` 이외에 불필요한 include나 함수들을 사용하지 않으려하는데, 여기서는 문자열의 길이를 바로 가져오는 `<string.h>` 헤더의 `strlen()` 함수를 이용하여 코드 가독성을 약간 더 올렸다. **입력으로 음수가 들어올 수도 있기 때문**에, <u>문자열의 가장 앞을 확인</u>하여 음수 여부를 우선 판단한다. 나머지는 숫자의 시작점부터 끝까지 각 index에 대응되는 칸에 숫자를 저장하도록 하자. `strlen()`으로 숫자의 전체 길이를 미리 저장해두었기 때문에 불필요한 반복을 돌지 않아도 구현할 수 있다.



#### 큰 수 객체 출력

이제 큰 수를 담은 변수를 만들었으니, 수가 제대로 입력되었는지 확인해보자. `printBigInteger()`라는 함수를 구현하여 숫자가 제대로 출력되는지 확인한다. 출력 자체는 그렇게 어렵지 않다. 우선 **숫자의 음수 여부**를 확인하고, 가장 앞에 `-`만 붙여주면 이 수가 음수임을 표현할 수 있다. 이후 <u>문자를 저장한 방식을 반대로 출력</u>해주면 된다.

```c
// BigInteger 출력 함수
void printBigInteger(BigInteger a) {
    if (a.isNegative == true && (a.length > 1 || a.digits[0] != 0)) {
        printf("-");
    }
    for (int i = a.length - 1; i >= 0; --i) {
        printf("%d", a.digits[i]);
    }
    return;
}
```

이제 우리는 큰 수를 생성하고, 제대로 구현되었는지 디버깅할 수 있는 함수까지 구현하였다. 이제 사칙연산을 구현해보자!



#### 사칙연산 - 덧셈

[큰 수 A+B (2)](https://www.acmicpc.net/problem/15353) 문제를 확인해보자. 이전 문제와 조건이 같은 문제지만, 이번에는 **제출 언어에 제한**이 존재한다. 우리가 구현한 큰 수 객체 덧셈을 잘 테스트해볼 수 있는 문제인 셈. 이 섹션에서는 $10^{10\,000}$ 범위 내의 큰 수의 덧셈을 구현하고 테스트해보도록 하자.

사실 이 문제를 [2020년](https://www.acmicpc.net/status?from_mine=1&problem_id=15353&user_id=joe2357&top=17513584)에 푼 적이 있다. 그 때 구현했던 코드는 아래와 같다. (이 때도 문자열로 구현해서 해결했었나보다)

```c
#include <stdio.h>
#include <string.h>
#define max(a, b) (a > b) ? a : b

int main() {
    char str[2][10010] = {0};
    scanf("%s %s", str[0], str[1]);
    int result[10010] = {0}, Max = max(strlen(str[0]), strlen(str[1]));
    for (int i = 0; i < 2; i++) {
        for (int j = 0; str[i][j]; j++)
            result[Max - strlen(str[i]) + j] += (str[i][j] - '0');
        for (int j = Max; j; j--)
            if (result[j] / 10)
                result[j - 1] += result[j] / 10, result[j] %= 10;
    }
    for (int i = 0; i < Max; i++)
        printf("%d", result[i]);
    return 0;
}
```

그럼 큰 수의 덧셈은 지금 이 코드로만 가져와도 제대로 동작은 할 것이다. 다만 하나 더 추가로 고려해야하는데, **이 문제는 음수가 입력되지 않는다**!

결국 제대로 된 덧셈을 구현하기 위해서는 음수가 포함된 덧셈도 구현해야한다. 위의 코드는 <u>큰 수의 절댓값의 합</u>으로 바꿔 생각해볼 수 있을 것. 이 구현을 최대한 활용하면서 모든 case에 대한 덧셈을 커버할 수 있는지 확인해보자.

- $A \geq 0$, $B \geq 0$인 경우 :: <u>두 수의 절댓값의 합</u>을 구하고 부호를 양수로 한다
- $A \geq 0$, $B < 0$인 경우
  - $\lvert A \rvert \geq \lvert B \rvert$인 경우 :: $\lvert A \rvert - \lvert B \rvert$를 구하고 부호를 양수로 한다
  - $\lvert A \rvert < \lvert B \rvert$인 경우 :: $\lvert B \rvert - \lvert A \rvert$를 구하고 부호를 음수로 한다
- $A < 0$, $B \geq 0$인 경우
  - $\lvert A \rvert \geq \lvert B \rvert$인 경우 :: $\lvert A \rvert - \lvert B \rvert$를 구하고 부호를 음수로 한다
  - $\lvert A \rvert < \lvert B \rvert$인 경우 :: $\lvert B \rvert - \lvert A \rvert$를 구하고 부호를 양수로 한다
- $A < 0$, $B < 0$인 경우 :: <u>두 수의 절댓값의 합</u>을 구하고 부호를 음수로 한다

우리는 이미 위에서 <u>두 수의 절댓값의 합</u>을 구하는 코드를 구현해두었다. 하지만 숫자를 저장하는 방법이 지금과 반대이다. 이번에 선언한 구조체의 형식에 맞게 정제해서 아래와 같이 정리하자.

```c
// 큰 수의 절댓값을 더하여 반환
// 부호 판단 여부에는 관여하지 않음
BigInteger addAbsoluteValue(BigInteger a, BigInteger b) {
    BigInteger result;
    result.length = 0;
    result.isNegative = false;

    int carry = 0;
    int maxLen = (a.length > b.length ? a.length : b.length);

    for (int i = 0; i < maxLen || carry; ++i) {
        int sum = carry;
        if (i < a.length) {
            sum += a.digits[i];
        }
        if (i < b.length) {
            sum += b.digits[i];
        }

        result.digits[result.length++] = sum % DIGIT;
        carry = sum / DIGIT;
    }
    return result;
}
```

그러나 아직 하나가 더 남았다. <u>두 수의 절댓값의 차</u>를 구하는 코드도 필요한 것. 두 수의 차의 부호는 이미 결정되어있으므로, 이 함수에서는 **절댓값이 큰 수에서 작은 수를 뺀다**는 case 하나만 가지도록 하자. 이러면 구현이 좀 더 쉬워진다. 그러려면 <u>어떤 수가 절댓값이 더 큰지</u> 판별하는 과정도 필요할 것이다. 당연히, 이것도 함수로 구현하도록 하자.

```c
// 두 수의 절댓값을 비교하는 함수
// return [ LEFT_BIGGER = 1, RIGHT_BIGGER = -1, EQUAL = 0 ]
int compareAbsoluteValue(BigInteger a, BigInteger b) {
    if (a.length != b.length) {
        return (a.length > b.length) ? LEFT_BIGGER : RIGHT_BIGGER;
    }
    for (int i = a.length - 1; i >= 0; --i) {
        if (a.digits[i] != b.digits[i]) {
            return (a.digits[i] > b.digits[i]) ? LEFT_BIGGER : RIGHT_BIGGER;
        }
    }
    return EQUAL;
}
```

먼저 두 수의 길이를 우선적으로 비교한다. **길이가 긴 수가 절댓값이 더 큰 수**임은 자명하다. 길이가 더 긴 수가 있다면, 그 쪽이 절댓값이 더 큰 수라고 반환해주자. 만약 두 수의 길이가 같다면, 이번에는 <u>문자열을 비교하듯이</u> 가장 앞 자릿수부터 끝까지 비교해가도록 하자. 자릿수가 달라졌을 때, **자릿수가 더 큰 수가 전체적으로 더 큰 수**가 된다. 만약 끝까지 비교하였는데 결정이 나지 않았다면, <u>두 수는 같은 수</u>라고 판단할 수 있다.

이렇게 두 수의 절댓값을 비교하여 더 큰 수가 어떤 것인지 판별해냈다면, 절댓값의 뺄셈도 구현할 수 있다.

```c
// 절댓값이 큰 수에서 작은 수를 뺀 결과 반환
// 결과값은 항상 음이 아닌 정수 :: |a| >= |b| 이어야 함
BigInteger subtractAbsoluteValue(BigInteger a, BigInteger b) {
    BigInteger result;
    result.length = 0;
    result.isNegative = false;

    int borrow = 0;
    for (int i = 0; i < a.length; ++i) {
        int diff = a.digits[i] - borrow - (i < b.length ? b.digits[i] : 0);
        if (diff < 0) {
            diff += DIGIT;
            borrow = 1;
        } else {
            borrow = 0;
        }

        result.digits[result.length++] = diff;
    }

    normalize(&result);
    return result;
}
```

이렇게 되니 문제가 발생했다. 크게 2가지 문제가 발생한 것.

- `result`의 `length`가 적절히 조절되지 않는다 :: 계산 결과값이 $5\,000$이지만 `length = 6`인 경우가 발생하여, 출력값이 $005\,000$이 되는 경우가 발생
- $0$에 대한 처리가 미흡하다 :: 계산 결과값이 $0$이지만 `isNegative = true`인 경우가 발생하여, 출력값이 $-0$이 되는 경우가 발생

이에 대한 처리를 원할히 하고자, **모든 연산 이후에는 항상** `normalize()`라는 함수를 호출하여 위 문제들을 해결하도록 하자.

```c
void normalize(BigInteger* bi) {
    // 앞자리 0 제거
    while (bi->length > 1 && bi->digits[bi->length - 1] == 0) {
        bi->length--;
    }

    // 0만 남았다면 부호는 양수로 통일
    if (bi->length == 1 && bi->digits[0] == 0) {
        bi->isNegative = false;
    }
    return;
}
```

자, 이제 모든 util을 구현하였으니 최종적으로 큰 수의 덧셈을 구현하는 코드를 완성시켜보자.

```c
// 큰 수의 덧셈 함수
// 결과값 :: a + b
BigInteger add(BigInteger a, BigInteger b) {
    BigInteger result;

    if (a.isNegative == b.isNegative) {
        // 같은 부호 -> 절댓값 덧셈 후 부호 유지
        result = addAbsoluteValue(a, b);
        result.isNegative = a.isNegative;
    } else {
        // 부호 다름 -> 절댓값 비교 후 큰 쪽 - 작은 쪽
        int cmp = compareAbsoluteValue(a, b);
        if (cmp == EQUAL) {
            result = createBigInteger("0");
        } else if (cmp == LEFT_BIGGER) {
            result = subtractAbsoluteValue(a, b);
            result.isNegative = a.isNegative;
        } else { // cmp == RIGHT_BIGGER
            result = subtractAbsoluteValue(b, a);
            result.isNegative = b.isNegative;
        }
    }
    normalize(&result);
    return result;
}
```



#### 사칙연산 - 뺄셈

이정도 구현했으면 뺄셈은 간단하다. 오히려 덧셈을 구현하면서 **이미 모든 구현은 완료되어있다!** 덧셈을 진행할 때 `add()` 함수에서 부호에 따라 제대로 된 답이 반환될 수 있도록 case work를 진행한 것을 기억하는가? 뺄셈도 똑같이 구현할 수 있다! $B$의 부호만 바뀌었다뿐이지, 부호 바꾸고 덧셈하는 것과 완전히 같으니 덧셈과 같은 방식으로 구현하면 된다.

```c
// 큰 수의 뺄셈 함수
// 결과값 :: a - b
BigInteger subtract(BigInteger a, BigInteger b) {
    BigInteger result;

    if (a.isNegative == false && b.isNegative == false) { // (+a) - (+b)
        int cmp = compareAbsoluteValue(a, b);
        if (cmp == EQUAL) {
            result = createBigInteger("0");
        } else if (cmp == LEFT_BIGGER) {
            result = subtractAbsoluteValue(a, b);
            result.isNegative = false;
        } else { // RIGHT_BIGGER
            result = subtractAbsoluteValue(b, a);
            result.isNegative = true;
        }
    } else if (a.isNegative == false && b.isNegative == true) { // (+a) - (-b) = a + |b|
        result = addAbsoluteValue(a, b);
        result.isNegative = false;
    } else if (a.isNegative == true && b.isNegative == false) { // (-a) - (+b) = -( |a| + |b| )
        result = addAbsoluteValue(a, b);
        result.isNegative = true;
    } else { // (-a) - (-b) = b - a
        int cmp = compareAbsoluteValue(a, b);
        if (cmp == EQUAL) {
            result = createBigInteger("0");
        } else if (cmp == LEFT_BIGGER) {
            result = subtractAbsoluteValue(a, b);
            result.isNegative = true;
        } else { // RIGHT_BIGGER
            result = subtractAbsoluteValue(b, a);
            result.isNegative = false;
        }
    }
    normalize(&result);
    return result;
}
```



#### 사칙연산 - 곱셈

[긴 자리 계산](https://www.acmicpc.net/problem/2338) 문제가 있었다. 큰 수의 덧셈, 뺄셈, **곱셈**을 구현하는 문제였다. 위에서 덧셈과 뺄셈을 구현하였으니, 곱셈을 구현하면 이 문제를 `C`로 해결할 수 있다! 덧셈과 같은 방식으로 구현할 수 있으며, 각 수의 부호를 굳이 비교할 필요가 없기 때문에 오히려 더 간단할 수도 있다.

각 자릿수를 알고 있으므로, 두 수의 곱셈 $A \times B$는 [곱셈](https://www.acmicpc.net/problem/2588) 문제의 방식으로 구현하면 된다. $B$의 모든 자릿수 $B[i]$에 대해 $A \times B = \sum_{i=0}^{len(B)} A \times B[i] \times 10^{i}$로 계산할 수 있다. 코드로 옮기면 아래와 같다.

```c
// 큰 수의 곱셈 함수
// 결과값 :: a * b
BigInteger multiply(BigInteger a, BigInteger b) {
    BigInteger result;
    memset(result.digits, 0, sizeof(result.digits));
    result.length = a.length + b.length;
    result.isNegative = (a.isNegative != b.isNegative);

    for (int i = 0; i < a.length; ++i) {
        int carry = 0;
        for (int j = 0; j < b.length || carry; ++j) {
            long long cur = result.digits[i + j] +
                            (long long)a.digits[i] * (j < b.length ? b.digits[j] : 0) + carry;
            result.digits[i + j] = (char)(cur % DIGIT);
            carry = (int)(cur / DIGIT);
        }
    }
    normalize(&result);
    return result;
}
```

이렇게 덧셈, 뺄셈, 곱셈을 모두 구현하였다! [제출 :: 긴 자리 계산](https://www.acmicpc.net/status?from_mine=1&problem_id=2338&user_id=joe2357&top=99342030)에서도 통과한 것을 확인할 수 있다. 현재까지 우리는 $10^{1\,000}$ 범위의 큰 수에 대해 **덧셈, 뺄셈, 곱셈**까지 계산할 수 있는 프로그램을 만들었다.

그럼 더 큰 범위에서는 어떨까? [큰 수 곱셈](https://www.acmicpc.net/problem/13277) 문제를 확인해보자. 이 문제에서 수의 범위는 최대 $10^{300\,000}$이다. 당연하게도 위에서 구현한 방법은 <u>시간 초과</u>를 받는다. 위 알고리즘의 구현은 $O(N^2)$의 시간복잡도를 가지기 때문. 하지만 이 이상의 좋은 알고리즘을 배우지는 못했기에 검색을 통해 더 좋은 방법들을 탐색해보자.



##### 곱셈 개선 - [카라추바 알고리즘](https://joe2357.github.io/posts/Karatsuba/)

찾은 알고리즘 중에 곱셈을 빠르게 하기 위한 알고리즘으로 카라추바 알고리즘이 있다고 한다. 기존의 $O(N^2)$의 복잡도를 $O(N^{\log 3})$으로 줄일 수 있는 알고리즘이라고 한다. $10^{300\,000}$ 범위의 수를 곱셈할 때 기존 방법은 $900$억번의 연산이 필요하다고 본다면, 카라추바 알고리즘은 대략 $4.5$억번의 연산으로 계산할 수 있다고 한다. [큰 수 곱셈](https://www.acmicpc.net/problem/13277) 문제의 제한시간은 3초이므로 시간 내에 실행될 것이라는 기대는 크게 하지 않지만 시도해보자.

```c
// 큰 수의 곱셈 함수 (Karatsuba 알고리즘)
// 결과값 :: a * b
BigInteger multiply(BigInteger a, BigInteger b) {
    normalize(&a);
    normalize(&b);

    int n = (a.length > b.length ? a.length : b.length);
    if (n <= KARATSUBA_THRESHOLD) { // 임계값 이하에서는 단순 곱셈
        return multiply_N2(a, b);
    }

    int m = n / 2;

    // low/high 분할
    BigInteger a_low, a_high, b_low, b_high;
    memset(&a_low, 0, sizeof(a_low));
    memset(&a_high, 0, sizeof(a_high));
    memset(&b_low, 0, sizeof(b_low));
    memset(&b_high, 0, sizeof(b_high));

    a_low.length = (a.length < m ? a.length : m);
    b_low.length = (b.length < m ? b.length : m);

    for (int i = 0; i < a_low.length; ++i) {
        a_low.digits[i] = a.digits[i];
    }
    for (int i = m; i < a.length; ++i) {
        a_high.digits[a_high.length++] = a.digits[i];
    }

    for (int i = 0; i < b_low.length; ++i) {
        b_low.digits[i] = b.digits[i];
    }
    for (int i = m; i < b.length; ++i) {
        b_high.digits[b_high.length++] = b.digits[i];
    }
    // 재귀 호출
    BigInteger z0 = multiply(a_low, b_low);
    BigInteger z2 = multiply(a_high, b_high);

    BigInteger a_sum = addAbsoluteValue(a_low, a_high);
    BigInteger b_sum = addAbsoluteValue(b_low, b_high);
    BigInteger z1 = multiply(a_sum, b_sum);
    z1 = subtract(z1, z0);
    z1 = subtract(z1, z2);

    // 결과 조합: z2 * 10^(2m) + z1 * 10^m + z0
    BigInteger res1 = shiftLeft(z2, 2 * m);
    BigInteger res2 = shiftLeft(z1, m);
    BigInteger result = add(add(res1, res2), z0);

    result.isNegative = (a.isNegative != b.isNegative);
    normalize(&result);
    return result;
}
```

하지만 아쉽게도 <strong style="color:red;">시간초과</strong>를 피할 수는 없었다. 더 효율적인 방법이 있나 찾아보니 [FFT 알고리즘](https://en.wikipedia.org/wiki/Fast_Fourier_transform)이 있다고 한다. 시간복잡도가 $O(N \log N)$이라고 하니 $N=300\,000$일 때 대략 $164$만 번의 연산으로 곱셈을 구할 수 있다고 한다. 이는 1초 이내에도 충분히 돌 정도의 연산 횟수이므로, 나중에 FFT를 배우면 구현해봐야겠다.



##### 곱셈 개선 - [FFT 알고리즘](https://en.wikipedia.org/wiki/Fast_Fourier_transform)

> TODO



#### 사칙연산 - 나눗셈

곱셈까지 구현했으니, 이제 큰 수의 나눗셈을 구현해보자. 일반 정수형 나눗셈은 CPU 명령어 한 줄로 수행되지만, 큰 수(BigInteger)는 직접 자리 단위로 시뮬레이션해야 한다.

나눗셈 알고리즘은 우리가 초등학교 때 배운 **긴 나눗셈(long division)** 과 동일한 원리로 동작한다. 즉, **피제수 $a$의 가장 높은 자리부터 차례대로 내려오면서**, 현재까지의 나머지(`remainder`)를 10배(`shiftLeft`) 하고 새로운 자릿수를 더해가며 몫의 각 자리수를 구한다.

우선 나머지를 활용한 나눗셈을 구현하기 위해  큰 수에 $10^m$배 연산을 수행하는 `shiftLeft()` 함수를 구현하자. 정수형의 bit shift와 유사하게 진행한다고 생각하면 이해하기 쉽다.

```c
// m자리 수만큼 왼쪽으로 시프트 (10^m 곱셈 효과)
// a가 0일 경우 그대로 반환
BigInteger shiftLeft(BigInteger a, int m) {
    if (a.length == 1 && a.digits[0] == 0) {
        return a;
    }
    for (int i = a.length - 1; i >= 0; --i) {
        a.digits[i + m] = a.digits[i];
    }
    for (int i = 0; i < m; ++i) {
        a.digits[i] = 0;
    }
    a.length += m;
    return a;
}
```

이후 맨 앞자리부터 나머지를 활용해 하나씩 나눠가며 몫을 구하는 함수 `divide()`를 구현하며 나눗셈 연산 구현을 완료하자.

```c
// 큰 수의 나눗셈 함수 (몫)
// 결과값 :: a / b
BigInteger divide(BigInteger a, BigInteger b) {
    BigInteger quotient, remainder;
    memset(&quotient, 0, sizeof(quotient));
    memset(&remainder, 0, sizeof(remainder));
    quotient.length = 0;
    remainder.length = 1;
    remainder.digits[0] = 0;
    quotient.isNegative = (a.isNegative != b.isNegative);
    remainder.isNegative = false;

    // 0 나눗셈 방지
    if (b.length == 1 && b.digits[0] == 0) {
        printf("[Error] Division by zero\n");
        return createBigInteger("0");
    }

    // 절댓값만으로 연산
    a.isNegative = false;
    b.isNegative = false;

    // 나눗셈 본체 (상위 자리부터)
    for (int i = a.length - 1; i >= 0; --i) {
        // remainder = remainder * 10 + a.digits[i]
        if (remainder.length != 1 || remainder.digits[0] != 0) {
            remainder = shiftLeft(remainder, 1);
        }
        remainder.digits[0] = a.digits[i];
        if (remainder.length == 1 && remainder.digits[0] == 0) {
            remainder.length = 1;
        } else {
            normalize(&remainder);
        }

        // q_digit = remainder / b (0~9 사이)
        int q_digit = 0;
        while (compareAbsoluteValue(remainder, b) != RIGHT_BIGGER) {
            remainder = subtractAbsoluteValue(remainder, b);
            q_digit++;
        }
        quotient.digits[i] = q_digit;
    }

    // quotient의 길이 계산
    quotient.length = a.length;
    normalize(&quotient);

    return quotient;
}
```



#### 연산 - 모듈러

나눗셈이 구현되었으니 나머지 연산도 바로 구현할 수 있다. 사실 나눗셈을 구현하기 위해 사용되던 `remainder`가 나머지값이므로 그대로 반환해주어도 된다. 하지만 기능 분할을 위해 몫과 나머지는 따로 구하는 것으로 구현하였다.

나머지를 계산하는 방법은 간단하다. 어떤 수 $y$를 $x$로 나눈 몫이 $Q$라고 할 때, $y = Qx + r$이 될 것이고, 이 때 $r$이 나머지가 될 것이다. $r$을 제외한 항들을 넘겨 정리하면 $r = y - Qx$가 되므로, `divide(y, x)`의 결과 $Q$를 계산한 다음 `r = subtract(y, multiply(Q, x))` 하면 나머지를 얻을 수 있다. 하지만 이 때 **나머지가 음수인 경우**가 간혹 존재한다. $x$가 음수였다던가 하는 이유들에 의해, `C`에서 있는 경우와 마찬가지로 등장할 수 있다. 실제로 `C`에서 $-13 \mod 4$는 $3$이 아닌 $-1$로 반환된다.

이를 위해 `normalize()` 전에 나머지가 $0 \leq r < x$의 범위 안에 있는지 확인하고, 없다면 $x$를 더하거나 빼서 나머지값을 보정해주어야한다.

```c
// 큰 수의 나눗셈 함수 (나머지)
// 결과값 :: a % b
BigInteger modular(BigInteger a, BigInteger b) {
    BigInteger q = divide(a, b);
    BigInteger qb = multiply(q, b);
    BigInteger r = subtract(a, qb); // r = a - q*b

    // 나머지값 보정
    BigInteger babs = b;
    babs.isNegative = false;
    if (r.isNegative == true) {
        r = add(r, babs);
    }
    while (compareAbsoluteValue(r, babs) != RIGHT_BIGGER) {
        r = subtractAbsoluteValue(r, babs);
    }
    return r;
}
```



#### 연산 - 거듭제곱

거듭제곱 연산은 곱셈을 여러번 반복하는 연산이다. 어떤 수 $N$을 $X$제곱을 하기 위해서는 큰 수 곱셈 연산을 $X$번 반복하면 된다.

```c
// 큰 수의 거듭제곱 함수
// 결과값 :: base ^ exp (exp는 0 이상의 정수)
BigInteger pow(BigInteger base, int exp) {
    BigInteger result = createBigInteger("1");

    bool resultNegative = false;
    if (base.isNegative == true && (exp % 2 == 1)) {
        resultNegative = true;
    }
    base.isNegative = false;

    for (int i = 0; i < exp; ++i) {
        result = multiply(result, base);
    }

    result.isNegative = resultNegative;
    normalize(&result);
    return result;
}
```

하지만 이 경우 **시간이 매우 오래 걸린다**는 단점이 있다. $10^{300\,000}$ 범위의 두 숫자를 곱하는 시간도 3초를 뛰어넘는데, 그 과정을 $X$번 반복하라는 것은 프로그램을 종료시키지 않겠다는 의미로도 들린다. 거듭제곱을 더 빠르게 수행해야한다.

물론 우리는 이미 방법을 하나 알고 있다. 바로 [분할정복 알고리즘](https://en.wikipedia.org/wiki/Divide_and_conquer)을 활용한 거듭제곱 연산. 제곱하는 횟수 $X$를 절반씩 줄여나가기 때문에 빠른 시간 내에 거듭제곱 연산을 완료할 수 있다. $O(N \times \text{곱셈 연산})$이 들던 기존의 방법에서 $O(\log N \times \text{곱셈 연산})$으로 복잡도를 줄일 수 있다.

```c
// 큰 수의 거듭제곱 함수 (빠른 제곱법)
// 결과값 :: base ^ exp (exp는 0 이상의 정수)
BigInteger pow(BigInteger base, int exp) {
    BigInteger result = createBigInteger("1");

    bool resultNegative = false;
    if (base.isNegative == true && (exp % 2 == 1)) {
        resultNegative = true;
    }
    base.isNegative = false;

    while (exp > 0) {
        if (exp % 2 == 1) {
            result = multiply(result, base);
        }
        base = multiply(base, base);
        exp /= 2;
    }

    result.isNegative = resultNegative;
    normalize(&result);
    return result;
}
```



### 소스코드

```c
#include <stdio.h>
#include <string.h>

typedef char bool;
const bool true = 1;
const bool false = 0;

#define MAX_DIGITS (int)(2e3 + 1)
#define MORE_SPACE_FOR_INPUT 10
const int DIGIT = 10;

const int LEFT_BIGGER = 1;
const int RIGHT_BIGGER = -1;
const int EQUAL = 0;

const int KARATSUBA_THRESHOLD = 128; // Karatsuba 알고리즘 임계값

// ===== 구조체 정의 =====
typedef struct BigInteger {
    char digits[MAX_DIGITS + MORE_SPACE_FOR_INPUT]; // 각 자리 숫자 (digit[i] = 10^i 자리 숫자)
    int length;                                        // 숫자의 길이
    bool isNegative;                                // 음수 여부
} BigInteger;

/* ===== 보조 함수 섹션 ===== */
void normalize(BigInteger* bi) {
    // 앞자리 0 제거
    while (bi->length > 1 && bi->digits[bi->length - 1] == 0) {
        bi->length--;
    }

    // 0만 남았다면 부호는 양수로 통일
    if (bi->length == 1 && bi->digits[0] == 0) {
        bi->isNegative = false;
    }
    return;
}

// 문자열을 BigInteger로 변환하는 함수
BigInteger createBigInteger(const char* str) {
    BigInteger bi;
    bi.length = 0;
    bi.isNegative = false;

    int start = 0;
    int len = strlen(str);

    // 부호 판별
    if (str[0] == '-') {
        bi.isNegative = true;
        start = 1;
    }

    // 뒤에서부터 역순 저장 (1의 자리 → 10의 자리)
    for (int i = len - 1; i >= start; --i) {
        if (str[i] >= '0' && str[i] <= '9') {
            bi.digits[bi.length++] = str[i] - '0';
        }
    }

    normalize(&bi);
    return bi;
}

// 큰 수의 절댓값을 더하여 반환
// 부호 판단 여부에는 관여하지 않음
BigInteger addAbsoluteValue(BigInteger a, BigInteger b) {
    BigInteger result;
    result.length = 0;
    result.isNegative = false;

    int carry = 0;
    int maxLen = (a.length > b.length ? a.length : b.length);

    for (int i = 0; i < maxLen || carry; ++i) {
        int sum = carry;
        if (i < a.length) {
            sum += a.digits[i];
        }
        if (i < b.length) {
            sum += b.digits[i];
        }

        result.digits[result.length++] = sum % DIGIT;
        carry = sum / DIGIT;
    }
    normalize(&result);
    return result;
}

// 절댓값이 큰 수에서 작은 수를 뺀 결과 반환
// 결과값은 항상 음이 아닌 정수 :: |a| >= |b| 이어야 함
BigInteger subtractAbsoluteValue(BigInteger a, BigInteger b) {
    BigInteger result;
    result.length = 0;
    result.isNegative = false;

    int borrow = 0;
    for (int i = 0; i < a.length; ++i) {
        int diff = a.digits[i] - borrow - (i < b.length ? b.digits[i] : 0);
        if (diff < 0) {
            diff += DIGIT;
            borrow = 1;
        } else {
            borrow = 0;
        }

        result.digits[result.length++] = diff;
    }

    normalize(&result);
    return result;
}

// 두 수의 절댓값을 비교하는 함수
// return [ LEFT_BIGGER = 1, RIGHT_BIGGER = -1, EQUAL = 0 ]
int compareAbsoluteValue(BigInteger a, BigInteger b) {
    if (a.length != b.length) {
        return (a.length > b.length) ? LEFT_BIGGER : RIGHT_BIGGER;
    }
    for (int i = a.length - 1; i >= 0; --i) {
        if (a.digits[i] != b.digits[i]) {
            return (a.digits[i] > b.digits[i]) ? LEFT_BIGGER : RIGHT_BIGGER;
        }
    }
    return EQUAL;
}

// m자리 수만큼 왼쪽으로 시프트 (10^m 곱셈 효과)
// a가 0일 경우 그대로 반환
BigInteger shiftLeft(BigInteger a, int m) {
    if (a.length == 1 && a.digits[0] == 0) {
        return a;
    }
    for (int i = a.length - 1; i >= 0; --i) {
        a.digits[i + m] = a.digits[i];
    }
    for (int i = 0; i < m; ++i) {
        a.digits[i] = 0;
    }
    a.length += m;
    return a;
}
/* ======================================== */

/* ===== 산술 연산 함수 섹션 ===== */
// 큰 수의 덧셈 함수
// 결과값 :: a + b
BigInteger add(BigInteger a, BigInteger b) {
    BigInteger result;

    if (a.isNegative == b.isNegative) {
        // 같은 부호 -> 절댓값 덧셈 후 부호 유지
        result = addAbsoluteValue(a, b);
        result.isNegative = a.isNegative;
    } else {
        // 부호 다름 -> 절댓값 비교 후 큰 쪽 - 작은 쪽
        int cmp = compareAbsoluteValue(a, b);
        if (cmp == EQUAL) {
            result = createBigInteger("0");
        } else if (cmp == LEFT_BIGGER) {
            result = subtractAbsoluteValue(a, b);
            result.isNegative = a.isNegative;
        } else { // cmp == RIGHT_BIGGER
            result = subtractAbsoluteValue(b, a);
            result.isNegative = b.isNegative;
        }
    }
    normalize(&result);
    return result;
}

// 큰 수의 뺄셈 함수
// 결과값 :: a - b
BigInteger subtract(BigInteger a, BigInteger b) {
    BigInteger result;

    if (a.isNegative == false && b.isNegative == false) { // (+a) - (+b)
        int cmp = compareAbsoluteValue(a, b);
        if (cmp == EQUAL) {
            result = createBigInteger("0");
        } else if (cmp == LEFT_BIGGER) {
            result = subtractAbsoluteValue(a, b);
            result.isNegative = false;
        } else { // RIGHT_BIGGER
            result = subtractAbsoluteValue(b, a);
            result.isNegative = true;
        }
    } else if (a.isNegative == false && b.isNegative == true) { // (+a) - (-b) = a + |b|
        result = addAbsoluteValue(a, b);
        result.isNegative = false;
    } else if (a.isNegative == true && b.isNegative == false) { // (-a) - (+b) = -( |a| + |b| )
        result = addAbsoluteValue(a, b);
        result.isNegative = true;
    } else { // (-a) - (-b) = b - a
        int cmp = compareAbsoluteValue(a, b);
        if (cmp == EQUAL) {
            result = createBigInteger("0");
        } else if (cmp == LEFT_BIGGER) {
            result = subtractAbsoluteValue(a, b);
            result.isNegative = true;
        } else { // RIGHT_BIGGER
            result = subtractAbsoluteValue(b, a);
            result.isNegative = false;
        }
    }
    normalize(&result);
    return result;
}

// 큰 수의 곱셈 함수 (짧은 숫자들의 곱셈)
// 결과값 :: a * b
BigInteger multiply_N2(BigInteger a, BigInteger b) {
    BigInteger result;
    memset(result.digits, 0, sizeof(result.digits));
    result.length = a.length + b.length;
    result.isNegative = (a.isNegative != b.isNegative);

    for (int i = 0; i < a.length; ++i) {
        int carry = 0;
        for (int j = 0; j < b.length || carry; ++j) {
            long long cur = result.digits[i + j] +
                            (long long)a.digits[i] * (j < b.length ? b.digits[j] : 0) + carry;
            result.digits[i + j] = (char)(cur % DIGIT);
            carry = (int)(cur / DIGIT);
        }
    }
    normalize(&result);
    return result;
}

// 큰 수의 곱셈 함수 (Karatsuba 알고리즘)
// 결과값 :: a * b
// TODO :: 백준 13277번 시간초과 해결
BigInteger multiply(BigInteger a, BigInteger b) {
    normalize(&a);
    normalize(&b);

    int n = (a.length > b.length ? a.length : b.length);
    if (n <= KARATSUBA_THRESHOLD) { // 임계값 이하에서는 단순 곱셈
        return multiply_N2(a, b);
    }

    int m = n / 2;

    // low/high 분할
    BigInteger a_low, a_high, b_low, b_high;
    memset(&a_low, 0, sizeof(a_low));
    memset(&a_high, 0, sizeof(a_high));
    memset(&b_low, 0, sizeof(b_low));
    memset(&b_high, 0, sizeof(b_high));

    a_low.length = (a.length < m ? a.length : m);
    b_low.length = (b.length < m ? b.length : m);

    for (int i = 0; i < a_low.length; ++i) {
        a_low.digits[i] = a.digits[i];
    }
    for (int i = m; i < a.length; ++i) {
        a_high.digits[a_high.length++] = a.digits[i];
    }

    for (int i = 0; i < b_low.length; ++i) {
        b_low.digits[i] = b.digits[i];
    }
    for (int i = m; i < b.length; ++i) {
        b_high.digits[b_high.length++] = b.digits[i];
    }
    // 재귀 호출
    BigInteger z0 = multiply(a_low, b_low);
    BigInteger z2 = multiply(a_high, b_high);

    BigInteger a_sum = addAbsoluteValue(a_low, a_high);
    BigInteger b_sum = addAbsoluteValue(b_low, b_high);
    BigInteger z1 = multiply(a_sum, b_sum);
    z1 = subtract(z1, z0);
    z1 = subtract(z1, z2);

    // 결과 조합: z2 * 10^(2m) + z1 * 10^m + z0
    BigInteger res1 = shiftLeft(z2, 2 * m);
    BigInteger res2 = shiftLeft(z1, m);
    BigInteger result = add(add(res1, res2), z0);

    result.isNegative = (a.isNegative != b.isNegative);
    normalize(&result);
    return result;
}

// 큰 수의 나눗셈 함수 (몫)
// 결과값 :: a / b
BigInteger divide(BigInteger a, BigInteger b) {
    BigInteger quotient, remainder;
    memset(&quotient, 0, sizeof(quotient));
    memset(&remainder, 0, sizeof(remainder));
    quotient.length = 0;
    remainder.length = 1;
    remainder.digits[0] = 0;
    quotient.isNegative = (a.isNegative != b.isNegative);
    remainder.isNegative = false;

    // 0 나눗셈 방지
    if (b.length == 1 && b.digits[0] == 0) {
        printf("[Error] Division by zero\n");
        return createBigInteger("0");
    }

    // 절댓값만으로 연산
    a.isNegative = false;
    b.isNegative = false;

    // 나눗셈 본체 (상위 자리부터)
    for (int i = a.length - 1; i >= 0; --i) {
        // remainder = remainder * 10 + a.digits[i]
        if (remainder.length != 1 || remainder.digits[0] != 0) {
            remainder = shiftLeft(remainder, 1);
        }
        remainder.digits[0] = a.digits[i];
        if (remainder.length == 1 && remainder.digits[0] == 0) {
            remainder.length = 1;
        } else {
            normalize(&remainder);
        }

        // q_digit = remainder / b (0~9 사이)
        int q_digit = 0;
        while (compareAbsoluteValue(remainder, b) != RIGHT_BIGGER) {
            remainder = subtractAbsoluteValue(remainder, b);
            q_digit++;
        }
        quotient.digits[i] = q_digit;
    }

    // quotient의 길이 계산
    quotient.length = a.length;
    normalize(&quotient);

    return quotient;
}

// 큰 수의 나눗셈 함수 (나머지)
// 결과값 :: a % b
BigInteger modular(BigInteger a, BigInteger b) {
    BigInteger q = divide(a, b);
    BigInteger qb = multiply(q, b);
    BigInteger r = subtract(a, qb); // r = a - q*b

    // 나머지값 보정
    BigInteger babs = b;
    babs.isNegative = false;
    if (r.isNegative == true) {
        r = add(r, babs);
    }
    while (compareAbsoluteValue(r, babs) != RIGHT_BIGGER) {
        r = subtractAbsoluteValue(r, babs);
    }
    return r;
}

// 큰 수의 거듭제곱 함수 (빠른 제곱법)
// 결과값 :: base ^ exp (exp는 0 이상의 정수)
BigInteger pow(BigInteger base, int exp) {
    BigInteger result = createBigInteger("1");

    bool resultNegative = false;
    if (base.isNegative == true && (exp % 2 == 1)) {
        resultNegative = true;
    }
    base.isNegative = false;

    while (exp > 0) {
        if (exp % 2 == 1) {
            result = multiply(result, base);
        }
        base = multiply(base, base);
        exp /= 2;
    }

    result.isNegative = resultNegative;
    normalize(&result);
    return result;
}
/* ======================================== */

/* ===== 출력 함수 섹션 ===== */
// BigInteger 출력 함수
void printBigInteger(BigInteger a) {
    if (a.isNegative == true && (a.length > 1 || a.digits[0] != 0)) {
        printf("-");
    }
    for (int i = a.length - 1; i >= 0; --i) {
        printf("%d", a.digits[i]);
    }
    return;
}
/* ======================================== */

int main() {
    char n[MAX_DIGITS + MORE_SPACE_FOR_INPUT + 1], m[MAX_DIGITS + MORE_SPACE_FOR_INPUT + 1];
    scanf("%s %s", n, m);

    /*
    printBigInteger(divide(createBigInteger(n), createBigInteger(m)));
    printf("\n");
    printBigInteger(modular(createBigInteger(n), createBigInteger(m)));
    */
    return 0;
}
```

