---
title: "Segment Tree"
author: Joe2357
categories: [Storage, Data Structure]
tags: [Data Structure]
uploaded_at: 2023-03-23 12:00:00 +0900
last_modified_at: 2023-03-23 12:00:00 +0900
description: "- 세그먼트 트리 / 느리게 갱신되는 세그먼트 트리"
math: true
---



## 세그먼트 트리

  - 배열에서, 구간에 대한 쿼리를 실현할 때 사용될 수 있는 자료구조
      - 특히, **구간에 대해 값을 변경하는 쿼리**가 있을 경우 복잡도에서 효과적



### 예시 상황

배열 $A$가 있고, 여기서 아래 2가지 연산을 수행할 수 있는 문제를 생각해보자.

1. 구간 $l$, $r$ ($l \leq r$) 이 주어졌을 때, $\sum_{i=l}^{r} A[i]$를 구하여 출력하기
2. $i$번째 수를 $v$로 바꾸기

배열 $A$의 길이가 $N$, 수행하는 연산이 $M$번일 때, 1번 연산의 복잡도는 $O(N)$, 2번 연산의 복잡도는 $O(1)$이므로 총 시간복잡도는 $O(NM)$이다.

만약 2번 연산이 없었다면, 1번 연산은 누적합 방법으로 시간복잡도를 해결할 수 있다. 구간합 배열 $S$를 만든다면, 아래와 같이 만들 수 있다.

```c
S[0] = A[0];
for(int i = 1; i <= n; ++i) {
    S[i] = S[i - 1] + A[i];
}
```

이렇게 만든다면 1번 연산을 수행하기 위해서는 $S[r] - S[l - 1]$로 계산할 수 있다. 이렇게 된다면 쿼리를 $O(1)$로 처리할 수 있으므로 시간복잡도는 $O(N) + O(M)$이다. 빨라보이지만, 2번 연산이 고려되지 않았다는 문제점이 있다. 2번 연산을 넣는 경우 갱신 한번에 $O(N)$이므로 결국 $O(NM)$의 시간복잡도를 가진다.



### 활용 방법

세그먼트 트리를 이용하면 위의 예시 상황에서의 문제를 빠르게 해결할 수 있다. 세그먼트 트리는 일반적인 이진트리 모양을 하고 있으며, 각 노드는 '자식 노드의 값의 **구간** 및 **합**'을 나타낸다. 자식 노드는 부모 노드의 구간을 반씩 나눠가져서 더 작은 구간의 합을 기록한다. 이렇게 하면 리프노드는 구간 길이가 1이므로 배열에서의 수 자체를 가지게 되고, 다른 노드들은 왼쪽 노드와 오른쪽 노드가 해당하는 구간 범위의 합을 가지게 된다. 분할정복의 결과값 저장 버전이라고도 이해할 수 있을 것 같다.

