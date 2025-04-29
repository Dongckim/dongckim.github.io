---
title: "Priority Queue & Heap"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [Heap, PriorityQueue]
use_math: true
---

## Priority Queue

Priority Queue는 일반 **Queue(큐)**의 변형이다. 큐처럼 요소들이 줄을 서 있지만, 삭제(removal)는 FIFO가 아닌 우선순위(priority)에 따라 이루어진다.  
→ 즉, 가장 높은 우선순위의 요소가 먼저 제거됨.  
→ A priority queue is a data structure where access or removal is on the highest priority element in the queue

### 동일 우선순위일 때
정의상 우선순위가 같을 경우 가장 오래된 요소가 먼저 나가야 공평함. 하지만, Java의 PriorityQueue는 이 공정성을 보장하지 않음.

**Can you think of a list-based solution to this?**

List-based PQ 구현 예 → 요소를 리스트에 삽입할 때, 우선순위 기준으로 정렬된 위치에 삽입 (Insertion Sort처럼)

삭제할 때는 **맨 앞 요소 (가장 높은 우선순위)**를 제거, or 그냥 리스트에 순서 없이 삽입하고, 제거할 때마다 전체에서 가장 우선순위 높은 요소를 찾아 제거 (비효율적)

```java
import java.util.*;

class PriorityQueueList<T> {
    static class Node<T> {
        T data;
        int priority;

        Node(T data, int priority) {
            this.data = data;
            this.priority = priority;
        }
    }

    private List<Node<T>> queue = new ArrayList<>();

    public void add(T data, int priority) {
        Node<T> newNode = new Node<>(data, priority);
        int i = 0;
        while (i < queue.size() && queue.get(i).priority <= priority) {
            i++;
        }
        queue.add(i, newNode);
    }

    public T remove() {
        if (queue.isEmpty()) return null;
        return queue.remove(0).data;
    }

    public T peek() {
        if (queue.isEmpty()) return null;
        return queue.get(0).data;
    }

    public boolean isEmpty() {
        return queue.isEmpty();
    }
}
```

## Heap 

Turns out that there is a more efficient way to do it than a list
- 힙은 완전 이진 트리 (complete binary tree) 형태를 가지는 특수한 트리 구조다.

### Min-Heap 조건
마지막 레벨 전까지는 모두 채워져 있어야 하고, 마지막 레벨의 노드들은 왼쪽부터 순서대로 차 있어야 해.

1. 빈 트리이거나,

2. 루트 노드가 가장 작은 값을 가지고 있어야 하고,

3. 왼쪽과 오른쪽 서브트리도 모두 힙이어야 해.

이런 성질을 Min-Heap이라고 부른다.

### Heap representation
힙은 완전 이진 트리이기 때문에 **배열(Array)**을 사용해서 깔끔하고 효율적으로 표현할 수 있어.
- 장점: 배열은 **임의 접근(random access)**이 가능하다 → 특정 인덱스에 O(1) 시간으로 접근 가능
- 트리 구조를 별도로 포인터 없이 표현할 수 있음 → 메모리 절약

**배열에서의 인덱싱 규칙**

1. 자식 노드 (Children):
- 왼쪽 자식: 2i + 1
- 오른쪽 자식: 2i + 2

2. 부모 노드 (Parent):
- 부모: (i - 1) / 2 → int로 나눔

### Heap Inserting 
기본 흐름:  
맨 끝에 새 요소 추가
→ 배열의 마지막 위치에 add() 한다
→ 이때까지는 완전 이진 트리 성질만 만족.

힙 성질(Min-Heap or Max-Heap)이 깨졌을 수 있음
→ 새로 추가된 요소가 부모보다 작다면, 올라가야 함. **이때 사용하는 과정:** `percolateUp() (또는 bubbleUp)`
→ 새 요소를 부모와 비교하며 올바른 위치까지 반복해서 교환함

```java
class PriorityQueueList {
    private ArrayList<Integer> queue = new ArrayList<>();

    public void enqueue(int val) {
        int i = 0;
        while (i < queue.size() && queue.get(i) <= val) {
            i++;
        }
        queue.add(i, val);
    }

    public int dequeue() {
        if (queue.isEmpty()) throw new NoSuchElementException();
        return queue.remove(0);
    }
}
```

### percolateUp()
percolateUp() 연산은 최소 힙(min-heap) 또는 최대 힙(max-heap)에서 새로운 원소를 추가한 후 힙 속성을 회복하기 위해 사용됩니다. 이 연산은 새로운 원소가 힙 속성을 만족할 때까지 위로 "버블업(bubble up)"하여 이동시키는 작업입니다.

