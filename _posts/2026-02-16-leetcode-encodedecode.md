---
title: "Python Coding Test - Encode and Decode Strings (Chunked Encoding)"
layout: single
Typora-root-url: ../
categories: Algorithm
tag: [python, string, twopointer, design]
use_math: true
---

## Encode and Decode Strings (문자열 인코딩과 디코딩)

문자열 리스트를 하나의 문자열로 인코딩하고, 네트워크를 통해 전송된 후 다시 원래의 리스트로 디코딩하는 알고리즘을 설계하는 문제이다.

이 글에서는 특수 구분자를 포함한 모든 문자를 처리할 수 있는 **Chunked Encoding**과 **Two Pointer** 접근법을 정리해본다.

---

## 문제의 핵심: 구분자 처리

단순하게 쉼표(`,`)나 공백 같은 구분자를 사용하는 방법을 떠올릴 수 있다. 하지만 입력 문자열 자체에 해당 문자가 포함되어 있다면 디코딩 과정이 깨진다. "데이터"와 "메타데이터"를 구분할 수 있는 방법이 필요하다.

---

## 핵심 로직: Chunked Encoding

가장 견고한 방법은 각 문자열 앞에 **문자열의 길이**와 **구분자**를 붙이는 것이다.

### 구현

```python
class Solution:
    def encode(self, strs: List[str]) -> str:
        # 형식: {길이}#{단어}
        res = ""
        for s in strs:
            res += str(len(s)) + "#" + s
        return res

    def decode(self, s: str) -> List[str]:
        res = []
        p1 = 0
        
        while p1 < len(s):
            p2 = p1
            # 탐색 포인터: 구분자를 찾는다
            while s[p2] != "#":
                p2 += 1
            
            # 길이와 단어를 추출
            word_len = int(s[p1:p2])
            res.append(s[p2 + 1 : p2 + 1 + word_len])
            
            # p1을 다음 청크의 시작점으로 점프
            p1 = p2 + 1 + word_len
            
        return res
```

---

## 전략: Two Pointer 진입 방식

`while` 루프와 투 포인터 전략을 결합하면 인코딩된 문자열을 가장 안정적으로 탐색할 수 있다.

- **p1 (진입 포인터)**: 메타데이터(단어 길이)의 시작 위치를 가리킨다.
- **p2 (탐색 포인터)**: `p1`에서 출발하여 `#` 구분자를 찾는다.
- **"점프"**: 길이를 파싱한 뒤, 정확한 문자 수만큼 슬라이싱하고 `p1`을 다음 세그먼트로 이동시킨다.

이 "동적 윈도우" 방식은 문자열 내부에 `#`이나 숫자가 포함되어 있더라도 혼동이 발생하지 않는다. 메타데이터가 위치할 특정 인덱스에서만 구분자를 찾기 때문이다.

---

## 최적화 및 고려 사항

### 1. 문자열 결합 성능

파이썬에서 문자열은 불변(immutable)이다. `res += ...`를 반복하면 매번 새로운 문자열 객체가 생성되어, 최악의 경우 인코딩에서 $O(N^2)$의 시간이 소요된다.

**개선**: `"".join()`을 사용하면 $O(N)$으로 효율적으로 처리할 수 있다.

```python
def encode(self, strs: List[str]) -> str:
    return "".join(f"{len(s)}#{s}" for s in strs)
```

### 2. 시간 및 공간 복잡도

- **시간 복잡도**: 인코딩과 디코딩 모두 $O(N)$, 여기서 $N$은 모든 문자열의 총 문자 수이다.
- **공간 복잡도**: 출력 리스트/문자열에 사용되는 메모리를 제외하면 $O(1)$이다.

---

## 요약 및 회고

### 왜 Two Pointer인가?

- **견고함**: 빈 문자열, 숫자로만 이루어진 문자열, 구분자 자체를 포함하는 문자열 등 엣지 케이스를 모두 처리할 수 있다.
- **효율성**: 길이 메타데이터를 활용하여 문자열의 "본문"을 건너뛰므로, 불필요한 문자 검사를 피할 수 있다.

### 결론

Chunked Encoding 방식과 Two Pointer 디코딩 전략의 조합은 직렬화된 데이터를 처리하는 정석적인 접근법이다. 단순하면서도 예상치 못한 입력 데이터에도 흔들리지 않는 프로토콜을 설계하는 방법을 배울 수 있는 유익한 문제였다.
