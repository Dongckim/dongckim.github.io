---
title: "Binary Trees"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [Tree, BinaryTree]
use_math: true
---

> 1. Understand binary-tree concepts and important properties, such as the Binary Tree Theorem and the External Path Length Theorem.
> 2. Be able to perform various traversals of a binary tree.

## Definition of Binary Tree

A binary tree t is either empty or consists of an element, called the root element, and two distinct binary trees, called the left subtree and right subtree of t.
→ **root요소 + left subtree + right subtree**

- 이진트리는 재귀적으로 정의되며, 이진트리 관련 많은 정의들도 자연스럽게 재귀적
- **Functional notation** (예: leftTree(t))을 사용하는 이유는, **Object notation** (예: t.leftTree())을 사용하기에는 통일된 이진트리 자료구조가 존재하지 않기 때문.
- 다양한 이진트리들은 삽입(insert)이나 삭제(remove) 등의 연산에 대해 서로 다른 메서드나 파라미터 리스트를 가진다

![]({{site.url}}/images/2025-04-27-binarytree/trees.png){: .align-center}

1. (a)와 (b)의 이진트리는 구조가 다르다.
2. 서브트리(subtree), 이진트리의 서브트리도 하나의 이진트리이다.

## Properties of Binary Trees

- Branch: 루트 요소에서 서브트리로 가는 선.
- Leaf (잎 노드): 왼쪽과 오른쪽 서브트리가 모두 빈 노드. 즉, 자식이 없는 노드. *ex) Figure 9.1e에서는 15, 28, 36, 68이 leaf이다.*

```java
if t is empty
    leaves(t)= 0
else if t consists of a root element only
    leaves(t)= 1
else
    leaves(t)= leaves(leftTree(t)) + leaves(rightTree(t))
```
- 왼쪽 서브트리와 오른쪽 서브트리의 leaf 수를 합친 것이 전체 tree의 leaf 수
- 트리의 각 요소는 위치(location) 로 고유하게 결정된다.
- 같은 값(-)을 가진 두 노드도 "어디에 위치했는가"로 구별할 수 있다.
- 실제로는 "요소"를 언급할 때 "특정 위치의 요소"를 뜻한다.

### Familial Terms
- **Sibling:** 같은 부모를 공유하는 노드.
- **Ancestor:** A가 B의 조상이라면, B는 A로부터 이어진 서브트리에 존재한다.
    - 재귀적 정의: A가 B의 부모이거나, A가 B의 부모의 조상이다.
- **Descendant (후손):** B가 A의 후손이면, A가 조상이고 B는 그 하위 서브트리에 있다.
    - 재귀적 정의: B가 A의 자식이거나, B가 A의 자식의 후손이다.

### 높이(Height)

루트에서 가장 먼 leaf까지 가는 경로상의 branch 수를 트리의 높이라고 한다.
```java
if t is empty,
    height(t)= −1
else
    height(t)= 1+ max (height(leftTree(t)), height(rightTree(t)))
```
- height(t):
    - t가 비어있으면 → -1
    - t가 루트 하나만 가지면 → 0
    - 그 외 → 1 + max(height(leftTree(t)), height(rightTree(t)))
    - 주의: 빈 트리의 높이는 -1로 정의한다.

> **빈 서브트리의 높이 = -1로 정의!!**

> **요소 하나만 있는 트리의 높이 = 0**

### 레벨(Level)

루트로부터 요소 x까지 몇 개의 branch를 거치는지 나타냄.** Depth = Level (같은 개념)**
```java
if x is the root element,
    level(x)= 0
else
    level(x)= 1+ level(parent(x))
```
- level(x) = 0 (x가 루트이면)
- level(x) = 1 + level(parent(x)) (x가 루트가 아니면)
> 트리의 높이(height) = 가장 깊은 leaf의 레벨(depth)

## Two-Tree (2-tree)

![]({{site.url}}/images/2025-04-27-binarytree/two-tree.png){: .align-center}

