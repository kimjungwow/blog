---
title: "Algospot :: NUMBERGAME (숫자게임)"
date: 2020-01-21 10:16:00 -0400
categories: 알고리즘_문제풀이 
tags: Algospot
---

# 숫자게임 ( [NUMBERGAME](https://algospot.com/judge/problem/read/NUMBERGAME) , 하)

## 문제
n개의 정수가 일렬로 놓인 게임판에서 현우와 서하가 게임을 한다. 현우가 먼저 시작하며, 자신의 차례에는 두 가지 일 중 하나를 할 수 있다.
- 게임판의 왼쪽이나 오른쪽 끝에서 숫자 하나를 택해서 가져간다.
- 게임판에 2개 이상의 숫자가 있을 경우 왼쪽 끝 혹은 오른쪽 끝에서 2개를 지운다.
현우와 서하가 모두 최선을 다할 때, 두 사람의 최종 점수 차이를 구해야 한다.

## 나의 풀이
[틱택토](https://kimjungwow.github.io/%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98_%EB%AC%B8%EC%A0%9C%ED%92%80%EC%9D%B4/TICTACTOE/)에서는 게임판을 보고 누구의 차례인지 파악해야 했다. 하지만 숫자게임에서는 항상 선플레이어인 현우의 입장에서 서하와 얼마나 차이나는지 계산하면 된다.  
게임판의 길이는 `1<=n<=50`이며, 게임이 진행되며 양쪽에서 숫자를 지워진다. 따라서 게임판의 남은 부분과, 현재까지 두 사람의 점수 차이를 기억함으로써 현재 게임의 상태를 알 수 있다.  
게임판이 `start번째 수 ~ end번째 수 `만 남은 상태에서, 두 사람이 최선을 다 했을 때 점수 차이를 계산하는 함수 `numbergame(int start, int end)`를 만들었다. 
- 기저 사례
  - `end<start`면 0을 리턴한다.
  - `end==start`면 start번째 수를 리턴한다.
- 재귀 호출 : 다음 네 가지 경우를 모두 실행하고, 최댓값을 리턴한다.
  - 왼쪽의 수 하나만 가져오는 경우 : `start번째 수 - numbergame(start+1,end)`
  - 오른쪽의 수 하나만 가져오는 경우 : `end번째 수 - numbergame(start,end+1)`
  - 왼쪽의 수 2개를 없애는 경우 : `-numbergame(start+2,end)`
  - 오른쪽의 수 2개를 없애는 경우 : `-numbergame(start,end+2)`
이 때, 현재 게임판에서 자신이 얻는 점수는 `자신의 가져온 수 - 남은 게임판에서 상대방이 최선을 다할 경우 상대의 점수`이기 때문에 재귀 호출시 -1을 곱해준다.  
  
`int start`와 `int end`를 이용해 메모이제이션을 했다. 각 수가 -1000 이상이고, 게임판의 최대 길이가 50이기 때문에 넉넉하게 `memset(cache, -500000, sizeof cache)`로 초기화했는데, 출력해보니 `-500000`이 아닌 `-522133280`이 들어있었다.  
[memset 함수 설명](https://en.cppreference.com/w/cpp/string/byte/memset)에 따르면, `memset`은 `void* memset(void* dest, int ch, std:;size_t count)`에서 `ch`라는 값을 `unsigned char type`으로 바꾼 뒤, `dest부터 첫 count개의 character`에 복사한다고 한다. **즉, `memset`은 `int ch`의 마지막 8비트(1바이트)를 `dest부터 count개의 바이트`에 복사하는 것이다.**  
  
![-500000](https://i.imgur.com/1PPrrpR.png)  
`-500000`의 마지막 8 비트는 `1110 0000`이다. 따라서 `int cache[51][51]`인 `cache`에 대해 `memset(cache, -500000, sizeof cache)`를 실행하면, `1110 0000`를 `cache`에 채우는 것이다.  
`cache`는 `int` 타입으로 정의 했기 때문에, 각 인덱스별로 4바이트인 `int``1110 0000 1110 0000 1110 0000 1110 0000`인 `-522133280`가 들어있는 것처럼 보이게 된다.  
![-522133280](https://i.imgur.com/c3SLJPm.png)  
  
지금까지 메모이제이션할 때 -1이나 0을 넣었는데, -1의 마지막 8비트는 `1111 1111`이고, 0의 마지막 8비트는 `0000 0000`이다. 이것이 4바이트 반복되어서 `1111 1111 1111 1111 1111 1111 1111 1111`과 `0000 0000 0000 0000 0000 0000 0000 0000`이 되어도 그대로 -1과 0이기 때문에 문제되지 않았다.  
결국 memset대신 for loop을 이용해 -500000을 채워넣었다.

## 나의 코드

<details>
<summary>NUMBERGAME - 내 코드</summary>
<div markdown="1">

```
#include <stdio.h>
#include <string.h>
#include <iostream>
#include <utility>
#include <vector>
#include <algorithm>
#include <climits>
#include <string>
#include <list>

#ifdef _MSC_VER
#define _CRT_SCURE_NO_WARNINGS
#endif

using namespace std;
int n;
vector<int> l;
int numbergame(int start, int end);
int cache[51][51];
int main()
{
    ios::sync_with_stdio(false);
    cin.tie(NULL);
    int iters;
    cin >> iters;

    for (int i = 0; i < iters; i++)
    {
        cin >> n;
        
        l.clear();
        for (int j=0;j<n;j++) {
            int temp;
            cin >>temp;
            l.push_back(temp);
        }
        for (int x=0;x<51;x++) {
            for (int y=0;y<51;y++) {
                cache[x][y]=-500000;
            }
        }
        // memset(cache,-500000,sizeof cache);
        cout<<numbergame(0,n-1)<<endl;

    }
    return 0;
}

int numbergame(int start, int end) {
    
    if(end<=start) {
        return end<start?0:l[start];
    }
    int& ret = cache[start][end];
    if(ret!=-500000)
        return ret;

    ret = max(ret, l[start]-numbergame(start+1,end));
    ret = max(ret, l[end]-numbergame(start,end-1));
    ret = max(ret, -numbergame(start+2,end));
    ret = max(ret, -numbergame(start,end-2));
    return ret;
}
```  

</div>
</details>  
  
## 기억할 점
- **`memset`은 바이트 단위로 복사한다.**
