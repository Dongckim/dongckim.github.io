---
title: "Binary Search Trees"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [Tree, BinarySearchTree]
use_math: true
---

BinarySearchTree는 삽입, 삭제, 검색에서 **평균적으로 O(log n)**의 시간 복잡도를 가진다. 그러나 최악의 경우에는 O(n) 시간 복잡도를 가질 수 있다. 이는 Array, ArrayList, LinkedList의 평균 O(n) 성능보다 훨씬 좋다.

1. BinarySearchTree vs ArrayList/LinkedList → 삽입, 삭제, 검색 시간 비교.
 
2. BinarySearchTree.contains vs Arrays/Collections.binarySearch → 이진 탐색(binary search)과 contains 메서드의 유사점과 차이점 이해.

3. remove 메서드와 TreeIterator.next 메서드 → 이 두 메서드는 구현이 약간 복잡한 이유 이해하기.

4. 네 가지 회전(rotations) 수행 가능 → (좌회전, 우회전, 좌-우 회전, 우-좌 회전) 이해 및 직접 수행.


# Binary Search Tree

A binary search tree t is a binary tree such that either t is empty or
1. each element in leftTree(t) is less than the root element of t;
2. each element in rightTree(t) is greater than the root element of t;
3. both leftTree(t) and rightTree(t) are binary search trees.

- inOrder traversal(중위 순회)을 하면 항상 오름차순으로 아이템들을 방문한다.
- BinarySearchTree에서는 중복 요소를 허용하지 않는다.
- BinarySearchTree의 기본 생성자(constructor)와 add 메서드 는 이 중복 허용 금지 조건을 반영해야 한다.

![]({{site.url}}/images/2025-04-28-binary-search-tree/extend.png){: .align-center}

BinarySearchTree는 Comparable 요소만 저장하고, 중복을 허용하지 않으며, 삽입/탐색/삭제는 평균 O(log n), 최악 O(n) 걸린다.

## contains() 

contains 메서드는 이진 탐색 트리(BST)에서 특정 요소(obj)를 검색하는 기능을 제공합니다. 트리가 정렬된 구조를 갖고 있기 때문에, 이 메서드는 그 구조를 활용하여 효율적으로 검색을 수행합니다.

```java
public boolean contains(Object obj) {
    Entry<E> temp = root;  // 트리의 루트에서 시작
    int comp;

    if (obj == null) {  // obj가 null인 경우 예외 발생
        throw new NullPointerException();
    }

    // 트리 탐색
    while (temp != null) {
        comp = ((Comparable)obj).compareTo(temp.element);  // obj와 현재 요소(temp.element) 비교

        if (comp == 0) {
            return true;  // 동일한 요소를 찾으면 true 반환
        } else if (comp < 0) {
            temp = temp.left;  // obj가 작으면 왼쪽 자식으로 이동
        } else {
            temp = temp.right;  // obj가 크면 오른쪽 자식으로 이동
        }
    }
    return false;  // 요소를 찾지 못하면 false 반환
}
```
- 최악의 경우 시간 복잡도: O(n) — 트리가 비정형적일 때, 예를 들어 모든 노드가 하나의 직선처럼 이어져 있을 경우.
- 평균적인 경우 시간 복잡도: O(log n) — 트리가 균형 잡힌 상태일 때, 탐색이 로그 시간 복잡도로 이루어집니다.

## add()

add 메소드는 이진 탐색 트리에 요소를 추가하는 방법을 정의합니다. 이 메소드는 트리의 루트부터 시작하여 트리 내의 적절한 위치를 찾아 새로운 요소를 삽입합니다.