- 트리가 비어 있거나,
- 모든 비-leaf 노드가 왼쪽과 오른쪽 둘 다 자식을 가질 때.
- Figure 9.5a는 → two-tree (모든 비-leaf가 자식 2개)
- Figure 9.5b는 → two-tree 아님 (어떤 노드는 자식 1개만 가짐)


### Full Binary Tree

높이가 h인 full binary tree는 정확히 2^(h+1) - 1개의 요소를 가짐.

### Complete Binary Tree

트리가 **다음-하위 레벨(next-to-lowest level)**까지는 꽉 차 있어야 하고, 가장 마지막 레벨의 노드들은 왼쪽부터 채워져야 함.

- 모든 full binary tree는 complete binary tree임.
- 하지만 complete binary tree가 반드시 full은 아님.

![]({{site.url}}/images/2025-04-27-binarytree/binary-tree.png){: .align-center}

- Figure 9.8a는 complete binary tree (하지만 full은 아님).
- Figure 9.8b는 next-to-lowest level이 꽉 차지 않아 complete 아님.
- Figure 9.8c는 마지막 레벨이 왼쪽부터 채워지지 않아 complete 아님.

## Position of Complete BT

- 루트: position = 0
- 왼쪽 자식: position = 2i + 1
- 오른쪽 자식: position = 2i + 2 (여기서 i는 부모 노드의 position)

즉, 부모 i로부터 왼쪽/오른쪽 자식의 위치를 간단히 계산할 수 있음.

### 빠른 계산을 위해 비트 연산 사용:

왼쪽 쉬프트 << 1 → 2배 곱하기와 같음.

```java
(i << 1) + 1 and (i << 1) + 2
```
그래서 자식의 위치:
- 왼쪽 자식: (i << 1) + 1
- 오른쪽 자식: (i << 1) + 2

부모 위치 계산:
- 자식 i에 대해 부모 위치는 (i - 1) >> 1
- (오른쪽 쉬프트 >> 1 → 2로 나누기)

**Complete binary tree는 ArrayList나 배열로 쉽게 구현 가능! 부모-자식 관계를 index 연산만으로 O(1) 시간에 찾을 수 있음.**


### 트리 요소수 세기 n(t)

```java
if t is empty:
    n(t) = 0
else:
    n(t) = 1 + n(leftTree(t)) + n(rightTree(t))
```
- Figure 9.10:

Complete binary tree를 array로 표현.
- 각 요소를 position에 맞는 인덱스에 저장함.
- Complete binary tree를 array로 표현가능.

각 요소를 position에 맞는 인덱스에 저장함.
- 즉, 현재 노드 1개 + 왼쪽 서브트리의 요소 수 + 오른쪽 서브트리의 요소 수.

## Binary Tree Theorem

임의의 non-empty 이진 트리 t에 대해:
- leaves(t) ≤ n(t)
- leaves(t) = n(t) ⟺ 동트리 t가 단 하나의 요소만 가질 때 (leaf가 root와 동일)

For any non-empty binary tree t,
- $leaves(t) \le \frac{n(t) + 1}{2.0}$
    - 1번의 등호가 성립 ⟺ t는 two-tree (각 노드가 0개나 2개의 자식을 가짐)

- $\frac{n(t) + 1}{2.0} \le 2^{height(t)}$
    - 2번의 등호가 성립 ⟺ t는 full (가득 찬 트리)

### Special Case: Chain
- **Chain**: 각 노드가 자식 하나만 갖는 트리.
- Chain에서는:
    - `height(t) = n(t) - 1`
    - 즉, 높이가 **선형(linear)** (O(n)).

그래서 대부분의 트리 기반 알고리즘은 **높이를 log n으로 유지하는 것**이 핵심 목표 → 높이가 커지면 탐색, 삽입, 삭제 시간 복잡도가 O(n)까지 늘어날 수 있음.


## External Path Length 

이 정리는 이진 트리의 외부 경로 길이(E(t))에 대한 하한선을 제공합니다. 외부 경로 길이는 트리의 모든 리프(말단 노드)의 깊이 합을 의미합니다.
E(t) 는 다음과 같은 하한을 갖는다. → $E(t) \ge \frac{k}{2} * \lfloor \log_2 k \rfloor$

