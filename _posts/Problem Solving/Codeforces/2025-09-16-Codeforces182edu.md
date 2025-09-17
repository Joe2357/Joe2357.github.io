---
title: "Educational Codeforces Round 182 (Rated for Div. 2) 후기"
author: Joe2357
categories: [Problem Solving, Codeforces]
tags: [Contest, Codeforces]
description: "- Educational Codeforces Round 182 (Rated for Div. 2) 후기"
math: true
---





### A. [Cut the Array](https://codeforces.com/contest/2144/problem/A)

<details markdown="1"><summary>풀이 보기</summary>
#### 풀이  
A번 문제답에 브루트포스로 풀리는 문제였다. 가능한 모든 $l$, $r$에 대해서 각 구간들의 원소의 합을 계산하고 문제 조건에 맞도록 **모두 다르거나 모두 같은 갯수**를 세면 되는 문제였다.

혹시 몰라 누적합 방법을 사용하여 각 구간의 원소의 합을 계산하는 과정을 $O(1)$에 계산하는 것까지 추가했다. 아마 안해도 큰 문제는 없었을 것 같긴 하다.

#### 코드

```c
#include <stdio.h>

#define MAX_IDX (40 + 1)

int arr[MAX_IDX];
int sum[MAX_IDX];
int n;

int main() {
    int t;
    scanf("%d", &t);
    while (t--) {
        scanf("%d", &n);
        int temp = 0;
        for (int i = 1; i <= n; ++i) {
            scanf("%d", arr + i);
            temp += arr[i];
            sum[i] = temp;
        }

        int l = 0, r = 0;
        for (int i = 1; i < n; ++i) {
            int a = sum[i];
            for (int j = i + 1; j < n; ++j) {
                int c = sum[n] - sum[j];
                int b = sum[n] - a - c;

                if (a % 3 != b % 3 && b % 3 != c % 3 && a % 3 != c % 3) {
                    l = i, r = j;
                    break;
                }
                if (a % 3 == b % 3 && b % 3 == c % 3) {
                    l = i, r = j;
                    break;
                }
            }

            if (l != 0 || r != 0) {
                break;
            }
        }

        printf("%d %d\n", l, r);
    }
    return 0;
}
```

</details>

### B. [Maximum Cost Permutation](https://codeforces.com/contest/2144/problem/B)

<details markdown="1"><summary>풀이 보기</summary>
#### 풀이  
문제를 풀 아이디어는 매우 빠르게 찾았다고 생각하는데, 구현에서 이슈가 많았다. 최종적으로는 2번 틀린 문제다.

문제에서 구간 하나를 선택 후 정렬하여 최종적으로 정렬되도록 해야한다고 했으며, <u>최소 길이의 구간</u>을 선택하여 정렬하였을 때 정렬될 수 있다면, 그 때의 구간 길이를 **순열의 cost**라고 정의했다. 즉, "정렬이 되어있지 않은 구간의 길이를 최대로 하는 것"이 우리의 목표가 될 것이다.

배열에서 값이 정해지지 않은 위치, 즉 $0$이 2개 이상 존재한다면, <u>반드시 그 위치에 맞지 않은 값을 넣을 수 있다</u>는 점을 통해, 그 위치의 값은 정렬이 필요한 위치로 생각할 수 있다. 반대로 $0$이 1개만 존재한다면 그 위치의 값은 이미 결정되어있으므로 그 값을 넣도록 하자. 배열에 존재하는 수들은 $1$부터 $N$까지이므로, 배열에서 등장한 수의 전체 합 $sum$을 계산해두었다면 $\frac{N(N+1)}{2} - sum$, 즉 한 번도 등장하지 않은 수를 배열에 넣을 수 있다.
이후에는 정렬이 되지 않은 두 자리를 찾도록 하자. 구간만 찾으면 그 안의 값들은 상관없으므로 `left`와 `right`를 찾기만 해도 될 것 같다. 원래의 위치에 존재하지 않는 수가 나올 때까지 $O(N)$으로 찾아도 시간에는 문제 없다.

#### 코드

