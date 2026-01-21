---
title: "Sorting Algorithm"
author: Joe2357
categories: [Storage, Algorithm]
tags: [Algorithm]
uploaded_at: 2026-01-21 18:45:00 +0900
last_modified_at: 2026-01-21 18:45:00 +0900
description: "- 다양한 정렬 알고리즘의 원리와 특징"
math: true
---



정렬 알고리즘은 무작위로 나열된 데이터를 특정 기준(오름차순, 내림차순 등)에 따라 재배열하는 과정이다. 효율적인 정렬은 탐색이나 이분 탐색과 같은 다른 알고리즘의 전제 조건이 되기도 한다. 요청한 6가지 알고리즘에 대해 정리했다.



## 버블 정렬 (Bubble Sort)

**버블 정렬**은 인접한 두 원소를 비교하여 조건에 맞지 않으면 자리를 바꾸는 과정을 반복하는 방식이다. 정렬 과정에서 큰 값이 마치 거품이 올라오듯 뒤로 밀려나기 때문에 이런 이름이 붙었다.

- **원리**: 첫 번째 원소부터 마지막 원소까지 인접한 원소와 비교하며 교환한다. 한 번의 순회가 끝나면 가장 큰 원소가 맨 뒤에 고정된다.
- **시간 복잡도**: $O(n^2)$
- **특징**: 구현이 매우 단순하지만 성능이 좋지 않아 실제 데이터 정렬에는 거의 쓰이지 않는다.

```c
void bubbleSort(int arr[], int n) {
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - i - 1; j++) {
            // 인접한 두 원소 비교 후 교환
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
}
```



## 삽입 정렬 (Insertion Sort)

**삽입 정렬**은 자료 배열의 모든 원소를 앞에서부터 차례대로 이미 정렬된 부분과 비교하여, 자신의 위치를 찾아 '삽입'하는 방식이다.

- **원리**: 두 번째 원소부터 시작하여 그 앞의 원소들과 비교한다. 자신이 앞의 원소보다 작다면 그 사이에 끼워 넣는 과정을 반복한다.
- **시간 복잡도**: 평균 $O(n^2)$, 최선의 경우(이미 정렬된 경우) $O(n)$
- **특징**: 데이터가 거의 정렬되어 있는 상태라면 어떤 알고리즘보다 빠르다는 장점이 있다.

```c
void insertionSort(int arr[], int n) {
    for (int i = 1; i < n; i++) {
        int key = arr[i];
        int j = i - 1;

        // key보다 큰 원소들을 오른쪽으로 한 칸씩 이동
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key; // 적절한 위치에 삽입
    }
}
```



## 기수 정렬 (Radix Sort)

**기수 정렬**은 값을 비교하지 않는 특수 정렬 방식이다. 숫자의 자릿수를 기준으로 버킷에 데이터를 나누어 담으며 정렬한다.

- **원리**: 낮은 자릿수(1의 자리)부터 시작하여 해당 자릿수의 숫자에 맞는 버킷에 넣는다. 이를 가장 높은 자릿수까지 반복한다.
- **시간 복잡도**: $O(dn)$ ($d$는 최대 자릿수)
- **특징**: 비교 연산을 하지 않아 자릿수가 적을 때 매우 빠르지만, 부동 소수점이나 문자열 정렬에는 적용하기 까다롭고 추가적인 메모리 공간이 필요하다.

```c
// 자릿수 기준 정렬을 위한 계수 정렬(Counting Sort) 보조 함수
void countingSort(int arr[], int n, int exp) {
    int output[n];
    int count[10] = {0};

    for (int i = 0; i < n; i++)
        count[(arr[i] / exp) % 10]++;

    for (int i = 1; i < 10; i++)
        count[i] += count[i - 1];

    for (int i = n - 1; i >= 0; i--) {
        output[count[(arr[i] / exp) % 10] - 1] = arr[i];
        count[(arr[i] / exp) % 10]--;
    }

    for (int i = 0; i < n; i++)
        arr[i] = output[i];
}

void radixSort(int arr[], int n) {
    int max = arr[0];
    for (int i = 1; i < n; i++)
        if (arr[i] > max) max = arr[i];

    // 1의 자리부터 시작해서 자릿수를 높여가며 정렬
    for (int exp = 1; max / exp > 0; exp *= 10)
        countingSort(arr, n, exp);
}
```



## 병합 정렬 (Merge Sort)

**병합 정렬**은 분할 정복(Divide and Conquer) 기법을 사용하는 알고리즘이다. 배열을 더 이상 나눌 수 없을 때까지 쪼갠 뒤, 다시 합치면서 정렬을 수행한다.

