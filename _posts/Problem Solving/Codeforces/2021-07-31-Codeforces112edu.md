---
title: "Educational Codeforces Round 112 (Rated for Div. 2) 후기"
author: Joe2357
categories: [Problem Solving, Codeforces]
tags: [Contest, Codeforces]
description: "- Educational Codeforces Round 112 (Rated for Div. 2) 후기"
math: true
---





### A. [PizzaForces](https://codeforces.com/contest/1555/problem/A)

<details markdown="1"><summary>풀이 보기</summary>
#### 풀이

생각보다 많이 꼬여있고, 많이 단순한 문제라고 생각한다.

만들 수 있는 피자의 종류는 6조각, 8조각, 10조각으로 총 3가지이다. 여러가지 조합하면 알 수 있는 점은 위 3종류의 피자로 $n ≥ 6$인 모든 짝수 피자 조각을 만들어낼 수 있다. 이외의 짝수 $n = 2, 4$ 피자조각은 무조건 6조각짜리 피자를 만들어야 생산해낼 수 있다.

홀수 개의 피자조각같은 경우 딱 맞춰 만들어낼 수 없으므로, 1조각을 추가해 짝수개의 문제로 바꾸고 풀어내는 것이 가능하다.

만들어야하는 피자조각의 개수를 정했으니, 시간이 얼마나 걸릴지 찾아야하는데, 피자 조각 개수가 달라지더라도 **각 조각당 필요한 시간은 변함이 없다**는 점을 문제에서 찾아볼 수 있다. 따라서 문제를 해결하는 순서는 아래와 같다.

- 조각의 개수가 홀수라면 1을 더해 짝수로 바꾼다
- 조각의 개수가 6보다 작다면 6조각짜리 피자를 하나 만들어야하므로, 필요한 시간은 15분이다
- 이외의 짝수 조각의 피자를 만들기 위해서는 각 조각당 필요한 시간을 곱해서 구할 수 있다

#### 코드

```c
#include <stdio.h>

typedef long long ll;

int t;

int main() {
    scanf("%d", &t);
    while (t--) {
        ll a, result = 0;
        scanf("%lld", &a);
        if (a % 2 == 1) {
            ++a;
        }

        if (a < 6) {
            printf("15\n");
        } else {
            printf("%lld\n", (a * 5) >> 1);
        }
    }
    return 0;
}
```

</details>

### B. [Two Tables](https://codeforces.com/contest/1555/problem/B)

<details markdown="1"><summary>풀이 보기</summary>
#### 풀이

별로 마음에 안드는 case work 문제이다. 따라서 문제 해설보다는 어떤 순서로 문제를 풀어나갔는가를 적으려한다.

- 모든 변수를 입력받는다
- 첫 번째 table을 설치한 이후 남는 여백을 계산한다
  - tx1 : table 왼쪽에 남는 공간
  - tx2 : table 오른쪽에 남는 공간
  - ty1 : table 아래에 남는 공간
  - ty2 : table 위에 남는 공간
- 추가로 table을 놓을 수 있는지 검사한다
  - 만약 추가로 놓을 공간을 다 합쳐도 부족하다면 $-1$을 출력한다
- 만약 x축, y축 중 한가지만 부족하다면, 가능한 축을 이용하여 table을 옮기면 추가로 table을 넣을 수 있는 공간을 마련할 수 있다. 그 거리를 계산한다
  - 물론 table을 옮기지 않아도 가능한 경우가 있으므로 예외처리하여 $0$을 출력하도록 하자
- 만약 x축, y축이 모두 가능하다면 각각 따로 계산하여 더 작은 값을 출력한다

#### 코드

```c
#include <math.h>
#include <stdio.h>

int t;

#define min(a, b) (((a) > (b)) ? (b) : (a))
#define abs(x) ((x < 0) ? (-x) : (x))
int main() {
    scanf("%d", &t);
    while (t--) {
        int W, H;
        scanf("%d %d", &W, &H);
        int x1, y1, x2, y2;
        scanf("%d %d %d %d", &x1, &y1, &x2, &y2);
        int w, h;
        scanf("%d %d", &w, &h);

        int tx1 = x1, tx2 = W - x2;
        int ty1 = y1, ty2 = H - y2;
        double a, b;

        if (tx1 + tx2 < w) {
            if (ty1 + ty2 < h) {
                printf("-1\n");
            } else {
                if (ty1 >= h || ty2 >= h) {
                    printf("0\n");
                } else {
                    printf("%d\n", min(h - ty1, h - ty2));
                }
            }
        } else {
            if (ty1 + ty2 < h) {
                if (tx1 >= w || tx2 >= w) {
                    printf("0\n");
                } else {
                    printf("%d\n", min(w - tx1, w - tx2));
                }
            } else {
                if (tx1 >= w || tx2 >= w) {
                    printf("0\n");
                } else if (ty1 >= h || ty2 >= h) {
                    printf("0\n");
                } else {
                    printf("%d\n", min(min(w - tx1, w - tx2), min(h - ty1, h - ty2)));
                }
            }
        }
    }
    return 0;
}
```

</details>

### C. [Coin Rows](https://codeforces.com/contest/1555/problem/C)

<details markdown="1"><summary>풀이 보기</summary>
#### 풀이

처음에는 게임이론 문제라고 생각했다. 아예 아닌 것은 아닌데, 게임이론 방법까지 생각하지는 않아도 되는 문제였다.

