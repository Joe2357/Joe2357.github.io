---
title: "Binary Search"
author: Joe2357
categories: [Storage, Algorithm]
tags: [Algorithm]
math: true
---

  > 이진 탐색 / 이분 탐색

## 이분 탐색이란?

이진 탐색 알고리즘은 **정렬되어있는** 리스트로부터 원하는 값이 존재하는지 탐색하는 알고리즘이다.

리스트에 있는 원소들은 오름차순 (혹은 내림차순) 으로 정렬되어있으므로, 우리는 리스트에서 찾으려는 값이 특정 index로부터 '왼쪽에 있을지' 혹은 '오른쪽에 있을지' 알아낼 수 있다. 만약 알아냈다면, 그 **반대에 위치한 값들**은 탐색할 필요가 없어지게 된다.

위 상식을 이용하여 리스트의 모든 범위를 탐색하지 않아도 리스트 내에 원하는 값이 존재하는지 확인할 수 있다. 그렇다면 그 특정 index를 어디에 위치시킬 것인지가 관건인데, 일반적으로는 <u>탐색하려는 범위의 정중앙</u>에 위치시킨다. 정중앙을 기준으로 탐색한다면 탐색 범위를 항상 반으로 줄일 수 있게 된다. 이때의 시간 복잡도는 $O(\log_{2} N)$이 된다.

## 구현 방법

```c
/*
  이진 탐색 알고리즘 :: 기본형
  시간 복잡도 : O(log N)
*/
int binary_search(int left, int right, int value) {
    while (left <= right) {
        int mid = (left + right) / 2;
        
        if (arr[mid] == value) {  /* 원하는 값을 찾은 경우 */
            return mid;
        }
        else if (arr[mid] > value) {  /* 원하는 값보다 더 큰 경우 */
            right = mid - 1;
        }
        else {  /* 원하는 값보다 더 작은 경우 */
            left = mid + 1;
        }
    }
    
    return -1;  /* 원하는 값이 존재하지 않은 경우 */
}
```



## 파생 알고리즘

이진 탐색 알고리즘은 정렬되어있는 리스트에서 **특정 값**이 어디에 존재하는가를 찾아내는 알고리즘이라고 하였다. 아래의 알고리즘들은 한 단계 더 나아가서 특정 조건을 만족하는 위치를 찾는 알고리즘이 되겠다.

### 1. Lower Bound

lower bound 함수는 찾으려는 값 **이상**이 처음 나타나는 위치를 찾는 알고리즘이다. 똑같은 원소가 여러개 있어도 상관없이 가장 첫 위치를 나타낸다.

리스트 내에 찾으려는 값 이상의 원소가 없는 경우 배열의 끝 ($N$) 을 출력해야 한다. 그렇기 때문에 탐색 범위 또한 배열의 끝을 포함하여 계산해야 한다. ($0$ ~ $N$)



```c
/*
  이진 탐색 알고리즘 :: lower bound
  시간 복잡도 : O(log N)
*/
int lower_bound(int left, int right, int value) {
    while (left < right) {
        int mid = (left + right) / 2;
        
        if (arr[mid] < value) {
            left = mid + 1;
        }
        else {
            right = mid;
        }
    }
    
    return mid + 1;
}

int main() {
    for (int i = 0; i < n; ++i) {
        scanf("%d", arr + i);
    }
    
    int target = 10;
    printf("%d\n", lower_bound(0, n, target));
    
    return 0;
}
```

### 2. Upper bound

upper bound 함수는 찾으려는 값 **초과**가 처음 나타나는 위치를 찾는 알고리즘이다. 똑같은 원소가 여러개 있어도 상관없이 가장 첫 위치를 나타낸다.

리스트 내에 찾으려는 값 이상의 원소가 없는 경우 배열의 끝 ($N$) 을 출력해야 한다. 그렇기 때문에 탐색 범위 또한 배열의 끝을 포함하여 계산해야 한다. ($0$ ~ $N$)



```c
/*
  이진 탐색 알고리즘 :: upper bound
  시간 복잡도 : O(log N)
*/
int upper_bound(int left, int right, int value) {
    while (left < right) {
        int mid = (left + right) / 2;
        
        if (arr[mid] <= value) {
            left = mid + 1;
        }
        else {
            right = mid;
        }
    }
    
    return mid + 1;
}

int main() {
    for (int i = 0; i < n; ++i) {
        scanf("%d", arr + i);
    }
    
    int target = 10;
    printf("%d\n", upper_bound(0, n, target));
    
    return 0;
}
```