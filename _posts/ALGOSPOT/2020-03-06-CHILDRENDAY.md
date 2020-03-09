---
title: "Algospot :: CHILDRENDAY (어린이날)"
date: 2020-03-06 15:30:00 +0900
categories: 알고리즘_문제풀이 
tags: Algospot BFS
---

# 어린이날 ( [CHILDRENDAY](https://algospot.com/judge/problem/read/CHILDRENDAY) , 상)

## 문제  
N명의 아이들에게 장난감을 나눠주는데, 그 중 M(`0<=M<N`)명의 욕심쟁이 어린이들은 남들보다 장난감을 한 개씩 더 줘야 만족한다. 종교적인 이유로 인해, 아이들에게 나눠줄 장난감의 총 수 C는 십진수로 썼을 때 특정 숫자만으로 구성되어야 한다. 이 때 가장 작은 C를 구해야 한다.

## 나의 풀이  
우선 문제를 잘못 이해했다. `3,4,5`만 사용할 수 있을 때, C가 `334`와 같은 수도 될 수 있지만 잘못 생각했다.  

가능한 C의 후보들(`N*X+M, X>0`)을 만들어보며, 특정 숫자들로만 이루어진 수로 확인했다. 수의 범위가 너무 커져서 메모리 초과가 떴다. `int64_t`로도 모자라서 더 큰 자료형을 써야 하나 고민했지만, 잠시 책을 보니 나머지 연산을 이용했다. 

그래서 나머지 연산과 BFS처럼 큐를 이용하여 풀려 했지만, 가능한 C의 후보들을 정점으로 생각해서 못 풀었다. C의 후보들은 정점으로 생각하면, 더 큰 C의 후보들을 구할 때 외에는 기존의 정점들이 쓰이지 않는다. 나머지는 책의 풀이를 보고 이해했다. 


## 책의 풀이  
이 문제는 다음의 조건을 만족하는 최소의 자연수 c를 찾는 것이다.
1. n+m 이상이어야 한다.
1. n으로 나눈 나머지가 m이어야 한다.
1. d에 포함된 숫자로만 구성되어야 한다.
내가 생각한 것 처럼, `m+an(a>=1)`인 모든 수가 1,2번 조건을 만족한다. 여기서 a를 1씩 늘려가며 3번 조건에 만족하는지 확인하는 것은 오래 걸린다. 따라서 내가 한 것처럼 3번 조건을 만족하는 수들 중에서 1,2번 조건을 만족하는 최소의 자연수를 찾아야 한다.  

문제는 c의 값이 아주 커질 수 있다는 것이다. 여기서 `((c%n)X10+x)%n == (c*10+x)%n`임을 이용하면, 중요한 것은 현재 숫자가 아니라 c를 n으로 나눈 나머지가 된다. 이를 통해 c의 값이 아주 커지는 것을 막을 수 있다. 

내 풀이와 가장 다른 점은, 책에서는 BFS를 사용할 때 c를 n으로 나눈 나머지를 정점으로 두었다는 점이다. 정점 y는 현재까지 만든 c를 n으로 나눈 나머지가 y임을 뜻한다. d의 숫자중 하나인 b에 대하여, `(yX10+b)%n==x`라면, y에서 x로 향하는 간선을 추가할 수 있다.  

이 때, 정점 0에서 정점 m으로 향하는 최단 경로를 구한다고 끝나지 않는다. 최단 경로의 길이는 c의 자릿수를 의미하지만, 자릿수가 같으며 조건을 만족하는 c가 여러 개 있을 수 있기 때문이다. 따라서 정점에서 간선들을 탐색할 때, 간선들의 번호(새롭게 더해주는 수)가 가장 작은 최단 경로를 찾아야 한다.  

`H_i`는 정점 0(시작점)으로부터 거리가 i인 정점이라고 할 때, 같은 `H_i`에 속한 정점들로 향하는 가장 작은 c는 자릿수가 같다. 따라서 각 정점에서 인접한 간선을 검사할 때, 번호가 증가하는 순서로 검사하면 같은 `H_i`에서도 c가 작은 정점부터 방문하게 된다. **문제에서 사용 가능한 수들이 주어질 때 정렬되어 주어지지 않는다. 테스트케이스는 정렬된 경우만 보였지만, 정렬되어 주어진다는 조건이 없으므로 직접 정렬해야 한다.** 이 때, `string` 타입에도 `sort()`와 `reverse()`를 사용할 수 있다.

또한 1번 조건을 고려해야 한다. 예를 들어, m=0인 경우 시작점이 정점 0이지만, 다른 정점을 들렀다가 다시 0으로 돌아오는 경로를 찾아야 한다. 이에 따라, 0~n-1마다 정점을 만들되 n보다 작은 경우와 n이상인 경우로 나누어서 총 2n개의 정점을 만들면 된다. 시작점은 n보다 작은 경우의 정점 0이고, 끝점은 n이상인 경우의 정점 k이다.

그래프는 2n개의 정점과 각 정점마다 d개의 간선으로 이루어지므로, 시간복잡도는 `O(2nX|d|)`이다.

## 책의 코드

<details>
<summary>CHILDRENDAY - 책의 코드 </summary>
<div markdown="1">

```
#include <bits/stdc++.h>
using namespace std;

int n, m;
queue<int> q;
vector<int> edges[10001];
int parent[20001], keep[20001];
string avail;
int main()
{
    ios::sync_with_stdio(false);
    cin.tie(NULL);
    int tc;
    cin >> tc;
    while (tc--)
    {
        cin >> avail;
        cin >> n >> m;
        memset(parent, -1, sizeof(int) * 2 * n);
        memset(keep, -1, sizeof(int) * 2 * n);
        
        sort(avail.begin(),avail.end());

        while (!q.empty())
            q.pop();
        q.push(0);
        parent[0] = 0;
        while (!q.empty())
        {
            int a = q.front();
            q.pop();
            for (int j = 0; j < avail.length(); j++)
            {
                int x = avail[j] - '0';
                if (10 * a + x < n)
                {
                    if (parent[10 * a + x] != -1)
                        continue;
                    q.push(10 * a + x);
                    parent[10 * a + x] = a;
                    keep[10 * a + x] = x;
                }
                else
                {
                    if (parent[n + (10 * a + x) % n] != -1)
                        continue;
                    q.push(n + (10 * a + x) % n);
                    parent[n + (10 * a + x) % n] = a;
                    keep[n + (10 * a + x) % n] = x;
                }
            }
        }
        list<int> route;
        int here = m + n;
        while (here != parent[here] && parent[here] != -1)
        {
            route.push_back(keep[here]);
            here = parent[here];
        }
        reverse(route.begin(), route.end());
        if (route.size() > 0)
            for (auto z : route)
                cout << z;
        else
            cout << "IMPOSSIBLE";
        cout << "\n";
    }

    return 0;
}

```
</div>
</details>  

 
## 기억할 점
- c가 아주 커지면 `int64_t`에도 담기지 않으므로, `((c%n)X10+x)%n == (c*10+x)%n`의 아이디어를 이용해야 함
- n으로 나눈 나머지를 정점으로 사용하는 아이디어
  - 이 때, n미만인 정점과 n이상인 정점을 분리
- 문제의 조건에는 d가 정렬되어 주어진다는 주어지지 않지만, 예제에서는 d가 정렬되어 주어지는 것처럼 보여도 직접 정렬해야 함