문제에서 원하는 score는 2번 플레이어가 획득한 점수임을 유의하자. 1번 플레이어가 얻는 점수는 아무 의미 없다.

각 플레이어는 오른쪽과 아래로만 이동할 수 있으므로, 이동할 수 있는 경우의 수가 정해져있다. 예시를 들면 아래 모양과 같을 것이다.

■■■■■■□□□□□□  
□□□□□■■■■■■■

모든 칸은 양수라는 전제가 있으므로, 2번 플레이어가 점수를 최대로 얻기 위해서는 2가지 중 선택할 수 있다. 

- 처음부터 아래로 내려가는 경로
- 가장 오른쪽까지 이동한 후 내려가는 경로

score를 최대로 가지기 위한 2번 플레이어의 행동으로, 위의 2가지 경우 중 더 큰 점수를 얻을 수 있는 방법을 선택할 것이다.

1번 플레이어는 이 score를 최소로 할 수 있는 지점에서 아래로 내려가는 경로를 선택하는 것이 최선의 방법일 것이다. 배열에서 **연속적인 합**을 자주 이용하는 것을 알아내었고, dp의 부분합 방식을 채용하여 구현했다.

#### 코드

```c
#include <stdio.h>

#define M (int)1e5
#define ROW 2

int t;
int m;
int arr[ROW][M + 1];
int dp[ROW][M + 1];

#define max(a, b) (((a) > (b)) ? (a) : (b))
#define min(a, b) (((a) > (b)) ? (b) : (a))
int main() {
    scanf("%d", &t);
    while (t--) {
        scanf("%d", &m);
        for (int i = 0; i < ROW; ++i) {
            for (int j = 1; j <= m; ++j) {
                scanf("%d", &arr[i][j]);
                dp[i][j] = dp[i][j - 1] + arr[i][j];
            }
        }

        int result = (int)(2e9 + 1);
        for (int i = 1; i <= m; ++i) {
            result = min(result, max(dp[1][i - 1] - dp[1][0], dp[0][m] - dp[0][i]));
        }
        printf("%d\n", result);
    }
    return 0;
}
```

</details>

### D. [Say No to Palindromes](https://codeforces.com/contest/1555/problem/D)

<details markdown="1"><summary>풀이 보기</summary>
#### 풀이

쿼리 문제이다. 최대 $2 \cdot 10^5 \times 2 \cdot 10^5$번 반복하므로 둘 중 하나를 $O(1)$에 풀어내야 한다는 점을 기억해야한다. TLE를 한번 받은 문제이다.

문자는 a, b, c 총 3개만을 사용한다고 되어있다. 이 3가지 문자를 이용하여 <u>모든 부분 문자열이 팰린드롬이 아니어야한다</u>는 조건을 만족하기 위해서는 문자열의 모습이 어느정도 정해진다. 말로 풀어쓰면 "3가지 문자를 모두 사용한 3글자짜리 문자열이 반복되는 형태" 정도일 것이다.

결국 입력된 문자열에 대해 모든 부분에 대해 미리 최적의 값을 계산해놓는 방법을 사용할 수 있다. 계산한 이후 쿼리가 들어올 때마다 구간을 계산하여 답을 찾아내었다.

#### 코드

```c
#include <stdio.h>

#define M (int)(2e5 + 1)
int n, m;
int arr[6];
char str[M];
char tp[6][4] = {
    "abc", "bca", "cab",
    "acb", "bac", "cba"};
int dp[6][M + 1];

int main() {
    scanf("%d %d", &n, &m);
    scanf("%s", str);

    for (int i = 0; i < 6; ++i) {
        for (int j = 1; j <= n; ++j) {
            dp[i][j] = dp[i][j - 1] + (str[j - 1] != tp[i][j % 3]);
        }
    }

    while (m--) {
        int a, b;
        scanf("%d %d", &a, &b);

        int x = dp[0][b] - dp[0][a - 1];
        for (int i = 1; i < 6; ++i) {
            //printf("arr[%d] = %d\n", i, arr[i]);
            if (x > dp[i][b] - dp[i][a - 1]) {
                x = dp[i][b] - dp[i][a - 1];
            }
        }
        printf("%d\n", x);
    }
    return 0;
}
```

</details>

### 결과

|                          Contest                          |   Start time    | Rank | Solved | Rating change |                New rating                |
| :-------------------------------------------------------: | :-------------: | :--: | :----: | :-----------: | :--------------------------------------: |
| Educational Codeforces Round #112<br/> (Rated for Div. 2) | 2021/7/30 23:35 | 2116 |   4    |      -3       | <strong style="color:blue">1625</strong> |

SCPC 2차예선이 일주일 남은 시점에서, contest의 감을 좀 다지고자 참여하였다.

rate 욕심도 있었고, 최대한 많이 풀어내자고 다짐하고 참여했다.B번 문제에서 시간을 너무 많이 잡아먹어버렸다. C번과 D번 문제를 30분 이내로 풀어내서 겨우 rate 방어는 성공했다.

case work에서 아직 많이 부족하다는 것을 알았다. SCPC 1라운드에서도 case work 문제에서 많은 시간을 소요했는데, 이 부분에 대한 생각 전환을 더 빠르게 할 방법을 찾아야겠다.

오히려 쉬운 문제에서 실수하는 경향이 많아졌다. 실수를 최대한 줄이는 것이 rate를 올리는 방법이 될 것이다.