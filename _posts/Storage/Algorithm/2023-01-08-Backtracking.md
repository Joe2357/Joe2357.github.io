---
title: "Backtracking"
author: Joe2357
categories: [Storage, Algorithm]
tags: [Algorithm]
description: "- 백트래킹 알고리즘"
math: true
---



## 백트래킹

  - 탐색 과정 중 특정 상황에서 이미 해가 아니라고 판단되면, <u>그 이후를 탐색하지 않고 되돌아가서</u> 다른 해를 찾는 방법
        - **유망하다 (promising)** : 현재 탐색 중인 상황에서는 해가 될 만한 상황이 존재한다
        - **가지치기 (pruning)** : 이미 유망하지 않은 상황이라고 판단되면 탐색하지 않는 것
  - 주로 트리나 그래프에서 DFS 탐색을 진행할 때, 효율성을 높이기 위해서 사용
      - DFS : 가능한 모든 경로를 탐색하므로, 이미 가망이 없어진 상황에서도 탐색을 진행
      - 백트래킹 : 가능한 경로를 탐색하다, 가망이 없어진 상태라면 탐색하지 않고 뒤로 되돌아감



### 구현 방법

- 기존 DFS 방식

  ```pseudocode
  /* 트리, 그래프 : 인접행렬 */
  define matrix[N][N] -> boolean 2D array
  
  // DFS 방식 : 재귀함수로 주로 구현
  def DFS(graph, current):
      ... make some operations with current vertex
      
      loop i = [1, N]:
          if 'vertex(i) was not visited' and 'there is a path from vertex(current) to vertex(i)':
              visit[i] <= true
              DFS(graph, i)
      return
  ```

- 백트래킹 방식

  ```pseudocode
  /* 트리, 그래프 : 인접행렬 */
  define matrix[N][N] -> boolean 2D array
  
  // 백트래킹 방식 : DFS 방식을 기본으로 구현
  def backtracking(graph, current):
      /* 조건 추가 : 유망한지 검사 */
      if 'not promising(current)':
          return
          
      ... make some operations with current vertex
      
      loop i = [1, N]:
          if 'vertex(i) was not visited' and 'there is a path from vertex(current) to vertex(i)':
              visit[i] <= true
              backtracking(graph, i)
      return
  ```

  