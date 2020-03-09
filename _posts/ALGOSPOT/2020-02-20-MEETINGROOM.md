---
title: "Algospot :: MEETINGROOM (회의실 배정)"
date: 2020-02-20 19:38:00 -0400
categories: 알고리즘_문제풀이 
tags: Algospot DFS
---

# 회의실 배정 ( [MEETINGROOM](https://algospot.com/judge/problem/read/MEETINGROOM) , 상)

## 문제  


## 나의 풀이  

## SAT 문제
- 불린 값 만족성 문제(Boolean satisfiability problem) : 참이냐 거짓이냐의 결정을 여러 번 해야 하는 문제
  - SAT 문제 예 : ` a && (!b || !a) && (c && (!a || !b))`을 참으로 만드는 변수 a,b,c가 존재하는가? (답은 X)
  


## 나의 코드

<details>
<summary>MEETINGROOM - 나의 코드 (오답) </summary>
<div markdown="1">

```


```
</div>
</details>  


## 책의 풀이  


## 책의 코드

<details>
<summary>MEETINGROOM - 책의 코드</summary>
<div markdown="1">

```


```
</div>
</details>  



## 기억할 점
- `vector<vector<int>> graph` 대신 `vector<int> graph[]` 꼴이 편하다.
- `dfs()` 내부에서 감시 카메라를 설치할 노드까지 선택하는 아이디어
- `dfs()` 내부에서 `int children[3]` 사용한 아이디어
- `dfs()`에서 `UNCENSORED`로 리턴된 것은 `dfsAll()`에서 처리하는 아이디어