- 완전 이진 트리: 완전 이진 트리의 높이가 ℎ일 때, 그 트리는 $2^h$개의 리프를 가짐.
- 트리 높이가 log2K일 때, 이 트리의 리프는 최대 k개임. 높이가 log2K이면, 리프의 수는 k개 이하로 제한.
- 이 레벨보다 한 단계 낮은 레벨에서 리프들을 제거했을 때, 남은 트리의 높이는 Log2K - 1임, 즉 이 트리에는 리프가 최대 k/2개 있을 수 있음.

외부경로의 길이는 최소한 $\frac{k}{2} * \log_2 k $

## Traversals of a Binary Tree

- 트리 순회(Traversal): 트리에서 각 요소를 정확히 한 번씩 처리하는 알고리즘.
- 알고리즘의 초점: 이진 트리 순회에 대한 알고리즘에 초점을 맞추고 있으며, BinaryTree 클래스나 인터페이스를 선언하는 것은 포함되지 않습니다.
    - 이유: BinaryTree 클래스나 인터페이스는 Java Collections Framework에서 이미 다양한 삽입 및 제거 방법을 지원하고 있기 때문에, 그것보다 더 유연한 방법이 필요합니다.

### in-order순회 (Left - Root - Right)
이 알고리즘은 재귀적으로 동작함. 순회의 기본 아이디어는 이렇다.

```java
inOrder(t)
{
    if (t is not empty)
    {
        inOrder(leftTree(t)); // 왼쪽 서브트리 순회
        process the root element of t; // 루트 처리
        inOrder(rightTree(t)); // 오른쪽 서브트리 순회
    } 
}
```

1. 왼쪽 서브트리에서 inOrder순회를 먼저 수행함.
2. 루트요소를 처리함.
3. 오른쪽 서브트리에서 inOrder 순회를 수행함.

*시간 복잡도 분석*
n은 트리 내의 요소 개수입니다. 트리의 각 요소마다 왼쪽 서브트리와 오른쪽 서브트리에 대한 순회를 각각 한 번씩 수행하게 되므로, 트리 내 모든 노드에 대해 2번의 재귀 호출이 발생합니다. 즉, 각 요소에 대해 2번씩 재귀가 호출되므로, 전체 호출 횟수는 2n이 됩니다.

따라서 **최악의 경우 시간 복잡도(worstTime(n))**와 **평균 시간 복잡도(averageTime(n))**는 **O(n)**입니다.

### post-Order 순회 (Left - Right - Root)
postOrder 순회의 알고리즘은 왼쪽 서브트리와 오른쪽 서브트리를 먼저 순회한 후, 마지막에 루트를 처리하는 방식입니다.
```java
postOrder(t)
{
    if (t is not empty)
    {
        postOrder(leftTree(t)); // 왼쪽 서브트리 순회
        postOrder(rightTree(t)); // 오른쪽 서브트리 순회
        process the root element of t; // 루트 처리
    }
}
```
postOrder 순회는 트리의 모든 요소를 순차적으로 처리하는 방식인데, 이 순서는 왼쪽 서브트리 → 오른쪽 서브트리 → 루트입니다.

**시간 복잡도 분석**
n은 트리 내의 요소 개수입니다.

트리의 각 요소마다 왼쪽 서브트리와 오른쪽 서브트리에 대한 순회를 각각 한 번씩 수행하므로, 트리 내의 모든 노드에 대해 2번의 재귀 호출이 발생합니다.

결과적으로, 2n번의 재귀 호출이 발생하므로 **최악의 경우 시간 복잡도(worstTime(n))**와 **평균 시간 복잡도(averageTime(n))**는 **O(n)**입니다.

### preOrder 순회 (Root - Left - Right)
preOrder 순회는 트리의 루트를 먼저 처리한 뒤, 왼쪽 서브트리와 오른쪽 서브트리를 차례대로 순회하는 방식입니다.

