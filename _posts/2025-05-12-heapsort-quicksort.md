---
title: "All about Sorting - HeapSort & QuickSort"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [sorting, quicksort, heapsort]
use_math: true
---

## HeapSort 메소드 정리

HeapSort는 1964년 J.W\.J. Williams가 제안한 효율적인 정렬 알고리즘으로, 기본적으로 힙(Heap) 자료구조를 활용하여 배열을 정렬합니다. Java의 PriorityQueue 클래스에서 구현할 수 있으며, 시간 복잡도는 최악의 경우 O(n log n), 공간 복잡도는 상수 시간인 O(1)입니다. 이번 글에서는 HeapSort의 기본 원리와 구현 방법을 단계별로 설명하겠습니다.

### 1. HeapSort의 기본 개념

HeapSort는 다음 세 단계로 이루어집니다:

#### 1.1 힙으로 변환 (Heapify)

* **목적:** 주어진 배열을 힙 구조로 변환하여 최대 또는 최소 힙 특성을 만족시키는 이진 트리로 만드는 단계입니다.
* **방법:** 배열의 마지막 비단말 노드부터 루트 노드까지 `siftDown` 메소드를 반복 호출하여 힙 특성을 만족하도록 배열을 정렬합니다.
* **예시:**

  * 입력 배열: `[59, 46, 32, 80, 46, 55, 87, 43, 44, 81, 95, 12, 17, 80, 75, 33, 40, 61, 16, 50]`
  * 배열을 트리로 시각화하면 다음과 같습니다:

```
                59
        46              32
    80      46      55      87
 43    44  81    95  17    12  75  80
33  40  61  16  50
```

* **Heapify 과정:** 마지막 비단말 노드부터 시작하여 루트까지 각 노드에 대해 `siftDown`을 호출하여 힙 특성을 만족시키는 방식으로 배열을 재정렬합니다. 이 과정이 끝나면 배열은 힙 구조를 갖게 됩니다.

#### 1.2 정렬 (Heap Sort)

* **목적:** 힙으로 변환된 배열에서 최대값(또는 최소값)을 반복적으로 추출하여 정렬된 배열을 생성하는 단계입니다.
* **방법:**

  * 가장 큰 값을 배열의 끝으로 보내고, 힙의 크기를 줄이며 `siftDown`을 반복하여 힙 구조를 유지합니다.
  * 이 과정을 통해 배열이 내림차순으로 정렬됩니다.

#### 1.3 배열 반전 (Reverse)

* **목적:** 힙 정렬의 결과는 내림차순이므로, 최종적으로 배열을 반전하여 오름차순으로 변환합니다.

### 2. 구현 코드

```java
public void heapSort(Object[] a) {
    queue = a;
    int length = queue.length;
    size = length;
    // 힙으로 변환
    for (int i = (size >> 1) - 1; i >= 0; i--)
        siftDown(i, (E)queue[i]);
    
    // 내림차순 정렬
    E x;
    for (int i = 0; i < length; i++) {
        x = (E)queue[--size];
        queue[size] = queue[0];
        siftDown(0, x);
    }
    
    // 배열 반전
    for (int i = 0; i < length / 2; i++) {
        x = (E)queue[i];
        queue[i] = queue[length - i - 1];
        queue[length - i - 1] = x;
    }
}
```

### 3. 시간 복잡도 분석

* **Heapify 단계:** O(n log n) (실제로는 O(n)으로 최적화 가능)
* **정렬 단계:** O(n log n)
* **반전 단계:** O(n)
* **전체:** O(n log n)

### 4. 특성

* **제자리 정렬 (In-Place Sort):** 추가 메모리를 거의 사용하지 않음
* **불안정 정렬 (Unstable Sort):** 동일한 값의 상대적인 순서를 보장하지 않음

HeapSort는 빠른 성능과 간단한 구현 덕분에 널리 사용되는 정렬 알고리즘 중 하나입니다. 다만, 안정성을 필요로 하는 경우에는 다른 정렬 알고리즘을 고려해야 합니다.

### 5. HeapSort_PPT 

![]({{site.url}}/images/2025-05-12-heapsort-quicksort/HeapSort.png){: .align-center}

#### 5-1. While 루프의 반복 횟수
```java
while (!h.isEmpty()) {
    print(h.removeMin());
}
```
- 이 루프는 힙이 비어있을 때까지 반복됩니다.
- 배열의 길이가 n이라면, 힙에서 모든 요소를 하나씩 제거하므로 총 n번 반복됩니다.

#### 5-2. `removeMin()` 메소드의 시간 복잡도
Heap에서의 `removeMin()`:
- 힙에서 최소값을 제거한 후, 힙 특성을 유지하기 위해 siftDown이 호출됩니다.
- 이 과정은 힙의 높이에 비례하여 동작합니다. → 최대 힙의 높이: log n (완전 이진 트리 구조)

따라서 removeMin()의 시간 복잡도는 **O(log n)**입니다.

#### 5-3. 전체 알고리즘의 총 시간 복잡도
Heap 생성 (buildHeap): 배열을 힙 구조로 변환하는 과정의 시간 복잡도는 **O(n)**입니다. 이 단계에서 모든 비단말 노드에 대해 siftDown이 호출되기 때문에 최적화된 형태로 **O(n)**가 됩니다.

- 루프 반복: 위에서 언급한 while 루프는 n번 반복되며, 각 반복에서 O(log n) 시간이 소요됩니다. 따라서 이 단계의 총 시간 복잡도는 **O(n log n)**입니다.

- 전체 복잡도: 
    - 초기 힙 생성: O(n)
    - 요소 제거: O(n log n)
    - 최종 복잡도: O(n log n)


