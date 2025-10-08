---
title: "Minimum Spanning Tree"
author: Joe2357
categories: [Storage, Algorithm]
tags: [Algorithm]
date: 2023-06-29 12:00:00 +0900
last_modified_at: 2023-06-29 12:00:00 +0900
description: "- 최소 신장 트리"
math: true
---



## 최소 신장 트리

- 스패닝 트리 (Spanning Tree), 신장 트리 : 그래프에서 <u>모든 정점을 연결한 트리 구조</u>들 중, **사이클을 포함하지 않는** 트리
  - 일단 트리이기 때문에, 정점의 개수가 $n$개라면 간선의 개수는 $n-1$개여야한다. 그래프의 간선들 중 일부만 선택해서 얻을 수 있는 트리구조인 셈
  - 하나의 그래프에서도 여러 스패닝 트리가 존재할 수 있다
- 최소 스패닝 트리 (Minimum Spanning Tree) : 그래프 내에 존재하는 스패닝 트리들 중, **간선들의 가중치 합이 가장 적은** 스패닝 트리
  - 즉, 그래프에서 <u>모든 정점을 연결</u>하면서 <u>사이클이 생기지 않고</u> <u>간선의 가중치 합이 가장 적은</u> 트리



### 구현 방법

크게 2가지 방법을 이용해서 그래프의 MST를 찾는다. 이 글에서는 2가지 방법에 대해 모두 설명하려한다.



#### 크루스칼 알고리즘 (Kruskal Algorithm)