- **원리**: 배열을 반으로 나눈다($\log n$번). 나뉜 배열들을 정렬하며 하나의 배열로 합친다($n$번).
- **시간 복잡도**: $O(n \log n)$ (항상 일정)
- **특징**: 안정 정렬(Stable Sort)이며 최악의 경우에도 성능이 보장되지만, 병합 과정에서 임시 배열이 필요하여 메모리 사용량이 많다.

```c
void merge(int arr[], int l, int m, int r) {
    int n1 = m - l + 1;
    int n2 = r - m;
    int L[n1], R[n2];

    for (int i = 0; i < n1; i++) L[i] = arr[l + i];
    for (int j = 0; j < n2; j++) R[j] = arr[m + 1 + j];

    int i = 0, j = 0, k = l;
    while (i < n1 && j < n2) {
        if (L[i] <= R[j]) arr[k++] = L[i++];
        else arr[k++] = R[j++];
    }
    while (i < n1) arr[k++] = L[i++];
    while (j < n2) arr[k++] = R[j++];
}

void mergeSort(int arr[], int l, int r) {
    if (l < r) {
        int m = l + (r - l) / 2;
        mergeSort(arr, l, m);
        mergeSort(arr, m + 1, r);
        merge(arr, l, m, r);
    }
}
```



## 퀵 정렬 (Quick Sort)

**퀵 정렬** 역시 분할 정복을 사용하지만, 병합 정렬과 달리 '피벗(Pivot)'이라는 기준점을 사용한다.

- **원리**: 피벗을 하나 선택하고, 피벗보다 작은 값은 왼쪽, 큰 값은 오른쪽으로 옮긴다(분할). 분할된 두 그룹에 대해 재귀적으로 같은 과정을 수행한다.
- **시간 복잡도**: 평균 $O(n \log n)$, 최악의 경우(이미 정렬된 경우 피벗 선택이 잘못되면) $O(n^2)$
- **특징**: 평균적으로 가장 빠르며 추가 메모리 공간을 거의 사용하지 않아(In-place sort) 가장 널리 사용된다.

```c
void swap(int* a, int* b) {
    int t = *a;
    *a = *b;
    *b = t;
}

int partition(int arr[], int low, int high) {
    int pivot = arr[high]; // 피벗을 마지막 원소로 선택
    int i = (low - 1);

    for (int j = low; j <= high - 1; j++) {
        if (arr[j] < pivot) {
            i++;
            swap(&arr[i], &arr[j]);
        }
    }
    swap(&arr[i + 1], &arr[high]);
    return (i + 1);
}

void quickSort(int arr[], int low, int high) {
    if (low < high) {
        int pi = partition(arr, low, high);
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}
```



## 보고 정렬 (Bogo Sort)

**보고 정렬**은 일명 '원숭이 정렬'이라고도 불리는, 유머 섞인 비효율적 알고리즘이다.

- **원리**: 배열의 원소를 무작위로 섞은 다음, 정렬되었는지 확인한다. 정렬될 때까지 무한히 반복한다.
- **시간 복잡도**: 평균 $O((n+1)!)$, 최악의 경우 무한대($\infty$)
- **특징**: 절대로 실무에서 사용하면 안 된다. 운이 좋으면 한 번에 정렬될 수 있으나, 이론적으로만 존재하는 비효율의 극치이다.

```c
#include <stdlib.h>
#include <stdbool.h>

bool isSorted(int arr[], int n) {
    while (--n >= 1) {
        if (arr[n] < arr[n - 1]) return false;
    }
    return true;
}

void shuffle(int arr[], int n) {
    for (int i = 0; i < n; i++) {
        int temp = arr[i];
        int r = rand() % n;
        arr[i] = arr[r];
        arr[r] = temp;
    }
}

void bogoSort(int arr[], int n) {
    while (!isSorted(arr, n)) {
        shuffle(arr, n);
    }
}
```



## 요약 테이블

| **알고리즘**  | **평균 복잡도** | **최악 복잡도** | **특징**                     |
| ------------- | --------------- | --------------- | ---------------------------- |
| **버블 정렬** | $O(n^2)$        | $O(n^2)$        | 매우 단순함, 성능 낮음       |
| **삽입 정렬** | $O(n^2)$        | $O(n^2)$        | 정렬된 상태에서 매우 빠름    |
| **기수 정렬** | $O(dn)$         | $O(dn)$         | 비교하지 않음, 자릿수 기준   |
| **병합 정렬** | $O(n \log n)$   | $O(n \log n)$   | 일정함, 메모리 사용 많음     |
| **퀵 정렬**   | $O(n \log n)$   | $O(n^2)$        | 실무에서 가장 선호됨, 빠름   |
| **보고 정렬** | $O((n+1)!)$     | $\infty$        | 무작위 셔플링, 극도로 비효율 |