```java
public boolean add(E element) {
    if (root == null) {
        if (element == null)
            throw new NullPointerException();
        root = new Entry<E>(element, null);
        size++;
        return true;
    } else {
        Entry<E> temp = root;
        int comp;
        while (true) {
            comp = ((Comparable)element).compareTo(temp.element);
            if (comp == 0)
                return false; // 이미 존재하는 요소
            if (comp < 0) { // element가 temp.element보다 작으면 왼쪽 자식으로
                if (temp.left != null)
                    temp = temp.left;
                else {
                    temp.left = new Entry<E>(element, temp);
                    size++;
                    return true;
                }
            } else { // element가 temp.element보다 크면 오른쪽 자식으로
                if (temp.right != null)
                    temp = temp.right;
                else {
                    temp.right = new Entry<E>(element, temp);
                    size++;
                    return true;
                }
            }
        }
    }
}
```

1. 빈 트리일 때:
    - 트리가 비어 있으면, 새로운 Entry 객체를 생성하여 root에 할당하고, 크기(size)를 1 증가시킨 후 true를 반환합니다.

2. 비어 있지 않은 트리일 때:
    - root부터 시작해 element와 temp.element를 비교하며 트리 아래로 내려갑니다.
    - 중복 삽입: 만약 element와 temp.element가 같으면 이미 해당 요소가 트리에 존재하므로 false를 반환합니다.
    - 왼쪽 자식으로 이동: element가 temp.element보다 작으면 왼쪽 자식(temp.left)으로 이동하고, 왼쪽 자식이 없으면 새로운 Entry 객체를 temp.left에 삽입합니다.
    - 오른쪽 자식으로 이동: element가 temp.element보다 크면 오른쪽 자식(temp.right)으로 이동하고, 오른쪽 자식이 없으면 새로운 Entry 객체를 temp.right에 삽입합니다.


## remove() 

삭제는 추가하는 것보다 복잡할 수 있으며, 삭제할 요소에 따라 트리 구조를 다시 조정해야 할 수 있습니다.

1. 삭제할 항목 찾기
    - remove 메서드는 먼저 getEntry 메서드를 호출하여 삭제할 요소를 포함하는 노드를 찾습니다.
    - 만약 요소가 없으면 false를 반환하고, 요소가 발견되면 deleteEntry 메서드를 호출하여 노드를 삭제합니다.

2. 삭제 시 처리해야 할 경우:
    - 자식이 없는 경우 (리프 노드): 부모의 자식 참조를 null로 설정합니다.
    - 자식이 하나인 경우: 부모와 자식 간의 링크를 수정하여, 삭제할 노드를 자식으로 교체합니다.

3. 자식이 두 개인 경우: 
    - 삭제할 노드의 후속자(즉, 오른쪽 서브트리에서 가장 왼쪽에 있는 노드)의 값을 삭제할 노드에 복사한 후, 후속자를 삭제합니다. 이 후속자 노드는 자식이 없거나 하나일 때 처리하는 방식에 따라 삭제됩니다.


# Balanced Binary Search Tree

1. 높이와 시간 복잡도:

    평균적인 경우에 이진 탐색 트리의 높이는 𝑂(log𝑛)로, 이때 삽입, 삭제, 탐색 연산은 효율적으로 이루어집니다. 최악의 경우에는 트리의 높이가 선형 O(n)이 될 수 있으며, 이 경우 삽입, 삭제, 탐색 연산의 시간 복잡도도 선형이 됩니다.

2. 균형 잡힌 이진 탐색 트리:

    균형 잡힌 이진 탐색 트리는 높이가 O(logn)로 유지되어야 합니다. 이를 위해 **회전(rotation)**이라는 방법을 사용하여 트리의 균형을 맞추고, 삽입이나 삭제 시 발생할 수 있는 불균형을 조정합니다.

    예를 들어, **왼쪽 회전(left rotation)**과 **오른쪽 회전(right rotation)**이 있습니다. 이 방법들은 트리의 균형을 복구하면서 트리의 구조를 재정렬합니다.

3. 균형 잡힌 이진 탐색 트리 종류:

    대표적인 균형 잡힌 이진 탐색 트리로는 AVL 트리, 레드-블랙 트리, 스플레이 트리 등이 있습니다.

# 레드-블랙 트리(Red-Black Trees)