```c
#include <stdio.h>

typedef char bool;
const bool true = 1;
const bool false = 0;

#define MAX_IDX (int)(2e5)

long long arr[MAX_IDX + 1];
long long sum = 0;
int n;
int zero_cnt;
int last_zero_idx;
int first, last;

void init() {
    zero_cnt = 0;
    last_zero_idx = 0;
    sum = 0;
    first = 0, last = 0;
    return;
}

int main() {
    int t;
    scanf("%d", &t);
    while (t--) {
        init();

        scanf("%d", &n);
        for (int i = 1; i <= n; ++i) {
            scanf("%lld", arr + i);
            sum += arr[i];
            if (arr[i] == 0) {
                ++zero_cnt;
                last_zero_idx = i;
            }
        }

        if (zero_cnt == 1) {
            long long temp = n;
            temp *= (n + 1);
            temp /= 2;
            arr[last_zero_idx] = temp - sum;
        }

        for (int i = 1; i <= n; ++i) {
            if (arr[i] == 0) {
                first = i;
                break;
            } else if (arr[i] != i) {
                first = i;
                break;
            }
        }

        for (int i = n; i >= 1; --i) {
            if (arr[i] == 0) {
                last = i;
                break;
            } else if (arr[i] != i) {
                last = i;
                break;
            }
        }

        // printf("first : %d, last : %d\n", first, last);
        printf("%d\n", (first == last) ? 0 : (last - first + 1));
    }
    return 0;
}
```

</details>

### C. [Non-Descending Arrays](https://codeforces.com/contest/2144/problem/C)

<details markdown="1"><summary>풀이 보기</summary>
#### 풀이  
이 시간에 경우의 수를 세며, $MOD$까지 존재하는 것으로 보아 이 문제는 거의 무조건 DP를 이용해서 풀어내야한다는 것을 예상할 수 있다. 역시 DP를 활용해서 문제를 해결하는 것이 정해인 것 같다.

`dp[i][j]`를 $j$번 index에 대해, swap을 했을 때와 하지 않았을 때의 subproblem의 정답을 기록해두도록 하자. $i=0$이면 `NO SWAP`, $i=1$이면 `SWAP`을 나타낸다. 그 이후는 문제의 조건에 맞게 구현하면 된다.

#### 코드

```c
#include <stdio.h>

typedef char bool;
const bool true = 1;
const bool false = 0;

#define MAX_IDX 100
const int MOD = 998244353;

int dp[2][MAX_IDX];
const int NO_SWAP = 0;
const int SWAP = 1;

int a[MAX_IDX];
int b[MAX_IDX];
int n;

void init() {
    for (int i = 0; i < 2; ++i) {
        for (int j = 0; j < MAX_IDX; ++j) {
            dp[i][j] = 0;
        }
    }
    dp[NO_SWAP][0] = 1;
    dp[SWAP][0] = 1;
    return;
}

int main() {
    int t;
    scanf("%d", &t);
    while (t--) {
        init();

        scanf("%d", &n);
        for (int i = 0; i < n; ++i) {
            scanf("%d", a + i);
        }
        for (int i = 0; i < n; ++i) {
            scanf("%d", b + i);
        }

        for (int i = 1; i < n; ++i) {
            if (a[i - 1] <= a[i] && b[i - 1] <= b[i]) {
                dp[NO_SWAP][i] += dp[NO_SWAP][i - 1];
                dp[NO_SWAP][i] %= MOD;
            }

            if (b[i - 1] <= a[i] && a[i - 1] <= b[i]) {
                dp[NO_SWAP][i] += dp[SWAP][i - 1];
                dp[NO_SWAP][i] %= MOD;
            }

            if (a[i - 1] <= b[i] && b[i - 1] <= a[i]) {
                dp[SWAP][i] += dp[NO_SWAP][i - 1];
                dp[SWAP][i] %= MOD;
            }

            if (b[i - 1] <= b[i] && a[i - 1] <= a[i]) {
                dp[SWAP][i] += dp[SWAP][i - 1];
                dp[SWAP][i] %= MOD;
            }
        }

        printf("%d\n", (dp[NO_SWAP][n - 1] + dp[SWAP][n - 1]) % MOD);
    }
    return 0;
}
```

</details>

### D. [Price Tags](https://codeforces.com/contest/2144/problem/D)