## 퀵 정렬 (Quick Sort)

퀵 정렬은 \*\*C.A.R. 호어(C.A.R. Hoare)\*\*가 1962년에 개발한 매우 효율적이고 널리 사용되는 정렬 알고리즘입니다. Java의 `Arrays` 클래스에서 퀵 정렬은 기본 자료형 배열을 정렬하는 `sort` 메서드로 구현되며, 객체 배열의 경우 \*\*병합 정렬(Merge Sort)\*\*이 사용됩니다. 기본 자료형 배열에 대해 각각 `int`, `byte`, `short`, `long`, `char`, `double`, `float` 타입에 대한 7가지 버전의 퀵 정렬 메서드가 있으며, 각각은 타입에 따라 약간의 차이가 있지만 기본적인 로직은 동일합니다.

![]({{site.url}}/images/2025-05-12-heapsort-quicksort/quick-sort.png){: .align-center}

---

### **sort 메서드의 기본 구조**

```java
/**
 * 지정된 int 배열을 오름차순으로 정렬합니다.
 * 최악의 시간 복잡도: O(n^2)
 * 평균 시간 복잡도: O(n log n)
 * @param a - 정렬할 배열
 */
public static void sort(int[] a) {
    sort1(a, 0, a.length);
}
```

이 메서드는 `sort1`이라는 **비공개(private)** 메서드를 호출하여 실제 정렬을 수행합니다.

---

### **sort1 메서드의 동작 원리**

`sort1` 메서드는 배열의 일부분을 재귀적으로 분할하여 정렬하는 핵심 메서드입니다. 기본적인 아이디어는 다음과 같습니다.

1. **피벗 선택**

   * 먼저 배열에서 하나의 요소를 피벗(pivot)으로 선택합니다.
   * 피벗은 배열을 왼쪽 부분 배열과 오른쪽 부분 배열로 나누는 기준이 됩니다.
   * 일반적으로 배열의 시작, 중간, 끝 요소 중 **중앙값**(median)을 피벗으로 선택하여 성능을 최적화합니다.

2. **분할 (Partition)**

   * 피벗을 기준으로 **작거나 같은** 값들은 왼쪽 부분 배열로, **큰** 값들은 오른쪽 부분 배열로 이동시킵니다.
   * 이 과정을 통해 피벗보다 작은 모든 요소가 왼쪽, 큰 모든 요소가 오른쪽으로 분할됩니다.

3. **재귀 호출 (Recursive Call)**

   * 분할이 완료된 왼쪽과 오른쪽 부분 배열에 대해 각각 `sort1`을 재귀적으로 호출하여 정렬을 완료합니다.

---

### **피벗 선택 최적화 (Median of Three)**

* 피벗 선택은 정렬 성능에 큰 영향을 미칩니다.
* 단순히 첫 번째 요소를 피벗으로 선택하면, 이미 정렬된 배열의 경우 \*\*O(n^2)\*\*의 최악의 성능을 보일 수 있습니다.
* 이를 방지하기 위해, **Median of Three** 방식을 사용하여 피벗을 선택합니다.
* 예를 들어, 다음과 같이 세 위치의 요소에서 중앙값을 피벗으로 선택합니다.

```java
private static int med3(int[] x, int a, int b, int c) {
    return (x[a] < x[b] ?
            (x[b] < x[c] ? b : x[a] < x[c] ? c : a) :
            (x[b] > x[c] ? b : x[a] > x[c] ? c : a));
}
```

---

### **정렬 로직 구현**

```java
private static void sort1(int[] x, int off, int len) {
    // 피벗 선택
    int m = off + (len >> 1); // 배열의 중간 인덱스
    int l = off;
    int n = off + len - 1;
    m = med3(x, l, m, n); // 세 개의 중앙값을 피벗으로 선택
    int v = x[m]; // 피벗 값
    int b = off;
    int c = off + len - 1;

    // 분할 과정
    while (true) {
        // 왼쪽에서 피벗보다 큰 값 찾기
        while (b <= c && x[b] < v) b++;
        
        // 오른쪽에서 피벗보다 작은 값 찾기
        while (c >= b && x[c] > v) c--;
        
        // 포인터가 엇갈리면 분할 종료
        if (b > c) break;
        
        // 스왑 후 인덱스 이동
        swap(x, b++, c--);
    }

    // 재귀 호출
    if (c + 1 - off > 1)    // 범위: off부터 c까지의 배열
        sort1(x, off, c + 1 - off);
    if (off + len - b > 1)    // 범위: b부터 현재 배열의 끝까지 
        sort1(x, b, off + len - b);
}
```

---

### **요소 교환 (Swapping)**

```java
private static void swap(int[] x, int a, int b) {
    int t = x[a];
    x[a] = x[b];
    x[b] = t;
}
```

---

### **시간 복잡도 분석**

* **최악의 경우 (Worst Case)**

  * 피벗 선택이 매우 비효율적인 경우 (`O(n^2)`)
  * 이미 정렬된 배열 또는 역순 배열

* **평균적인 경우 (Average Case)**

  * 대부분의 무작위 배열 (`O(n log n)`)
  * 재귀적 분할이 균등하게 이루어질 때

---

### **공간 복잡도**

* **최악의 경우**: O(n) (재귀 깊이와 관련)
* **평균적인 경우**: O(log n)

---

### **특징**

* **비안정 정렬 (Unstable Sort)**

  * 같은 값을 가진 요소의 상대적인 순서가 바뀔 수 있습니다.
* **분할 정복 알고리즘 (Divide and Conquer)**

  * 배열을 재귀적으로 분할하여 정렬하는 방식입니다.