레드-블랙 트리는 이진 탐색 트리(BST)의 일종으로, 각 노드에 빨간색(Red) 또는 검정색(Black) 색상을 부여합니다.

## Red Rule (빨간 규칙)
- 빨간색 노드는 절대로 빨간색 자식을 가질 수 없습니다. (즉, 빨간색 노드는 자식이 있다면 반드시 검정색이어야 함)

## Path Rule (경로 규칙)
- 루트에서 자식이 없거나 하나만 있는 노드까지 가는 모든 경로에는 검정색 노드의 수가 같아야 합니다.


![]({{site.url}}/images/2025-04-28-binary-search-tree/red-black.png){: .align-center}

정상적인 레드-블랙 트리 (예시 그림 12.1)
- 루트 노드는 검정색입니다.
- 빨간색 노드는 빨간색 자식을 가지지 않습니다. (Red Rule 만족)
- 루트에서 자식이 없는 노드 또는 하나만 자식을 가진 노드까지 가는 모든 경로에 검정색 노드 개수가 동일합니다. (Path Rule 만족)

→ 레드-블랙 트리가 올바르게 구성되어 있습니다.

![]({{site.url}}/images/2025-04-28-binary-search-tree/no-redblack.png){: .align-center}

레드-블랙 규칙을 위반한 경우 (예시 그림 12.2)
- 빨간색 규칙은 만족하지만, 경로 규칙을 위반합니다.
- 예를 들어, 루트(70)에서 40으로 가는 경로는 검정색 노드가 3개, 110으로 가는 경로는 검정색 노드가 4개입니다.

→ 이 트리는 Path Rule을 위반하였기 때문에 레드-블랙 트리가 아닙니다.

## Red-Black Tree의 높이 공식

h(t)≤2log(n(t)+1)

### Red-Black Tree의 성질 (중요)
1. 트리의 높이는 항상 O(logn) 이다.

2. 탐색, 삽입, 삭제가 모두 O(logn) 시간에 가능하다.

3. 일반 BST보다 더 안정적이다.

## Insertion

1.  먼저 BST 방식으로 삽입한다 → 기본적으로 Binary Search Tree(BST) 처럼 새 노드를 삽입해.
    - 즉, 작으면 왼쪽, 크면 오른쪽으로 이동하면서 위치를 찾는다.
    - Color? Red is safe (why?) 
        - 새 노드를 검정색으로 삽입하면, black-height를 깨뜨릴 위험이 있음.
        - 빨간색이면 일단 black-height 유지에 문제가 안 생긴다.

    - 문제점:
        하지만, 빨간 노드를 삽입하면 "빨간 노드가 빨간 자식을 가지지 못한다" 규칙을 위반할 수 있다.
        (예: 부모가 빨간색일 때, 새로 추가된 노드도 빨간색이면 문제가 됨.)

2. 규칙을 만족하도록 트리를 고친다
    새로 삽입한 노드에서 시작해서, 위쪽(부모, 조부모 방향)으로 올라가면서 규칙 위반을 고쳐야 한다.

### Insertion case 1: 부모가 black
x.parent.color == BLACK
- 빨간 노드는 빨간 자식을 가질 수 없다는 규칙이 있는데, 부모가 검정이면 이 규칙이 깨지지 않음.

### Insertion case 2: 부모가 빨간색
- 빨간 노드의 자식도 빨간색이 되어버려서 레드-블랙 규칙 위반. → x의 삼촌(uncle) 을 확인 (x의 부모의 형제노드 = y)

#### case1. 삼촌 y가 빨간색일 때
- x의 부모도 빨간색 🔴
- x의 삼촌(y)도 빨간색 🔴

1. 부모(x.parent)의 색을 검정색으로 바꾼다 🟥 ➔ ⬛

2. 삼촌(y)의 색도 검정색으로 바꾼다 🟥 ➔ ⬛

3. 할아버지(x의 grandparent) 색을 빨간색으로 바꾼다 ⬛ ➔ 🟥