<details markdown="1"><summary>풀이 보기</summary>
#### 풀이  
의외로 **브루트포스**로 풀리는 문제다! $O(\frac{n}{1} + \frac{n}{2} + \cdots + \frac{n}{n}) = O(n \log n)$이라는 점을 알고 있었으면, 대충 될 수도 있다는 믿음이 필요했다. A번 문제도 완전한 브루트포스였으니, 이 문제도 일단 시도해보자는게 중론.

우선 처음 상태의 가격표의 개수를 기록하는 `cnt_cost[]` 배열을 만들자. 가격표의 개수이기도 하지만, 이후 가격들의 그룹을 만들 때 활용하기도 하는 유용한 배열이다. 다음으로는 `prefixSum_cost[]` 배열이다. 배열의 이름에서도 알 수 있겠지만, `cnt_cost[]`의 누적합 배열이다. 이 배열을 통해 $(lower, upper]$ 범위를 $v$ 하나로 묶어낼 수 있게 계산할 수 있다. 자세한 것은 코드 참고. 

$x$로 가격을 나눈 뒤의 가격 $v$가 같은 원소끼리 묶는 과정을 빠르게 할 수 있으면, 나머지는 모두 브루트포스로 구현하여 답을 도출할 수 있다. 애초에 문제에서 원하는 답을 계산하는 과정을 그대로 만들면 틀릴 일이 없다.

#### 코드

```c
#include <stdio.h>

typedef char bool;
const bool true = 1;
const bool false = 0;

typedef long long ll;
typedef unsigned long long ull;

#define MAX_IDX (int)(2e5)
const ll INF = 1LL << 60;

int cnt_cost[MAX_IDX + 1];
int prefixSum_cost[MAX_IDX + 1];
int n;
ll y;
int max_c;

#define min(a, b) (((a) > (b)) ? (b) : (a))
#define max(a, b) (((a) > (b)) ? (a) : (b))

void init() {
    max_c = -1;
    for (int i = 0; i <= MAX_IDX; ++i) {
        cnt_cost[i] = 0;
        prefixSum_cost[i] = 0;
    }
    return;
}

void read_input() {
    scanf("%d %lld", &n, &y);
    for (int i = 0; i < n; ++i) {
        int a;
        scanf("%d", &a);
        max_c = max(max_c, a);
        cnt_cost[a] += 1;
    }
    return;
}

void solve() {
    /* error case 1 :: 모든 cost들이 초기에 1 */
    if (max_c == 1) {
        printf("%d\n", n);
        return;
    }

    for (int i = 1; i <= max_c; ++i) {
        prefixSum_cost[i] = prefixSum_cost[i - 1] + cnt_cost[i];
    }
    ll total_cost = -INF;

    // brute force
    for (int x = 2; x <= max_c; ++x) {
        int ceil_max_cost = (max_c - 1) / x + 1;
        ll sum_prices = 0;
        ll reused_pricetag = 0;

        for (int v = 1; v <= ceil_max_cost; ++v) {
            // (lower, upper] 범위의 모든 값들은 x로 나누었을 때 v로 묶인다
            int lower_bound = (v - 1) * x;
            int upper_bound = min(max_c, v * x);

            int cnt_newPrice = prefixSum_cost[upper_bound] - prefixSum_cost[lower_bound];
            if (cnt_newPrice > 0) {
                sum_prices += (ll)v * cnt_newPrice;

                if (v <= max_c) {
                    reused_pricetag += (cnt_cost[v] < cnt_newPrice ? cnt_cost[v] : cnt_newPrice);
                }
            }
        }

        // 현재 x에서의 결과 비교 및 정리
        ll printed = (ll)n - reused_pricetag;
        ll income = sum_prices - y * printed;
        total_cost = max(total_cost, income);
    }
    printf("%lld\n", total_cost);
    return;
}

int main() {
    int t;
    scanf("%d", &t);
    while (t--) {
        init();
        read_input();
        solve();
    }
    return 0;
}
```

</details>

### 결과

|                          Contest                          |   Start time    | Rank | Solved | Rating change |                New rating                |
| :-------------------------------------------------------: | :-------------: | :--: | :----: | :-----------: | :--------------------------------------: |
| Educational Codeforces Round #182<br/> (Rated for Div. 2) | 2021/9/15 23:35 | 1562 |   4    |      +67      | <strong style="color:cyan">1566</strong> |
