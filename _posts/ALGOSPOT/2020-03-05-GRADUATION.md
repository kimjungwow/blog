---
title: "Algospot :: GRADUATION (졸업 학기)"
date: 2020-03-05 18:26:00 +0900
categories: 알고리즘_문제풀이 
tags: Algospot Bitmask
---

# 졸업 학기 ( [GRADUATION](https://algospot.com/judge/problem/read/GRADUATION) , 중)

## 문제  
태우는 졸업하기 위해서 K과목을 들어야 한다. 앞으로 M학기 동안 어떤 과목들이 열리고, 각 과목의 선수 과목은 무엇이며 한 학기당 L개 이하의 과목을 들을 수 있을 때 태우는 최소 몇 학기를 다녀야 졸업할 수 있는지 출력해야 한다. 수업을 하나도 듣지 않는 학기는 휴학한 것으로 생각해 학기 수에 포함하지 않으며, 졸업할 방법이 없으면 IMPOSSIBLE을 출력해야 한다.

## 나의 풀이  
`uint32_t` 타입을 이용하여 0~N-1번째 과목을 각각 들었는지 안 들었는지를 비트로 표시했다. gcc/g++의 `__builtin_popcount()`를 사용하기 위해 `uint32_t` 타입을 이용했다.  

특정 수업을 들었는지를 확인하는 `already()`, 특정 수업의 선수 과목을 모두 들었는지를 확인하는 `allPre()`를 만들었다.  

이후 재귀함수 `getSems(int sems, int curr, uint32_t status)`를 만들었다. 이 함수는 `sems`학기를 이미 다녔고 수강한 수업들은 `status`로 표시되었을 때 `curr`번째 학기를 다닐 지 고민하는 중일 때 졸업에 필요한 최소 학기를 출력한다.  
우선 기저 사례로는 `status`에 1인 비트가 K개 이상이면 `sems`를 리턴하고, `curr`가 m이상이면 (즉 모든 학기를 다닐지 안 다닐지 따져본 상태) m+1을 리턴했다. 모든 경우를 따졌을 때 최소 학기가 m+1학기이면 불가능한 경우이다.  
그리고 `curr`번째 학기를 다니지 않을 때와 다닐 때로 생각했다. 주목할 점은 특정 학기에는 들을 수 있는 수업이 l개 이상 있을 수 있어서, 그 경우에는 그 학기의 수업들의 subset 중 크기가 l인 경우만 따져주었다.

## 나의 코드

<details>
<summary>GRADUATION - 나의 코드 (45m) </summary>
<div markdown="1">

```

#include <bits/stdc++.h>
using namespace std;
int n = 0, k, m, l, taken;
uint32_t pre[13];
vector<int> classes[11];

bool already(int status, int classNum)
{
    return status & (1 << classNum);
}

bool allPre(int status, int classNum)
{
    return (status & pre[classNum]) == pre[classNum];
}

int getSems(int sems, int curr, uint32_t status)
{
    if (__builtin_popcount((uint32_t)status) >= k)
        return sems;
    else if (curr == m)
        return m + 1;

    int ret = getSems(sems, curr + 1, status);
    uint32_t thisSem = 0;
    for (auto x : classes[curr])
    {
        if (already(status, x) == false && allPre(status, x))
        {
            thisSem |= ((uint32_t)1 << x);
        }
    }

    if (thisSem)
    {
        if (__builtin_popcount((uint32_t)thisSem) >= l)
        {
            for (uint32_t subset = thisSem; subset; subset = ((subset - 1) & thisSem))
            {
                if (__builtin_popcount(subset) == l)
                    ret = min(ret, getSems(sems + 1, curr + 1, status | subset));
            }
        }
        else
        {
            ret = min(ret, getSems(sems + 1, curr + 1, status | thisSem));
        }
    }

    return ret;
}

int main()
{
    int tc;
    cin >> tc;
    while (tc--)
    {
        cin >> n >> k >> m >> l;
        taken = 0;
        for (int i = 0; i < n; i++)
        {
            int temp;
            uint32_t mask = 0;
            cin >> temp;
            while (temp--)
            {
                uint32_t pos;
                cin >> pos;
                mask |= (1 << pos);
            }
            pre[i] = mask;
        }
        for (int i = 0; i < m; i++)
        {
            classes[i].clear();
            int temp;
            cin >> temp;
            while (temp--)
            {
                int c;
                cin >> c;
                classes[i].push_back(c);
            }
        }
        int ret = getSems(0, 0, (uint32_t)0);
        if (ret == m + 1)
            cout << "IMPOSSIBLE" << endl;
        else
            cout << ret << endl;
    }
}

```
</div>
</details>  


