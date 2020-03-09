---
title: "Algospot :: MATCHORDER (출전 순서 정하기)"
date: 2020-03-09 21:08:00 +0900
categories: 알고리즘_문제풀이 
tags: Algospot Greedy_Method
---

# 출전 순서 정하기 ( [MATCHORDER](https://algospot.com/judge/problem/read/MATCHORDER) , 중)

## 문제  
러시아팀과 한국팀이 코딩 게임을 한다. 각 선수의 능력치가 주어지며, n명이 차례로 나와 1대1로 승부를 겨룬다. 능력치가 높은 선수가 무조건 이기며, 더 많이 이긴 팀이 승리한다. 러시아 선수들의 출전 순서가 주어질 때, 한국팀은 최대 몇 승을 할 수 있을까?

## 나의 풀이
더 지니어스2의 [흑과 백](https://namu.wiki/w/%EB%8D%94%20%EC%A7%80%EB%8B%88%EC%96%B4%EC%8A%A4:%EB%A3%B0%20%EB%B8%8C%EB%A0%88%EC%9D%B4%EC%BB%A4/11%ED%99%94#s-2)이라는 게임과 비슷하다고 생각했다. 게임을 이기기 위해서는, 질때는 최대한 큰 차이로 지고 이길 때는 근소한 차이로 이겨야 한다. 먼저 한국 팀의 선수들을 능력치 오름차순으로 정렬한다. 뒤에서부터 확인하며, 상대를 가장 근소한 차이로 이길 수 있는 선수를 내보낸다. 이길 수 없는 선수가 없는 경우, 혹은 질 선수가 없는 경우에는 남은 선수 중 가장 능력치가 낮은 선수를 내보낸다.



## 나의 코드

<details>
<summary>MATCHORDER - 나의 코드 (11m)  </summary>
<div markdown="1">

```
#include <bits/stdc++.h>
using namespace std;
int n;
vector<int> russia, korea;
int main()
{
    int tc;
    cin >> tc;
    while (tc--)
    {
        cin >> n;
        russia.clear();
        korea.clear();
        for (int i = 0; i < n; i++)
        {
            int temp;
            cin >> temp;
            russia.push_back(temp);
        }
        for (int i = 0; i < n; i++)
        {
            int temp;
            cin >> temp;
            korea.push_back(temp);
        }
        sort(korea.begin(), korea.end());
        int wins = 0;
        for (int i = 0; i < n; i++)
        {
            for (int j = korea.size() - 1; j >= 0; j--)
            {
                if (j == korea.size() - 1 && korea[j] < russia[i])
                {
                    korea.erase(korea.begin());
                    break;
                }
                else if (korea[j] < russia[i])
                {
                    korea.erase(korea.begin() + j + 1);
                    wins++;
                    break;
                }
                else if (j == 0)
                {
                    korea.erase(korea.begin());
                    wins++;
                    break;
                }
            }
        }
        cout << wins << endl;
    }
}
```
</div>
</details>  

## 책의 풀이  
- greedy choice property 증명    
  - 현재 상대편 선수에게 가장 근소한 차이로 이기는 선수를 내거나, 이길 수 없는 경우 제일 능력치가 낮은 선수를 내보내는 최적해가 존재한다. 현재 상대편 선수에게 나의 계획과 다른 선수를 낸 최적해가 있다고 가정하고, 승패에 상관 없이 내 전략대로 내는 최적해가 있음을 보이자.
    - 무조건 지는 경우 : 현재 상대편 선수 X에게 더 작은 차이로 지는 선수 B를 내는 최적해가 존재한다고 하자. 그 상대편 선수 X의 상대로 더 작은 차이로 지는 선수 B와 가장 큰 차이로 지는 선수 A를 바꾼다고 생각하자. 그 상대편 선수 X를 상대로는 여전히 진다. 기존에 A를 상대한 상대편 선수 Y는 더 높은 B와 상대하게 되므로, 승패가 줄어들 일은 없다.
    - 더 큰 차이로 이기는 경우 : 현재 상대편 선수 X에게 더 큰 차이로 이기는 선수 B를 내는 최적해가 존재한다고 하자. 그 상대편 선수 X의 상대로 더 큰 차이로 이기는 선수 B와 가장 작은 차이로 이기는 선수 A의 순서를 바꾼다고 생각하자. 그 상대편 선수 X를 상대로는 여전히 승리한다. 기존에 A를 상대하던 상대편 선수 Y는 더 능력치가 높은 우리편 선수 B가 붙게 되므로, 승패가 줄어들 일은 없다.
    - 이길 수 있는데 지는 경우 : 현재 상대편 선수 X에게 지는 선수 B를 내는 최적해가 있다고 하자. 그 상대편 선수를 제일 근소하게 이기는 선수 A와 B의 순서를 바꾼다고 생각하자. 기존에 A를 상대하던 선수 Y와의 경기에서는 질 수 있겠지만, 원래 지던 X를 이기게 되므로 승과 패가 하나씩 증가한다. 따라서 여전히 최적해이다.

- optimal structure 증명
  - 첫 번째 경기에 나갈 선수를 선택하고 나면 남은 선수들을 경기에 배정하는 부분 문제를 얻을 수 있다. 이 때 남은 경기에서도 당연히 최다승을 거두는 것이 좋으니 최적 부분 구조도 자명하게 성립한다.

**Greedy choice property(탐욕적 선택 속성)을 증명할 때에는, 우리가 선택한 방법을 포함하지 않는 최적해가 있다고 가정한 뒤 이것을 적절히 조작하여 우리가 선택한 방법을 포함하는 최적해를 만들어낸다.**

또한 책에서는 나처럼 2중 for loop을 사용하는 대신, `multiset`을 이용해서 상대를 가장 근소한 차이로 이기는 선수를 `O(lgn)`에 찾았다. 따라서 시간 복잡도는 `O(nlgn)`이다.  

## 책의 코드

<details>
<summary>MATCHORDER - 책의 코드  </summary>
<div markdown="1">

```
#include <bits/stdc++.h>
using namespace std;
int n, input;
vector<int> russia, korea;
int main()
{
    int tc;
    cin >> tc;
    while (tc--)
    {
        cin >> n;
        russia.clear();
        korea.clear();
        for (int i = 0; i < n; i++)
        {
            
            cin >> input;
            russia.push_back(input);
        }
        for (int i = 0; i < n; i++)
        {
            
            cin >> input;
            korea.push_back(input);
        }
        sort(korea.begin(), korea.end());
        multiset<int> koreabinary(korea.begin(),korea.end());
        int wins = 0;
        for (int i = 0; i < n; i++)
        {
            if(*koreabinary.rbegin()<russia[i]) {
                koreabinary.erase(koreabinary.begin());
            } else {
                koreabinary.erase(koreabinary.lower_bound(russia[i]));
                wins++;
            }

        }
        cout << wins << endl;
    }
}
```
</div>
</details>  


## 기억할 점
- **Greedy choice property(탐욕적 선택 속성)을 증명할 때에는, 우리가 선택한 방법을 포함하지 않는 최적해가 있다고 가정한 뒤 이것을 적절히 조작하여 우리가 선택한 방법을 포함하는 최적해를 만들어낸다.**
- `multiset`을 사용해서 lower_bound(가장 근소한 차이로 이기는 수)와 upper_bound(가장 근소한 차이로 지는 수)를 `O(lgn)`에 찾을 수 있다.