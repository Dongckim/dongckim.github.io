---
title: "99클럽 JAVA 코딩테스트 예시답안 9일차 [이분 그래프]"
layout: single
Typora-root-url: ../
categories: 99CLUB
tag: java
use_math: true
---

이분 그래프

## 문제 안에서 특이사항 확인하기

이분 그래프의 정의부터 확실하게 이해하는게 좋겠죠?

> 그래프의 정점의 집합을 둘로 분할하여, 각 집합에 속한 정점끼리는 서로 인접하지 않도록 분할할 수 있을 때, 그러한 그래프를 특별히 이분 그래프 (Bipartite Graph) 라 부른다

이게 무슨 소리일까.

`서로 인접하지 않도록 분할할 수 있을 때` ? 이게 무슨 소리일까?

보통 `정점`이라고 하면, 다들 최상단에 위치한 부모 노드를 생각하곤 한다.

![]({{site.url}}/images/2025-01-23-java-daynine/class.jpg)

*요식업계의 정점? 뻘소리이다.*

정점은 연결의 대상이 되는 개체 또는 위치를 의미한다. 즉, 노드와 같은 의미를 갖는다. 꼭 최상단일 필요가 없다는 것이다.

문제에서 주어진 말을 다시 되짚어보면, 두 개의 집합을 주는데, 각 집합끼리는 인접하지만 않으면 될 것 같다는 느낌만 받고 가면 아주 Best하다.

## 제한사항 꼭 확인하기

- 2 ≤ K ≤ 5
- 1 ≤ V ≤ 20,000
- 1 ≤ E ≤ 200,000

그래프이기 때문에, 대놓고 BFS,DFS를 티내고 있기 때문에, 갯수에 대한 부담감은 없겠다.

그럼 bfs와 dfs중 선택해야겠다, 뭐가 맞을까?

사실 따지고 보면 두 방법 모두로 풀 수 있다. 다만 어떤게 더 효율적이고 올바른 방법인지는 생각해볼 필요가 있어보인다.
200,000이 개수가 나는 살짝 걸리긴 한다. DFS 재귀로 풀었을 때 worst의 경우 저 숫자만큼 재귀함수를 실행할 수 있을까? 

그래서 BFS로 선택하게 되었다. 

## 입출력 비교하기

![]({{site.url}}/images/2025-01-23-java-daynine/io.png)

입출력을 잘 확인했어야 했다.

2개가 주어진다는 것은, 두 줄이 주어진다는 뜻이 아니라,
두 개의 테스트 케이스이고, 두 개의 테스트 케이스는 뭉쳐 있기 때문에 분리해서 확인할 줄 알아야 한다.

1. `V = 3`과 `E = 2` 인 케이스를 먼저 보면, 
- 1−3: 정점 1과 정점 3이 연결됨.
- 2−3: 정점 2와 정점 3이 연결됨.
```
1: [3]
2: [3]
3: [1, 2]
```
- 이 때 방향은 전혀 중요하지 않다.

2. 바로 이어서 `V = 4`과 `E = 4` 인 케이스가 이어져서 나온다.
- 1−2: 정점 1과 정점 2가 연결됨.
- 2−3: 정점 2와 정점 3이 연결됨.
- 3−4: 정점 3과 정점 4가 연결됨.
- 4−2: 정점 4와 정점 2가 연결됨.
```
1: [2]
2: [1, 3, 4]
3: [2, 4]
4: [3, 2]
```

이때, 정점 3으로 이동한 경우를 따져보면,
정점 3은 이미 색 1로 칠해져있음ㅇ에도, 정점 3의 인접 정점 2는 색 -1로 칠해져 있기 때문에 문제가 없지만, 정점 3의 인접 정점 4는 이미 색 1로 칠해져 있음 → 문제 발생 (같은 색).


## Psudo Code

```java
import java.io.*;
import java.util.*;

public class Main {
	static int v, e;
	static ArrayList<Integer>[] al;
	static int visit[];

	public static void main(String[] args) throws Exception {
		BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
		StringTokenizer stz = new StringTokenizer(br.readLine());
		int t = Integer.parseInt(stz.nextToken());

		for(int tc = 0; tc < t; tc++) {
			stz = new StringTokenizer(br.readLine());
			v = Integer.parseInt(stz.nextToken());
			e = Integer.parseInt(stz.nextToken());
			visit = new int[v+1];
			al = new ArrayList[v+1];

			for(int i = 0; i <= v; i++)
				al[i] = new ArrayList<Integer>();

			for(int i = 0; i < e; i++) {
				stz = new StringTokenizer(br.readLine());
				int p1 = Integer.parseInt(stz.nextToken());
				int p2 = Integer.parseInt(stz.nextToken());

				al[p1].add(p2);
				al[p2].add(p1);
			}
			grouping();
		}
	}

	public static void grouping() {
		Queue<Integer> q = new LinkedList<Integer>();

		for(int i = 1; i <= v; i++) {
			if(visit[i] == 0) {
				q.add(i);
				visit[i] = 1;
			}

			while(!q.isEmpty()) {
				int now = q.poll();

				for(int j = 0; j < al[now].size(); j++) {
					if(visit[al[now].get(j)] == 0) {
						q.add(al[now].get(j));
					}
					
					if(visit[al[now].get(j)] == visit[now]) {
						System.out.println("NO");
						return;
					}

					if(visit[now] == 1 && visit[al[now].get(j)] == 0)
						visit[al[now].get(j)] = 2;
					else if(visit[now] == 2 && visit[al[now].get(j)] == 0)
						visit[al[now].get(j)] = 1;
				}
			}
		}

		System.out.println("YES");
	}

}
```