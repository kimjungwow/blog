---
title: "BAEKJOON :: Icy Perimeter (17025)"
date: 2020-03-28 20:01:00 -0400
categories: 알고리즘_문제풀이 
tags: BAEKJOON BFS
---

# Icy Perimeter ( [17025](https://www.acmicpc.net/problem/17025) )

## 문제

NxN grid에 `#`과 `.`으로 해당 칸에 아이스크림이 있는지 없는지 나타내진다. 상하좌우로 연결된 아이스크림들이 하나의 blob을 이룬다. 이 때, blob은 내부에 hole이 있을 수 있다. grid에 존재하는 모든 blob 중 가장 넓은 blob의 넓이와 그 blob의 둘레 길이를 출력해야 한다. 가장 넓은 blob이 여러 개면 가장 짧은 둘레 길이를 출력해야 한다.

## 나의 풀이

DFS나 BFS를 이용하면 각 blob에 속하는 칸의 개수 = 각 blob의 넓이는 쉽게 구할 수 있다. 둘레를 어떻게 구해야 할 지 고민하다가, 각 blob마다 존재하는 가로 칸의 개수와 세로 칸의 개수를 각각 세기로 했다. 가로 칸의 개수를 구할 때는 해당 blob의 점들을 column별로 구분해서 확인한다. 해당 blob에 속하는 점들 중 하나의 column 안에서 몇 개의 blob으로 나누어지는지 확인한다. 그 개수가 z이면, 각 column마다 `2X(z+1)`을 더해준다. 세로 칸은 column 대신 row를 이용한다.

## 나의 코드

<details>
<summary>Icy Perimeter - 나의 코드 (소요시간 : 41m)</summary>
<div markdown="1">

  
```
#include <bits/stdc++.h>
using namespace std;
int graph[1000][1000];
int n, comp = 1, maxArea = 0, minPerimeter = 0;
priority_queue<pair<int, int>> rs, cs;
int dx[4] = {0, 1, 0, -1}, dy[4] = {1, 0, -1, 0};
int dp(int i, int j)
{
    if (i < 0 || j < 0 || i >= n || j >= n)
        return 0;
    if (graph[i][j] != -1)
        return 0;
    rs.push(make_pair(i, j));
    cs.push(make_pair(j, i));
    int ret = 1;
    graph[i][j] = comp;
    for (int x = 0; x < 4; x++)
    {
        ret += dp(i + dx[x], j + dy[x]);
    }
    return ret;
}

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(NULL);

    string s;
    memset(graph, 0, sizeof graph);
    cin >> n;
    for (int i = 0; i < n; i++)
    {
        cin >> s;
        for (int j = 0; j < n; j++)
        {
            if (s[j] == '#')
                graph[i][j] = -1;
        }
    }
    for (int i = 0; i < n; i++)
    {
        for (int j = 0; j < n; j++)
        {
            if (graph[i][j] != -1)
                continue;
            int v = dp(i, j), prevR = n, prevC = n, nums = 0, wls = 0;
            if (v > 0)
                comp++;
            if (maxArea>v) {
                while(!rs.empty()) rs.pop();
                while(!cs.empty()) cs.pop();
                continue;
            }
            while (!rs.empty())
            {
                pair<int, int> tp = rs.top();
                rs.pop();
                if (tp.first < prevR)
                {
                    wls += 2 * nums;
                    prevC = tp.second;
                    prevR = tp.first;
                    nums = 1;
                }
                else
                {
                    if (prevC != tp.second + 1)
                    {
                        nums++;
                    }
                    prevC = tp.second;
                }
            }
            wls += 2 * nums;
            prevR = n;
            prevC = n;
            nums = 0;
            
            while (!cs.empty())
            {
                pair<int, int> tp = cs.top();
                cs.pop();
                if (tp.first < prevC)
                {
                    wls += 2 * nums;
                    prevC = tp.first;
                    prevR = tp.second;
                    nums = 1;
                }
                else
                {
                    if (prevR != tp.second + 1)
                    {
                        nums++;
                    }
                    prevR = tp.second;
                }
            }
            wls += 2 * nums;
            if (maxArea < v)
            {
                maxArea = v;
                minPerimeter=wls;
            }
            else 
            {
                minPerimeter=min(minPerimeter,wls);
            }
            
            
        }
    }
    cout<<maxArea<<" "<<minPerimeter;
}
```
</div>
</details>


## 좋은 풀이

**둘레를 간단히 구하는 방법은 각 칸이 인접한 빈 칸(`.`으로 표시된 칸 혹은 전체 grid의 가장자리)의 개수를 세는 것이다.**