[그리디 알고리즘](https://en.wikipedia.org/wiki/Greedy_algorithm) 방법으로 정점들을 그룹핑해가는 방법. 특히 간선을 중심으로 진행되는 알고리즘이다.

가중치가 적은 간선들을 우선적으로 이용해서 정점들을 이어가다보면 가중치 합이 최소인 트리가 만들어질 것이라는 생각에서 만들어진 방법이다. 그래서, 간선들을 가중치 순서로 정렬해서 사용한다.

일반적인 과정은 아래와 같다.

1. 간선을 가중치 순서로 오름차순 정렬한다
2. 정렬된 모든 간선에 대하여 아래 과정을 반복한다
   - 현재 보고 있는 간선을 MST에 추가할 수 있는지 검사해야한다
     - MST의 조건 중 **사이클을 포함하지 않는다**는 조건을 확인한다
     - 사이클이 만들어질 간선이라면 제외한다
     - 사이클을 만들지 않는 간선이라면 선택하여 MST에 추가한다
3. 선택된 간선들이 MST를 이루는 간선들이다

크루스칼 알고리즘은 모든 간선에 대해 **"이 간선을 추가하면 사이클이 발생하는가?"** 라는 질문에 대한 답을 계속해서 찾아야 한다. 이 과정을 진행하기 위해, 선택된 간선들에 의해 <u>이어질 정점들의 관계</u>를 저장해두면 편하다. 바꿔 말하면 선택된 간선에 의해 이어질 정점들을 **그룹핑**하는 것이다. 이 때 사용할 수 있는 방법은 [분리 집합](https://joe2357.github.io/posts/Disjoint-Set/)이 있다.

예시 구현 코드는 아래와 같다.

```c
/* 크루스칼 알고리즘 */

void disjoint_init() {
    for (int i = 1; i <= v; ++i) {
        parent[i] = i;
    }
    return;
}
int find(int x) {
    if (parent[x] == x) {
        return x;
    } else {
        return parent[x] = find(parent[x]);
    }
}
void merge(int a, int b) {
    int x = find(a), y = find(b);
    parent[b] = parent[y] = x;
    return;
}

int main() {
    /* 정점, 간선 정보 입력 */

    disjoint_init();
    qsort(edge, e, sizeof(ED), cmp); // 간선 오름차순 정렬

    int result = 0;
    int merged_count = 1;
    
    /* 모든 정점을 이을 때까지 반복 */
    for (int i = 0; merged_count < v; ++i) {
        ED cur = edge[i];

        if (find(cur.start) != find(cur.end)) { // 사이클이 생기지 않는지 검사
            merge(cur.start, cur.end);
            result += cur.weight;
            merged_count += 1;
        }
    }

    printf("%d", result);
    return 0;
}
```





#### 프림 알고리즘 (Prim Algorithm)

하나의 정점에서 시작해서 점차적으로 **인접한 정점들을 이어나가는** 방법. 정점을 중심으로 진행되는 알고리즘이다. 시작점은 <u>아무거나</u> 잡아도 된다.

일반적인 과정은 아래와 같다.

1. 어떤 한 정점을 시작점으로 지정한다
2. 모든 정점이 연결될 때까지 반복한다
   - 현재까지 만들어진 MST에 속하지 않으면서 **인접한 다른 정점들**을 MST에 이을 수 있는지 검사한다
     - 그러한 정점들이 여러 개라면, 가능한 간선들 중 **가중치가 최소인 것**을 선택한다
3. 선택된 간선들이 MST를 이루는 간선들이다

프림 알고리즘은 현재까지 연결된 정점들에서 다른 정점을 연결할 수 있는지를 확인한다. 시작지점이 정해지기 때문에, 특정 시점까지의 정점들을 그룹으로 저장할 필요까지는 없고, **현재 MST에 포함되어있는가**만 저장해두면 된다. 다만 <u>MST와 인접한 정점</u>을 항상 찾아야하는 과정은 번거롭다. 심지어 그러한 정점들 중 <u>사용할 간선의 가중치가 최소인 것</u>을 또 찾아야 한다.

그 과정을 쉽게 하기 위해, 간선의 가중치를 기준으로 [우선순위 큐](https://joe2357.github.io/posts/Heap/) 구조를 이용할 수 있다. 선택할 수 있는 간선들 중 **가중치가 가장 적은 것**을 빠르게 찾아줄 수 있다. 그러면 우리는 '선택할 수 있는 간선이 무엇인지?', '선택한 간선이 **사이클을 만들지 않는지?**'를 고려하면 된다. 선택할 수 있는 간선은 지금까지 선택된 정점들로부터 뻗어나가는 간선들이 될 것이고, 사이클을 만들지 않는 조건은 간선의 두 끝점이 모두 이미 MST에 포함되어있지 않아야 한다.

예시 구현 코드는 아래와 같다.

```c
/* 프림 알고리즘 */

bool isHeapNotEmpty() { return heapTop > 0; }
void insertHeap(ED x) {
    heap[++heapTop] = x;
    int i = heapTop;

    while (i != 1 && x.weight < heap[i / 2].weight) {
        heap[i] = heap[i / 2];
        heap[i / 2] = x;
        i /= 2;
    }
    return;
}
ED popHeap() {
    ED retval = heap[1];
    heap[1] = heap[heapTop--];

    int i = 1, tp = 2;
    while (tp < heapTop) {
        tp += (heap[tp].weight > heap[tp + 1].weight);
        if (heap[i].weight <= heap[tp].weight) {
            break;
        }

        ED a = heap[tp];
        heap[tp] = heap[i], heap[i] = a;
        i = tp, tp *= 2;
    }

    if (tp == heapTop && heap[i].weight > heap[tp].weight) {
        ED a = heap[tp];
        heap[tp] = heap[i], heap[i] = a;
    }

    return retval;
}

int main() {
    /* 정점, 간선 정보 입력 */

    // 시작점에 대한 전처리 진행
    isMerged[START] = true;
    LN* s = edge[START];
    while (s != NULL) {
        insertHeap(s->edge);
        s = s->next;
    }

    int result = 0;
    
    // 힙 안에 존재하는 모든 간선에 대해 진행
    while (isHeapNotEmpty() == true) {
        ED cur = popHeap();

        if (isMerged[cur.end] == false) { // 현재 MST에 존재하지 않는 정점이라면, MST에 포함시키면서 진행
            isMerged[cur.end] = true;
            result += cur.weight;

            // 선택된 정점으로부터 갈 수 있는 모든 방향을 고려
            LN* s = edge[cur.end];
            while (s != NULL) {
                insertHeap(s->edge);
                s = s->next;
            }
        }
    }

    printf("%d", result);
    return 0;
}
```