**percolateUp() 종료 보장**  
- `percolateUp()` 연산은 힙의 루트까지 올라가거나, 부모가 자식보다 작은 경우에만 스왑이 일어나므로 종료가 보장됩니다. 새로운 원소는 반드시 부모보다 작거나 클 경우에만 스왑되므로, 마지막에는 더 이상 스왑이 일어나지 않아 종료됩니다.

**시간 복잡도**
- `percolateUp()` 연산은 트리의 높이만큼 반복됩니다.
- 힙의 높이는 **O(log n)** 이므로, 시간 복잡도는 **O(log n)** 입니다.

### 최소값 제거 (Removing the Minimum)

최소 힙에서 최소값은 루트 노드에 위치합니다. 그러나 루트를 단순히 삭제하는 것만으로는 힙 속성을 유지할 수 없습니다. 따라서 최소값을 제거하는 방법을 살펴보겠습니다.

1. **루트를 삭제하는 대신 마지막 노드를 루트로 이동:**
    we switch the root and the right-most leaf

    힙의 마지막 노드를 루트로 이동시킵니다.  
    ![]({{site.url}}/images/2025-04-29-priorityqueue/swap.png){: .align-center}

    we can safely remove the original root but, 

    그런 다음, 힙 속성을 회복하기 위해 새로운 루트를 적절한 위치로 이동시켜야 합니다.

2. **힙 속성 회복:**

    새로운 루트가 위치한 자리에 올바르게 값을 배치하기 위해 "이동" 과정이 필요합니다.  
    **"이동"**은 루트에서부터 시작하여 자식들과 비교한 후, 힙 속성을 유지할 수 있도록 스왑을 통해 노드를 아래로 내려가게 합니다.
    -  Exact opposite of percolateUp()

3. **두 자식 중 더 작은 자식을 선택:**

    최소 힙에서는 부모가 자식보다 작아야 하므로, 루트와 그 자식들을 비교할 때 두 자식 중 더 작은 자식과 스왑을 진행  
    이 과정에서 부모가 자식보다 크면 자식과 스왑을 진행하고, 자식보다 작으면 이동을 멈춥니다.

## 힙과 우선순위 큐의 관계

**우선순위 큐(Priority Queue)**는 각 항목이 **우선순위(priority)**를 가지고 있고, 우선순위가 높은 항목이 먼저 처리되는 큐입니다. 

- 힙(특히 최소 힙 또는 최대 힙)은 우선순위 큐를 구현하는 데 매우 효율적인 자료구조입니다. 

- 우선순위 큐에서 우선순위가 높은 항목이 먼저 **dequeue(제거)**됩니다.

- **큐 연산과 힙:** 큐에서 항목을 추가하거나 제거하는 연산은 힙의 insert(삽입)와 remove-min(최소값 제거) 연산으로 구현할 수 있습니다.

### enqueue (삽입):
새로운 데이터를 큐에 삽입할 때는 힙에 데이터를 삽입하고, 이를 percolateUp(위로 올리기) 연산을 통해 올바른 위치로 이동시킵니다.

### dequeue (제거): 
가장 우선순위가 높은 항목(최소값)을 제거할 때는 힙에서 min을 제거하고, 힙 속성을 유지하기 위해 percolateDown(아래로 내리기) 연산을 수행합니다.

### 최소 힙에서 최대 힙으로 변경하기

1. 비교 연산을 반대로 변경: 최소 힙에서는 부모가 자식보다 작아야 하지만, 최대 힙에서는 부모가 자식보다 커야 합니다. 따라서 부모와 자식 비교 연산을 바꾸면 됩니다.

2. Heapify 과정: percolateUp과 percolateDown을 수행할 때 부모와 자식의 관계를 반대로 바꾸면 됩니다.

### 시간복잡도

- 삽입 연산(enqueue): O(log n) 시간 복잡도.
- 제거 연산(dequeue): O(log n) 시간 복잡도.
둘 다 힙의 높이에 비례하는 O(log n) 시간이 걸리며, 힙의 구조적 특성에 따라 효율적으로 우선순위 큐를 구현할 수 있습니다.

### 왜 다른 트리 관련 연산을 살펴보지 않나요?
우리는 우선순위 큐(priority queue)를 힙을 사용하여 구현하고 있습니다.힙은 최소값(min-heap) 또는 **최대값(max-heap)**을 빠르게 찾고 제거하는 데 유리한 자료구조입니다. 우선순위 큐에서는 기본적으로 enqueue(삽입)과 dequeue(제거) 연산만 필요합니다. 

→ 즉, 큐는 삽입과 삭제에 집중하는 자료구조입니다. 따라서 우선순위 큐에서 **remove(obj)**나 get(obj) 같은 연산을 고려할 필요는 없습니다.

