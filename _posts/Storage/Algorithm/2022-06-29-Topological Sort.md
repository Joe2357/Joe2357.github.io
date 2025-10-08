---
title: "Topological Sort"
author: Joe2357
categories: [Storage, Algorithm]
tags: [Algorithm]
date: 2022-06-29 12:00:00 +0900
last_modified_at: 2022-06-29 12:00:00 +0900
description: "- 위상 정렬"
math: true
---



## 위상정렬

  - 비순환 방향 그래프 ( Directed Acyclic Graph ) 에서, 정점들을 선형으로 정렬하는 방법
      - 정렬 방법 : 모든 간선 $(u, v)$에 대해 정점 $u$가 정점 $v$보다 먼저 오는 순서대로 정렬
- 같은 비순환 방향 그래프에서도 여러 개의 위상 순서가 나타날 수 있음



### 구현 방법

#### [BFS 탐색](https://en.wikipedia.org/wiki/Breadth-first_search)을 이용하는 방법

- 알고리즘 순서

  - 모든 정점에 대해 in_degree 배열을 설정 ( 이 정점으로 향하는 간선의 개수 )
  - in_degree가 0인 정점들을 queue에 추가 ( 순서의 시작점이 될 것 )
  - queue가 비게 될 때까지 반복
    - 가장 앞의 원소 a를 꺼내 순서 배열에 기록
    - a에서 갈 수 있는 간선을 통해서 도달할 수 있는 정점들 중 visit되지 않은 정점들의 in_degree를 1 낮춤
      - in_degree가 0이 되는 정점이 있다면 그 정점을 queue에 추가

- 소스코드

  ```c
  void topologicalSort() {
      for (int i = 1; i <= n; ++i) {
          
          // 시작지점 찾기
          if (deg[i] == 0) {
              queue[rear++] = i;
              visit[i] = true;
              order[len++] = i;
          }
      }
  
      // 추가진행 탐색
      while (front < rear) {
          int a = queue[front++]; // 현재 정점
          
          // 추가 진행 여부 탐색
          for (int i = 1; i <= n; ++i) {
              if (!visit[i] && matrix[a][i]) {
                  if (--deg[i] == 0) {
                      queue[rear++] = i;
                      visit[i] = true;
                      order[len++] = i;
                  }
              }
          }
      }
      return;
  }
  ```



#### [DFS 탐색](https://en.wikipedia.org/wiki/Depth-first_search)을 이용하는 방법

- 알고리즘 순서

  - 방문하지 않은 모든 정점에 대해 DFS 수행
    - 하나의 정점에서 시작하며, visit 표시를 하면서 간선을 따라 다음 정점을 따라감
    - 더 이상 진행할 수 있는 간선이 없다면 list에 그 정점을 추가 ( 가장 마지막 순서가 될 것 )
    - 역추적을 통해 이전 정점들을 방문하며, **방문하지 않은 간선이 더 있는지 파악**
      - 방문 가능한 간선이 있다면 추가진행
      - 방문 가능한 간선이 없다면 현재 정점도 list에 기록
  - 모든 정점을 방문할 때까지 반복

- 소스코드

  ```c
  void topologicalSort() {
      for (int i = 1; i <= n; ++i) {
          if (!visit[i]) { // 추가탐색 진행
              DFS(i);
          }
      }
      return;
  }
  
  void DFS(int x) {
      visit[x] = true;
      for (int i = 1; i <= n; ++i) { // 추가탐색 진행
          if (matrix[x][i] && !visit[i]) {
              DFS(i);
          }
      }
      
      order_stack[top++] = x; // 최종 정점 입력
      return;
  }
  ```



### 시간 복잡도

- 그래프를 만드는 방법이 2가지 존재
  - 인접 리스트를 사용하는 경우 $O(V+E)$
  - 인접 행렬을 사용하는 경우 $O(V^2)$