말만 트리지 사실상 배열을 이용하여 구현할 수 있는데, [우선순위 큐](https://joe2357.github.io/posts/Heap/)을 만들던 방식과 똑같이, 루트 노드를 $1$번, 자식노드는 $2x$, $2x + 1$로 표현할 수 있다.

위에서도 얘기했다시피 세그먼트 트리를 사용하는 가장 큰 이유는 시간복잡도에 있다. 세그먼트 트리를 활용하면 1번 연산을 $O(\log N)$, 2번 연산을 $O(\log N)$으로 처리할 수 있다. 1번 연산은 **구간에 포함되는 노드의 합**으로 표현 가능하고, 2번 연산은 **i를 포함하는 노드의 값을 전부 바꾸는 것**으로 구현 가능하다. 이진트리의 특성상 노드가 $N$개 있다면 그 높이는 $\log N$이 되기 때문에 가능한 계산. 이렇게 하면 전체 계산을 $O(M \log N)$으로 처리할 수 있게 된다!



### 구현 방법

분할정복 방법대로 재귀함수로 구현하는 것이 일반적이다. 다만 그때의 상황의 결과를 트리에 기록한다는 느낌으로 구현한다.

먼저 처음 배열값을 트리에 저장하는 `init` 함수이다.

```c
typedef long long ll;

ll a[MAX_IDX];
ll tree[MAX_IDX * 4];

/*
    a: 배열 a
    tree: 세그먼트 트리
    node: 현재 보고있는 노드 번호
    node가 담당하는 합의 범위가 start ~ end
*/
ll init(int node, int start, int end) {
    if (start == end) {  // 리프노드인 경우
        return tree[node] = a[start];
    } else {  // 왼쪽 자식과 오른쪽 자식을 init시키고 그 결과의 합을 기록
        ll leftInit = init(node * 2, start, (start + end) / 2);
        ll rightInit = init(node * 2 + 1, (start + end) / 2 + 1, end);
        return tree[node] = leftInit + rightInit;
    }
}
```



다음으로는 특정 결과를 반환하는 `query`함수이다. 이 문제에서는 예시 상황에 맞게 특정 구간의 합을 출력하는 함수라고 생각할 수 있다. 우리가 계산해야하는 수의 범위가 $[l, r]$이라면 각 노드가 담당하는 구간 $[s, e]$에 따라 여러가지 상황에 마주할 수 있다.

1. $[l, r]$이 $[s, e]$와 구간이 겹치지 않는 경우
2. $[l, r]$가 $[s, e]$를 완전히 포함하는 경우
3. $[s, e]$가 $[l, r]$을 완전히 포함하는 경우
4. $[l, r]$와 $[s, e]$ 구간이 겹치는 부분이 있는 경우

1번같은 경우는 `l < e || r > s`에 해당하고, 우리가 구하려는 값이 $[s, e]$ 구간에는 없으므로 해당사항이 없음, 0을 반환한다. 그리고 이 노드 아래로는 탐색을 이어갈 필요가 없다. 자식 노드들의 구간이 어짜피 $[s, e]$에 포함되는 구간이고, 그 구간들은 $[l, r]$과는 겹치지 않을 것이기 때문이다.

2번같은 경우는 `l <= s && r >= e`에 해당한다. 우리가 구하려는 값이 $[s, e]$를 모두 포함하고 있기 때문에 그 노드가 담당하고 있는 모든 값을 반환해주면 된다. 즉 `tree[node]`값을 반환하게 된다. 이것도 1번과 마찬가지로 자식 노드들을 탐색할 이유는 없다.

3번과 4번은 현재 탐색 중인 노드 자체를 사용할 수는 없다. $[s, e]$ 구간에는 필요한 값이 있기도 하지만 필요 없는 값도 존재하기 때문이다. 이때는 문제를 반으로 나누어 탐색을 이어나가는 방식을 사용한다. 즉 자식노드의 탐색을 추가적으로 진행하게 된다.

이것을 코드로 풀어내면 아래와 같아진다.

```c
// node가 담당하는 구간이 start ~ end이고, 구해야하는 합의 범위는 left ~ right
ll query(int node, int start, int end, int left, int right) {
    if (left > end || right < start) {
        return 0;
    }
    if (left <= start && end <= right) {
        return tree[node];
    }
    
    ll leftRet = query(node * 2, start, (start + end) / 2, left, right);
    ll rightRet = query(node * 2 + 1, (start + end) / 2 + 1, end, left, right);
    return leftRet + rightRet;
}
```



마지막으로 사실상 세그먼트 트리를 사용하는 이유, '값을 변경하는 연산'을 해결하기 위한 함수 `update`이다. 이 부분은 위의 쿼리보다는 쉽게 구현이 가능하다. 값을 변경하려는 index가 현재 탐색 중인 범위에 존재하지 않는다면 변화 없이 리턴하면 되고, 만약 해당하는 구간 안에 포함이 되어있다면 각 노드는 **배열의 값이 변경되었다고 가정**하고 그 때의 결과값을 새로 기록하도록 한다. 이 때 `diff`라는 값을 주로 사용하는데, 배열의 값이 바뀌면서 그 차이를 이용하는 것이다. 이후 자식 노드들도 값을 변경하도록 재귀적으로 함수를 호출하게 된다. 이 호출은 리프 노드가 아닐 때까지 반복한다.

```c
void update(int node, int start, int end, int index, ll diff) {
    if (index < start || index > end) return;
    tree[node] = tree[node] + diff;
    if (start < end) {
        update(node * 2, start, (start + end) / 2, index, diff);
        update(node * 2 + 1, (start + end) / 2 + 1, end, index, diff);
    }
}
```


위의 과정을 통해 세그먼트 트리를 구현할 수 있다. 사용처가 매우 다양하기 때문에 노드의 업데이트나 쿼리 동작들이 매우 다양하게 수정할 수 있는 요소가 있다. 하지만 구조 자체는 비슷하게 흘러갈 수 있어서 일반적인 구조 코드 자체는 라이브러리화시킬 수 있을 것 같다.

```c
#include <stdio.h>

typedef long long ll;

#define MAX_IDX 100000
ll a[MAX_IDX];
ll tree[MAX_IDX * 4];

/*
    a: 배열 a
    tree: 세그먼트 트리
    node: 현재 보고있는 노드 번호
    node가 담당하는 합의 범위가 start ~ end
*/
ll init(int node, int start, int end) {
    if (start == end) {     // 리프노드인 경우
        return tree[node] = a[start];
    } else {  // 왼쪽 자식과 오른쪽 자식을 init시키고 그 결과의 합을 기록
        return tree[node] = init(node * 2, start, (start + end) / 2) + init(node * 2 + 1, (start + end) / 2 + 1, end);
    }
}

// node가 담당하는 구간이 start ~ end이고, 구해야하는 합의 범위는 left ~ right
ll query(int node, int start, int end, int left, int right) {
    if (left > end || right < start) {
        return 0;
    }
    if (left <= start && end <= right) {
        return tree[node];
    }

    return query(node * 2, start, (start + end) / 2, left, right) + query(node * 2 + 1, (start + end) / 2 + 1, end, left, right);
}

void update(int node, int start, int end, int index, ll diff) {
    if (index < start || index > end) return;
    
    /* 값을 변경하는 쿼리 */

    if (start < end) {
        update(node * 2, start, (start + end) / 2, index, diff);
        update(node * 2 + 1, (start + end) / 2 + 1, end, index, diff);
    }
}

int main() { return 0; }
```



## 느리게 갱신되는 세그먼트 트리

배열의 값을 바꾸는 연산이 있는 경우 세그먼트 트리가 유용하다는 것은 이전에 얘기했다. 하지만 아래와 같은 연산이 있다면 어떨까?

1. 구간 $l$, $r$ ($l \leq r$) 이 주어졌을 때, $\sum_{i=l}^{r} A[i]$를 구하여 출력하기
2. $i$번째 수를 $v$로 바꾸기
3. 구간 $[a, b]$에 각각 $x$를 더하기

기존의 세그먼트 트리는 2번 연산까지 $O(M \log N)$의 복잡도로 문제를 해결할 수 있었다. 하지만 3번 연산이 추가되면 2번 연산을 총 $b-a+1$번 하는 과정을 거칠 것이고, 이것은 시간복잡도로 $O(N \log N)$이다. 기존의 누적 합 방법 $O(N)$보다 오히려 복잡도가 늦어지는 문제점이 생긴다!

이를 해결하기 위해 탐색 횟수를 줄이는 방법이 필요하게 되었다. 그리고 그 방법은 **노드의 갱신을 늦게 한다**는 아이디어로 해결할 수 있게 되었다. 이것을 우리는 "느리게 갱신되는 세그먼트 트리" (Lazy Propagation) 방법이라고 말한다.

기존의 세그먼트 트리에 우리는 `lazy[]` 배열을 하나 더 생성할 수 있다. lazy 배열은 '그 노드에 추가로 업데이트해야할 값'을 의미한다. 즉, 원래대로라면 세그먼트 트리의 노드에 값을 업데이트해야하지만 구간 안의 모든 노드를 전부 업데이트하는 것은 시간이 오래 걸리니까, <u>그 구간에 업데이트되어야할 정보를 따로 기록해두자</u>는 것이다. 이후 세그먼트 트리의 노드를 탐색할 때 노드의 값을 lazy 값을 이용하여 업데이트하고 lazy를 자식 노드들에게도 전파시킨다.

```c
void update_range(int node, int s, int e, int l, int r) {
    if (l > e || r < s) {
        return;
    } else if (l <= s && e <= r) {
        if (s < e) {
            lazy[node * 2] += lazy[node];
            lazy[node * 2 + 1] += lazy[node];
        }
        return;
    }

    update_range(node * 2, s, (s + e) / 2, l, r);
    update_range(node * 2 + 1, (s + e) / 2 + 1, e, l, r);
    return;
}

int query(int node, int s, int e, int l, int r) {
    
    if (l > e || r < s) {
        return 0;
    } else if (l <= s && e <= r) {
        return tree[node];
    }

    return query(node * 2, s, (s + e) / 2, l, r) + query(node * 2 + 1, (s + e) / 2 + 1, e, l, r);
}
```

`lazy[]`를 이용하면 3번 연산을 빠른 시간 안에 해결할 수 있다. 다만 lazy만 추가한다고 해서 끝나는 것이 아니다. 쿼리를 계산하려면 항상 세그먼트 트리에서 탐색 중인 노드들은 항상 lazy 값이 적용되었을 때의 결과를 기록하고 있어야한다는 것. 그렇다고 노드 업데이트를 다 해버리면 lazy를 만든 이유가 없다. 두 상황을 모두 해결하려면 lazy를 실제 노드에 적용해야할 타이밍과 그 방법을 정해야할 것이다.

lazy를 적용하는 타이밍은 당연하겠지만 그 노드를 탐색하는 경우일 것이다. 기존의 값과 lazy가 모두 적용되었을 때의 결과를 가지고 있어야하니까 노드를 탐색했다면 그 시점에는 무조건 최종 값을 가지고 있어야한다. 그리고 그 결과는, lazy가 다른 노드에도 모두 적용되었을 때의 결과값이어야한다. 문제마다 어떻게 값을 계산할지는 제각각이겠지만, 이 문제에서는 `lazy[]`가 각 구간에 더해야하는 값을 가지고 있을 것이므로, 구간의 길이 $\times$ lazy값 을 추가해주면 될 것이다.



### 소스 코드

최종적으로 느리게 갱신되는 세그먼트 트리는 lazy 관리를 어떻게 할 것인지에 대한 생각이 필요하다.

```c
#include <stdio.h>

#define MAX_IDX 100000
int tree[MAX_IDX * 4];
bool lazy[MAX_IDX * 4];
int n;

void update_lazy(int node, int s, int e) {
    if (lazy[node] == true) {
        tree[node] += (e - s + 1) * lazy[node];
        if (s < e) {
            lazy[node * 2] = lazy[node];
            lazy[node * 2 + 1] = lazy[node];
        }

        lazy[node] = 0;
    }
    return;
}
void update_range(int node, int s, int e, int l, int r) {
    update_lazy(node, s, e);

    if (l > e || r < s) {
        return;
    } else if (l <= s && e <= r) {
        tree[node] += (e - s + 1) * lazy[node];
        if (s < e) {
            lazy[node * 2] = lazy[node];
            lazy[node * 2 + 1] = lazy[node];
        }
        return;
    }

    update_range(node * 2, s, (s + e) / 2, l, r);
    update_range(node * 2 + 1, (s + e) / 2 + 1, e, l, r);
    tree[node] = tree[node * 2] + tree[node * 2 + 1];
    return;
}

int query(int node, int s, int e, int l, int r) {
    update_lazy(node, s, e);
    
    if (l > e || r < s) {
        return 0;
    } else if (l <= s && e <= r) {
        return tree[node];
    }

    return query(node * 2, s, (s + e) / 2, l, r) + query(node * 2 + 1, (s + e) / 2 + 1, e, l, r);
}

int main() {
    return 0;
}
```



