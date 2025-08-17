---
title: "Bipartite Matching"
author: Joe2357
categories: [Storage, Algorithm]
tags: [Algorithm]
description: "- 이분 매칭 알고리즘"
math: true
---



## 이분 매칭 알고리즘

이분 매칭을 알기 전에 '이분 그래프'라는 것을 먼저 알아야한다. 그래프에서, **정점 그룹을 2개로 완벽하게 분할할 수 있다면** 이분 그래프(Bipartite Graph) 라고 한다. 정점 그룹을 완벽하게 분할한다는 것은 "같은 그룹의 정점으로 연결되어 있는 간선이 없도록" 정점을 그룹짓는 것을 말한다. 그래프에 존재하는 모든 간선에 대해, <u>간선들의 각 정점들은 서로 다른 그룹 상에 존재</u>한다는 것이 특징이다.

이분 매칭이란, 이런 이분 그래프 상에서 한 그룹을 $X$, 다른 그룹을 $Y$라고 하였을 때, $X \rightarrow Y$인 경로에서의 최대 유량을 계산하는 문제이다. 특히 모든 간선의 용량을 $1$로 놓고, 하나의 노드가 다른 하나의 노드를 **점유**한다는 표현을 사용한다. 노드를 점유한다는 것은 현재 노드를 다른 노드가 선택하고, 다른 노드들이 현재 노드와 연결되지 못하도록 하는 것을 말한다. 쉽게 생각하면 다른 그룹에 있는 노드와 **중복 없이 하나의 쌍을 만드는 것**이라고 할 수 있다. 이분 매칭 문제는 이러한 쌍을 최대 몇개 만들 수 있는가를 찾는 문제이다.

  

### 구현 방법

네트워크 플로우 문제인 만큼, 네트워크 플로우에서 사용할 수 있는 알고리즘을 사용할 수 있다고 한다. 그리고 이분 그래프의 특성에 한해 다른 알고리즘으로 시간을 크게 줄일 수도 있다고 한다. 이 아래부터는 현재 작성자가 배우고 사용하고 있는 방법에 대해서만 설명한다.



#### DFS를 이용하는 방법

그룹 $X$에 있는 모든 정점에 대하여 [dfs 탐색](https://en.wikipedia.org/wiki/Depth-first_search)을 이용하여 구현한다. 그룹 $X$의 노드는 그룹 $Y$의 노드 중 하나를 점유하려고 시도한다. 이 때 아래와 같은 선택지에 마주한다. 아래에서는 그룹 $X$의 노드를 $x$, 그룹 $Y$의 노드 중 현재 점유하려고 시도 중인 노드를 $y$라고 하겠다.

- $x$는 모든 $y$에 대하여 $x$가 $y$를 점유할 수 있는지 판단
  - $x$가 점유 시도하려는 $y_i$가 현재 점유 상태가 아닌 경우
    - => $x$가 $y_i$를 점유할 수 있다
  - $y_i$가 $X$의 다른 노드 $a$에게 이미 점유당한 상태인 경우
    - => $a$가 $Y$의 다른 노드를 점유할 수 있는지 판단. 다른 노드를 점유할 수 있다면 $a$는 그 노드를 점유하도록 하고, $x$는 $y_i$를 점유한다
    -  => $Y$의 모든 노드에 대해 $x$가 점유할 수 없다고 판단되면 매칭 실패로 간주

$x$가 노드를 점유할 때마다 `retval`에 $1$을 더한다. 모든 알고리즘이 종료된 이후 `retval` 값은 그래프 상에서 최종적으로 매칭된 개수를 의미한다.

```c
#include <stdio.h>

typedef char bool;
const bool true = 1;
const bool false = 0;

/* 노드의 번호 : 1 ~ n */
#define MAX_IDX (100 + 1)
#define NOTMATCHED 0

bool matrix[MAX_IDX][MAX_IDX];    // 간선 정보를 가지는 인접행렬
bool visit[MAX_IDX];
int matched[MAX_IDX];
int n;

bool dfs(int x) {
    for (int i = 1; i <= n; ++i) {    // 모든 정점에 대해 들어갈 수 있는지 판단
        if (matrix[x][i] == true) {
            if (visit[i] == true) {
                continue;  // 이미 처리한 정점은 고려하지 않음
            }
            visit[i] = true;

            /*
            정점 선택 가능 조건 : i가 현재 매칭되어있지 않거나,
            i를 점유한 정점이 다른 정점과 매칭될 수 있다면 대체할 수 있음
            */
            if (matched[i] == NOTMATCHED || dfs(matched[i]) == true) {
                matched[i] = x;     // a그룹의 x가 b그룹의 i를 선택
                return true;
            }
        }
    }
    return false;
}

int main() {
    /* 간선 정보 입력받기 */
    // 조건 : 정점 그룹이 2개로 완벽하게 분할될 수 있어야한다.
    /* ----------------- */

    int retval = 0;     // 최대 매칭 개수

    // 모든 노드를 매칭하려고 시도
    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= n; ++j) {    // visit 체크 해제
            visit[j] = false;
        }

        if (dfs(i) == true) {  // 현재 노드를 매칭할 수 있는지 탐색
            ++retval;
        }
    }

    printf("최대 매칭 개수 : %d\n", retval);
    for (int i = 1; i <= n; ++i) {    // 어떤 노드에 매칭되었는지 출력
        if (matched[i] != NOTMATCHED) {
            printf(" ㄴ %d -> %d\n", matched[i], i);
        }
    }
    return 0;
}
```

- 시간 복잡도 : $O(VE)$