## 책의 풀이  
책에서는 이 문제에 메모이제이션을 적용하여 더욱 DP스럽게 풀었다. 나는 따져보는 학기 `curr`가 `m`이 된 경우를 기저 사례로 생각하여 지금까지 수강한 학기를 나타내는 `sems`를 리턴했다. 이를 위해 `getSems()`의 파라미터가 3개가 필요했다. 하지만, `getSems(int curr, int status)`를 지금까지 들은 수업들을 `status`로 나타내고, 현재 `curr`번째 학기를 고민 중일 때, 앞으로 최소 몇 학기를 더 들어야 하는지로 정하면 파라미터의 수를 하나 줄일 수 있고, 메모이제이션도 적용할 수 있다.  

또한 나는 학기별로 열리는 수업을 `vector<int>`의 꼴로 저장했는데, 이 역시 비트마스크를 이용해 하나의 `uint32_t`에 저장하면 더욱 효율적이다.  

그리고 이번 학기에 들을 수 있는 수업의 수와 상관 없이, 크기가 l이하인 모든 `subset`에 대하여 재귀적으로 실행해주면 된다. 나는 크기가 정확히 l인 `subset`에 대하여 실행했지만, 이는 엄밀한 근거 없이 나의 추측에 근거한 코딩이었다.


## 책의 코드

<details>
<summary>GRADUATION - 책의 코드 </summary>
<div markdown="1">

```

#include <bits/stdc++.h>
using namespace std;
int n = 0, k, m, l, taken;
uint32_t pre[13];
uint32_t classes[11];
int cache[10][1<<12];

bool already(int status, int classNum)
{
    return status & (1 << classNum);
}

bool allPre(int status, int classNum)
{
    return (status & pre[classNum]) == pre[classNum];
}

int getSems(int curr, uint32_t status)
{
    if (__builtin_popcount((uint32_t)status) >= k)
        return 0;
    else if (curr == m)
        return m + 1;
    
    int& ret=cache[curr][status];
    if(ret!=-1) return ret;

    ret = getSems(curr + 1, status);
    uint32_t thisSem = classes[curr]&(~status);
    for (int i=0;i<n;i++) {
        if(!allPre(status,i)) thisSem &= ~(1<<i);
    }

    if (thisSem)
    {
        for (uint32_t subset = thisSem; subset; subset = ((subset - 1) & thisSem))
        {
            if (__builtin_popcount(subset) <= l)
                ret = min(ret, 1+getSems(curr + 1, status | subset));
        }
    }

    return ret;
}

int main()
{
    int tc;
    cin >> tc;
    while (tc--)
    {
        memset(cache, -1, sizeof cache);
        cin >> n >> k >> m >> l;
        taken = 0;
        for (int i = 0; i < n; i++)
        {
            int temp;
            uint32_t mask = 0;
            cin >> temp;
            while (temp--)
            {
                uint32_t pos;
                cin >> pos;
                mask |= (1 << pos);
            }
            pre[i] = mask;
        }
        for (int i = 0; i < m; i++)
        {
            int temp;
            cin >> temp;
            int thisClasses=0;
            while (temp--)
            {
                int c;
                cin >> c;
                thisClasses|=(1<<c);
            }
            classes[i]=thisClasses;
        }
        int ret = getSems(0, (uint32_t)0);
        if (ret == m + 1)
            cout << "IMPOSSIBLE" << endl;
        else
            cout << ret << endl;
    }
}

```
</div>
</details>  


## 기억할 점
- 이전에 공부한 DP, 메모이제이션 활용하기
- 경우의 수가 적은 경우는 `vector<int>` 대신 `uint32_t` 사용하기