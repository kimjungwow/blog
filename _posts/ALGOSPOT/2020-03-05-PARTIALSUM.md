---
title: "Algospot :: PartialSum"
date: 2020-03-05 23:33:00 +0900
categories: 알고리즘_공부
tags: Algospot PartialSum
---

# PartialSum (부분 합) : AOJ 17단원  
## 부분 합이란
부분 합(혹은 누적 합)이란 배열의 각 위치에 대해 배열의 시작부터 현재 위치까지의 원소의 합을 구해 둔 배열이다. 다양한 `a, b`에 대하여 a등에서 b등까지의 평균 점수를 계산할 일이 많다면, 부분 합을 미리 구해두면 좋다.  

`psum[i] = SUM_0_to_i(scores[j])`일 때, `psum[-1]=0`이라 하면 `scores[a]`부터 `scores[b]`까지의 합은 `psum[b]-pusm[a-1]`으로 구할 수 있다.  

### 부분 합 계산하기
`STL`에도 `partial_sum()`이 있다. 직접 구현하려면 for 문을 이용해 배열을 순회하며 구하면 되는데, 반복문을 통해 이후에 구간 합을 구하려면 최대 `O(N)`이 걸리므로 구간 합을 두 번 이상 구해야 한다면 부분 합을 구해두는 것이 이득이다.

### 부분 합으로 분산 계산하기
배열 `A[]`의 구간 `A[a...b]`의 분산은
![1](https://latex.codecogs.com/gif.latex?v=\frac{1}{b-a&plus;1}\cdot&space;\sum_{i=a}^{b}(A[i]-m_{a,b})^{2})
로 정의된다. 이 때 `m_a,b`는 해당 구간의 평균이다.  

위 식을 정리하면
![2](https://latex.codecogs.com/gif.latex?v=\frac{1}{b-a&plus;1}\cdot&space;\sum_{i=a}^{b}(A[i]-m_{a,b})^{2}=\frac{1}{b-a&plus;1}\cdot&space;\sum_{i=a}^{b}(A[i]^{2}-2A[i]\cdot&space;m_{a,b}&plus;m_{a,b}^{2})=\frac{1}{b-a&plus;1}\cdot&space;(&space;\sum_{i=a}^{b}A[i]^{2}-2m_{a,b}\cdot&space;\sum_{i=a}^{b}A[i]&plus;(b-a&plus;1)m_{a,b}^{2}&space;))
  
인데, 마지막 식의 세 항 중, 가운데 항과 오른쪽 항은 `psum`을 이용해 쉽게 계산할 수 있다. `A[i]^2`의 합이 문제인데, 이것 또한 `A[]`의 각 원소의 제곱의 부분 합을 미리 저장해두면 `O(1)`에 계산할 수 있다.

### 2차원으로의 확장

![3](https://latex.codecogs.com/gif.latex?psum[y,x]=\sum_{i=0}^{y}\sum_{j=0}^{x}A[i,j]) 로 하면, `psum[y,x]`는 `(0,0)`을 왼쪽 위 칸, `(y,x)`를 오른쪽 아래 칸으로 갖는 직사각형 구간에 포함된 원소들의 합이다. 이 때, x의 범위는 x1~x2이고, y의 범위는 y1~y2인 부분의 합 `sum(y1,x1,y2,x2)`는 `psum[y2,x2]-psum[y2,x1-1]-psum[y1-1,x2]+psum[y1-1,x1-1]`으로 구할 수 있다.

### 예제 : 합이 0에 가장 가까운 구간

양수와 음수가 모두 포함된 배열 A[]가 있을 때, 그 합이 0에 가장 가까운 구간을 찾아야 한다. 이 때 배열의 모든 구간을 순회하면서 확인하면 `O(N^2)`이다. 하지만 `SUM_k=i_to_j(A[k]) = psum[j]-psum[i-1]`임을 이용하면, 이 값이 0에 가깝다는 것은 psum[]의 두 값의 차이가 가장 적다는 것이다. 부분 합을 정렬한 뒤 인접한 우너소들을 확인하여 가장 작은 차가 곧 가장 0에 가까운 구간 합이다. 정렬은 `O(NlgN)`이고, 부분 합을 구하는 것과 인접한 원소를 확인하는 것 모두 `O(N)`이므로 알고리즘의 수행 시간은 `O(NlgN)`이다.  