4. x를 할아버지 노드로 이동시킨다 (x = x.grandparent)

→ '어떤 경로로 가든 검정 노드 개수 같음' 규칙 만족

#### case2. 삼촌 y는 검정색, 그리고 x는 오른쪽 자식

이 상황은 독립적인 해결 케이스가 아니야. → Case 3으로 가기 위한 "준비 작업"이야!

1. x를 부모로 설정한다 (x = x.parent)

2. x에서 왼쪽 회전 (rotateLeft(x)) 수행

3. 이제 Case 3로 이동한다

#### case3. 삼촌 y는 검정색, 그리고 x는 왼쪽 자식

삼촌(y)이 검정색 ⬛ (또는 null ➔ 검정 취급) → x는 부모의 왼쪽 자식임(균형 복구)작업이다.

1. x의 부모(parent)를 검정색으로 바꾼다 (color = BLACK)

2. x의 조부모(grandparent)를 빨간색으로 바꾼다 (color = RED)

3. x의 조부모가 존재한다면, 거기서 오른쪽 회전 (rotateRight) 수행한다

## Deletion

간단한 종료 조건 → x가 root거나 x가 RED이면 ➔ 걍 끝!
- root이면 그냥 남겨둬도 됨
- red이면 black 부모 밑에 있어서 위반 아님
이 경우, 그냥 x를 black으로 바꾸거나 삭제하면 됨.

**x가 root도 아니고, BLACK이면 규칙 위반 가능성 있음 ➔ fix-up 필요**

### Deletion Setting 4 cases

1. Case 1: colorOf(sib) == RED
    
    형제 노드 sib이 RED라면 ➔ sib의 색과 부모의 색을 서로 교환하고, 부모를 기준으로 다시 시도

    의도: 빨간 sib을 위로 보내서 black sibling이 생기도록 만듦

    "Case 1" 이후 → Case 2로 넘어감

2. Case 2: colorOf(leftOf(sib)) == BLACK && colorOf(rightOf(sib)) == BLACK
    
    sib의 양쪽 자식이 둘 다 BLACK (또는 NULL로 간주)

    ➔ sib을 RED로 칠하고 x를 부모로 올림   
    ➔ 상위 레벨에서 다시 fix-up 해야 할 수도 있음

3. Case 3: colorOf(rightOf(sib)) == BLACK
    
    sib의 오른쪽 자식만 BLACK (왼쪽은 RED일 수 있음)

    ➔ sib과 왼쪽 자식 색깔을 바꾸고, sib을 오른쪽으로 회전   
    ➔ 이 과정을 통해 Case 4로 넘어가게 유도

4. Case 4: colorOf(rightOf(sib)) == RED
    
    sib의 오른쪽 자식이 RED

    ➔ sib을 부모의 색으로 칠하고, 부모와 오른쪽 자식을 BLACK으로 칠함   
    ➔ 부모를 중심으로 왼쪽 회전   
    ➔ fix-up 완료 (루프 탈출)

```java
if (colorOf(sib) == RED) {
    // Case 1: 색깔 바꾸고 부모를 새 기준으로
}
if (colorOf(leftOf(sib)) == BLACK && colorOf(rightOf(sib)) == BLACK) {
    // Case 2: sib을 RED로 만들고 x를 위로 올림
}
else {
    if (colorOf(rightOf(sib)) == BLACK) {
        // Case 3: sib을 오른쪽 회전
    }
    else {
        // Case 4: sib을 부모 색으로 칠하고 부모를 왼쪽 회전
    }
}
```

### Case1: colorOf(sib) == RED

x는 삭제 후 "double black" 상태 (여기선 4가 삭제됨 → x는 4의 자리)

1. Set sib's color to black
2. Set X's parent's color to red
3. Rotate x's parent left
    sib이 오른쪽에 있는 RED노드이므로, sib을 부모로 끌어올려 black sibling이 생기도록 만들기 위해 회전   
    회전 후에는 이제 RED인 노드(8)가 sib이 되어, 다음 case로 진행가능.