```java
import java.util.ArrayList;
import java.util.List;

public class MinHeap {
    private List<Integer> heap;

    public MinHeap() {
        heap = new ArrayList<>();
    }

    // 부모 노드의 인덱스를 반환
    private int parent(int index) {
        return (index - 1) / 2;
    }

    // 왼쪽 자식 노드의 인덱스를 반환
    private int leftChild(int index) {
        return 2 * index + 1;
    }

    // 오른쪽 자식 노드의 인덱스를 반환
    private int rightChild(int index) {
        return 2 * index + 2;
    }

    // 위로 올리기 (percolateUp)
    private void percolateUp(int index) {
        while (index > 0 && heap.get(index) < heap.get(parent(index))) {
            // 부모와 자식 값을 교환
            int temp = heap.get(index);
            heap.set(index, heap.get(parent(index)));
            heap.set(parent(index), temp);
            index = parent(index);
        }
    }

    // 아래로 내리기 (percolateDown)
    private void percolateDown(int index) {
        int smallest = index;
        int left = leftChild(index);
        int right = rightChild(index);

        // 왼쪽 자식이 더 작으면
        if (left < heap.size() && heap.get(left) < heap.get(smallest)) {
            smallest = left;
        }

        // 오른쪽 자식이 더 작으면
        if (right < heap.size() && heap.get(right) < heap.get(smallest)) {
            smallest = right;
        }

        // 자식 중 더 작은 값이 있으면 교환하고 내려감
        if (smallest != index) {
            int temp = heap.get(index);
            heap.set(index, heap.get(smallest));
            heap.set(smallest, temp);
            percolateDown(smallest);
        }
    }

    // 원소 삽입
    public void insert(int value) {
        heap.add(value);
        percolateUp(heap.size() - 1);
    }

    // 원소 제거 (root 제거)
    public int remove() {
        if (heap.isEmpty()) {
            throw new IllegalStateException("Heap is empty");
        }
        int root = heap.get(0);
        int last = heap.remove(heap.size() - 1);
        
        if (!heap.isEmpty()) {
            heap.set(0, last);
            percolateDown(0);
        }
        return root;
    }

    // 임의 원소 제거 (원소를 마지막 노드와 교환 후 percolateDown 또는 percolateUp 수행)
    public void removeElement(int value) {
        // 먼저 값이 있는 인덱스를 찾아야 함
        int index = heap.indexOf(value);
        if (index == -1) {
            System.out.println("Element not found in heap");
            return;
        }
        
        // 마지막 원소와 교환
        int last = heap.remove(heap.size() - 1);
        if (index < heap.size()) {
            heap.set(index, last);
            // percolateDown 또는 percolateUp을 수행
            percolateDown(index); // 또는 percolateUp(index);
        }
    }
}
```

## 허프만 인코딩 - 최소 우선순위 큐 사용

허프만 인코딩(Huffman Encoding)은 문자의 빈도를 기반으로 최적의 접두어-free 코드를 생성하는 손실 없는 데이터 압축 알고리즘입니다. 이 과정은 **최소 우선순위 큐(min-priority queue)**를 사용하여 가장 빈도가 낮은 두 문자를 반복적으로 추출

```java
'h': 12
'd': 31
'b': 41
'f': 45
'c': 56
'g': 78
'a': 102
'i': 121
'e': 412
```

### Step 1: 최소 우선순위 큐 생성

처음에는 각 문자와 빈도 수를 포함하는 노드를 우선순위 큐에 넣습니다. 큐는 빈도수에 따라 정렬되며, 빈도가 가장 낮은 문자가 큐의 앞에 위치합니다.

### Step 2: 허프만 트리 생성

1. 두 개의 가장 빈도가 낮은 문자 추출:

    큐에서 가장 빈도가 낮은 문자 두 개를 추출합니다. 예를 들어, 'h'(12)와 'd'(31)를 추출합니다.

2. 새로운 내부 노드 생성:

    'h'와 'd'를 결합하여 새로운 내부 노드를 만듭니다. 이 노드의 빈도는 12 + 31 = 43이 됩니다. 이 새로운 노드는 허프만 트리에서 'h'와 'd'의 부모 노드 역할을 합니다.

    이 새로운 노드는 큐에 다시 삽입됩니다.
    ```json
    'b': 41, 'hd': 43, 'f': 45, 'c': 56, 'g': 78, 'a': 102, 'i': 121, 'e': 412
    ```
3. 이 과정을 반복

    큐에서 다시 두 개의 가장 빈도가 낮은 문자(혹은 노드)를 추출하여 새로운 내부 노드를 만들고, 이를 큐에 다시 삽입합니다.

    이 과정은 큐에 하나의 노드만 남을 때까지 반복됩니다.