```java
preOrder(t)
{
    if (t is not empty)
    {
        process the root element of t; // 루트 처리
        preOrder(leftTree(t)); // 왼쪽 서브트리 순회
        preOrder(rightTree(t)); // 오른쪽 서브트리 순회
    }
}
```
preOrder 순회는 루트 → 왼쪽 서브트리 → 오른쪽 서브트리의 순서로 트리의 요소를 처리합니다.

**시간 복잡도 분석**
n은 트리 내의 요소 개수입니다. 트리의 각 요소에 대해 한 번씩 루트 처리, 왼쪽 서브트리 순회, 오른쪽 서브트리 순회가 이루어집니다.

결과적으로, 모든 노드에 대해 2n번의 재귀 호출이 발생하므로 **최악의 경우 시간 복잡도(worstTime(n))**와 **평균 시간 복잡도(averageTime(n))**는 **O(n)**입니다.

#### DFS Depth-First Search
preOrder 순회는 **깊이 우선 탐색(DFS, Depth-First Search)**의 한 예입니다. 이 방식은 왼쪽 서브트리를 최대한 깊이 탐색한 후, 오른쪽 서브트리를 탐색하는 방식입니다.

DFS에서는 트리의 깊은 곳을 우선적으로 탐색하고, 더 이상 깊이가 없을 때 오른쪽 서브트리로 이동합니다.


### breadthFirst 순회 (Level-By-Level)
breadthFirst 순회는 레벨 단위로 트리의 노드를 처리하는 방식입니다. 즉, 트리의 루트를 먼저 처리하고, 그 다음으로 루트의 자식들을 왼쪽부터 오른쪽으로 순차적으로 처리한 후, 그의 자식들을 또 왼쪽부터 오른쪽으로 순차적으로 처리하는 방식입니다.

레벨별로 서브트리의 참조를 리스트에 저장하고, 이 리스트에서 각 서브트리를 왼쪽부터 오른쪽 순으로 꺼내 처리합니다.

이러한 방식은 **큐(queue)**를 사용하여 구현할 수 있습니다. 큐는 선입선출(FIFO, First In, First Out) 방식으로 작동하므로, 입력된 순서대로 요소를 처리할 수 있습니다.

```java
breadthFirst(t)
{
    if (t is not empty)
    {
        queue = new Queue();
        queue.enqueue(t); // 루트 트리를 큐에 추가

        while (queue is not empty)
        {
            current = queue.dequeue(); // 큐에서 트리의 노드를 꺼냄
            process(current); // 현재 노드 처리

            if (current has left child) 
                queue.enqueue(leftTree(current)); // 왼쪽 자식 큐에 추가
            if (current has right child) 
                queue.enqueue(rightTree(current)); // 오른쪽 자식 큐에 추가
        }
    }
}

```

**시간 복잡도 분석**
n은 트리 내의 요소 개수입니다. breadthFirst 순회는 트리의 모든 노드를 한 번씩 처리하므로, 모든 노드를 처리하는 데 걸리는 시간은 **O(n)**입니다.

각 노드는 큐에 한 번 들어가고, 한 번 나가므로 큐에 관련된 연산은 **O(1)**입니다. 결국 전체 알고리즘의 시간 복잡도는 **O(n)**입니다.

## 완전 이진 트리와 배열 사용
만약 트리가 완전 이진 트리라면, 트리를 **배열(Array)**로 구현할 수 있습니다. 이 경우, 너비 우선 순회는 배열을 순차적으로 순회하는 방식으로 매우 효율적으로 처리됩니다.

루트 노드는 배열의 인덱스 0에 위치하고, 루트의 왼쪽 자식은 인덱스 1, 오른쪽 자식은 인덱스 2, 그 다음으로 왼쪽 자식의 자식은 인덱스 3, 이런 식으로 트리의 노드를 배열 인덱스에 맞춰 저장합니다.

배열에서는 트리의 레벨 순서를 그대로 반영할 수 있기 때문에, 배열을 한 번 순회하는 것만으로 너비 우선 순회를 구현할 수 있습니다.