4. Set sib to rightOf(parentOf(x))

Why is sib ← rightOf(parentOf(x)) and not left;
- 현재 x가 왼쪽 자식이기 때문에 sib은 항상 오른쪽 자식.

### Case2: colorOf(leftOf(sib)) == BLACK && colorOf(rightOf(sib)) == BLACK

Case 1에서 sib이 RED인 걸 BLACK으로 만들었기 때문에, 그 이후 sib은 BLACK임이 논리적으로 보장

1. Set sib's color to red.
2. Set x to its parent.
3. Continue on with the main loop.

### Case3: colorOf(rightOf(sib)) == BLACK

왜 sib의 왼쪽 자식이 RED임이 보장되는가? → Case 1과 Case 2가 이미 아닌 경우라는 조건

- Case 1: sib이 RED → 아님 (지금 sib은 BLACK)
- Case 2: sib의 왼쪽, 오른쪽 자식 모두 BLACK → 아님 (지금 오른쪽 자식은 BLACK이지만, 왼쪽 자식은 RED여야 함)

1. Set the color of sib's left child to black
2. Set sib's color to red
3. rotateRight(sib)
4. Set sib to be the sibling of X

It seems there's no improvement at all, but we directly move on the case 4.

### Case4: colorOf(rightOf(sib)) == RED

Color of Sib must be black by now (Why?)
Case 3에서 이미 sib을 black으로 만들었기 때문. Case 3의 마지막 단계는 rotateRight(sib) 하고, sib = sibling of x로 갱신.

1. Set sib's color to that of x's parent 
2. Set X's parent's color to black
3. Set sib's right child's color to black
4. Rotate left on x's parent
5. Set x to be the root (this is done to break out of the loop)

**Since x is now root, the loop is ended. What next?**

삭제 대상 노드 (예: 5번 노드) 는 이미 트리 구조상에서 삭제됨. 하지만 Red-Black Tree는 삭제로 인해 발생한 black-height 불균형을 해결해야 하므로, fixAfterDeletion(x) 같은 함수가 호출.

위 case 4에서는 x를 루트로 만들면서 루프를 종료 → 루프 종료 시점은 fix-up이 완료 → 결과 트리가 valid한 Red-Black Tree임이 보장된다.


### Deletion 정리
``` java
while (조건) {
    Case 1: sib이 RED
        → 색 바꾸고 rotate, sib 갱신
    Case 2: sib과 자식들이 BLACK
        → sib을 RED로, x를 부모로 올림
    Case 3/4: sib 자식 중 하나 RED
        → rotate, 색 바꾸고 종료
}
```

#### 시간 복잡도?

1. 삭제할 노드 찾기 — O(log n)
    이진 탐색 트리이므로, 값 v를 가진 노드를 찾는 데 필요한 시간은 트리의 높이에 비례함.
    - Red-Black Tree는 항상 균형 잡힌 이진 탐색 트리이므로, 높이는 O(log n).

2. 삭제 수행 — O(1) ~ O(log n)
    삭제 대상 노드가 자식이 하나뿐이거나 없는 경우, 단순 삭제로 끝나므로 시간은 작음.
    - 그러나 양쪽 자식이 모두 있는 경우, 후계자(predecessor 또는 successor)를 찾아서 값을 바꾸고 다시 삭제해야 함 → 이 과정도 O(log n)

3. Fix-up 과정 (recolor + rotate) — O(log n)
    삭제로 인해 Red-Black Tree의 속성이 깨질 수 있으므로, 재색칠 (recoloring) 과 회전 (rotation)



## 레드-블랙 트리의 최대 높이 (Worst-case Height)
모든 노드를 가능한 한 많이 빨간색으로 채우면 트리의 높이가 최대로 늘어남. 그래도 최악의 경우에도 높이는 2 log₂(n) 보다 작다.

즉, n개의 노드를 가질 때, 높이는 대략 2 log₂(n) 이내이다.

예시: n = 1,000,000 (백만 개 노드)라면, 레드-블랙 트리의 높이 ≈ 40


## 예시문제

### BinarySearchTree 클래스 ➔ leaves() 메서드를 추가하는 것.

```java
public class BinarySearchTree<E extends Comparable<E>> {
    protected static class Entry<E> {
        E element;
        Entry<E> left = null, right = null;
        
        Entry(E element) {
            this.element = element;
        }
    }

    protected Entry<E> root = null;

    public int leaves() {
        return leaves(root);
    }

    private int leaves(Entry<E> node) {
        if (node == null) {
            return 0;
        }
        if (node.left == null && node.right == null) {
            return 1;
        }
        return leaves(node.left) + leaves(node.right);
    }
}
```

#### Printing a BinarySearchTreeObject

```java
public String toString() {
    StringBuilder sb = new StringBuilder();
    toStringHelper(root, 0, sb);
    return sb.toString();
}

private void toStringHelper(Node<E> node, int level, StringBuilder sb) {
    if (node == null) {
        return;
    }
    toStringHelper(node.right, level + 1, sb);
    for (int i = 0; i < level; i++) {
        sb.append("    ");
    }
    sb.append(node.data.toString()); 
    sb.append("\n");
    toStringHelper(node.left, level + 1, sb);
}
```

### Recitation 다시 풀어보기

```java
class IntTree{
    int data;
    Arrraylist <IntTree> children;

}

public int findMin(IntTree t){
    if (t == null) {
        return Integer.MAX_VALUE;
    }

    int currentMin = t.data;

    if (t.children != null && !t.children.isEmpty()) {
        for (IntTree child : t.children) {
            int childMin = findMin(child);
            currentMin = Math.min(currentMin, childMin);
        }
    }
    return currentMin;
}

public boolean contains(int n, IntTree t){
    if (t == null) {
        return false;
    }
    if (t.data == n) {
        return true;
    }

    if (t.children != null && !t.children.isEmpty()) {
        for (IntTree child : t.children) {
            if (contains(n, child)) {
                return true;
            }
        }
    }
    return false;
}

public int findMin(IntTree t) {
    if (t == null) {
        return Integer.MAX_VALUE;
    }
    int currentMin = t.data;
    if (t.children != null && !t.children.isEmpty()) {
        int numChildren = t.children.size();
        for (int i = 0; i < numChildren; i++) {
            IntTree child = t.children.get(i);
            int childMin = findMin(child);
            currentMin = Math.min(currentMin, childMin);
        }
    }
    return currentMin;
}

public boolean contains(int n, IntTree t) {
    if (t == null) {
        return false;
    }
    if (t.data == n) {
        return true;
    }

    if (t.children != null && !t.children.isEmpty()) {
        int numChildren = t.children.size();
        for (int i = 0; i < numChildren; i++) {
            IntTree child = t.children.get(i);

            if (contains(n, child)) {
                return true;
            }
        }
    }
    return false;
}
```

### 왼쪽 회전

```java
public void leftRotate(Entry<E> x) {
    if (x == null || x.right == null) {
        return; 
    }

    Entry<E> y = x.right; 

    x.right = y.left;
    if (y.left != null) {
        y.left.parent = x;
    }

    y.parent = x.parent;
    if (x.parent == null) {
        root = y;
    } else if (x == x.parent.left) {
        x.parent.left = y;
    } else {
        x.parent.right = y;
    }

    y.left = x;
    x.parent = y;
}

public String toTreeString() {
    StringBuilder sb = new StringBuilder();
    toTreeStringHelper(root, sb, 0);
    return sb.toString();
}

private void toTreeStringHelper(Entry<E> node, StringBuilder sb, int depth) {
    if (node != null) {
        for (int i = 0; i < depth; i++) {
            sb.append("  ");
        }
        sb.append(node.element).append("\n");
        toTreeStringHelper(node.left, sb, depth + 1);
        toTreeStringHelper(node.right, sb, depth + 1);
    }
}
```
