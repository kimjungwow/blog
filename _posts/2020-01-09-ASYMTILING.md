---
title: "Algospot :: ASYMTILING (비대칭 타일링)"
date: 2020-01-08 20:24:00 -0400
categories: 알고리즘_문제풀이 
tags: Algospot
---

# 삼각형 위의 최대 경로 개수 세기 ( [ASYMTILING](https://algospot.com/judge/problem/read/ASYMTILING) , 하)

## 문제
2xn 크기의 사각형을 2x1 크기의 사각형으로 빈틈없이 채우는 경우의 수를 구하던 [TILING2](https://algospot.com/judge/problem/read/TILING2)에서 난이도를 높혀, 좌우 비대칭으로 채우는 경우의 수를 구해야 한다.  

## 나의 풀이
TILING2에서 했듯이 tiling2()를 이용하여 2xn 크기의 사각형을 2x1 크기의 사각형으로 빈틈없이 채우는 경우의 수를 구했다.  
이제 2xn 크기의 사각형을 2x1 크기의 사각형으로 빈틈없이 `좌우대칭`으로 채우는 경우의 수를 asymtiling()으로 구한 뒤, tiling2()의 결과에서 빼주었다.  
좌우대칭으로 채우는 경우를 두 가지로 나누었다.
- 양쪽 끝에 세로로 채워지는 경우  
- 양쪽 끝에 가로로 채워지는 경우  
모든 경우는 이 두 가지 선택지에 포함되며, 양쪽 선택지 모두에 포함되는 경우는 없다.  
2xn 사각형을 좌우대칭으로 채울 때 양쪽 끝에 세로로 채워지는 경우의 수는 2x(n-2) 크기의 사각형을 좌우대칭으로 채우는 경우의 수와 같다.  
마찬가지로 2xn 사각형을 좌우대칭으로 채울 때 양쪽 끝에 가로로 채워지는 경우의 수는 2x(n-4) 크기의 사각형을 좌우대칭으로 채우는 경우의 수와 같다.  
이 때, n<4를 기저 사례로 처리해주었다. n=1,3일 때는 좌우대칭으로 채우는 경우의 수가 1이고, n=2일 때는 2이다. n=0인 경우는 아무 것도 놓지 않아도 채워졌으며 좌우대칭으로 생각할 수 있으므로 경우의 수가 1이라고 생각했다.  
이는 n=4일 때 좌우대칭인 경우의 수가 `양 끝이 세로로 채워지는 경우 (2)` + `양 끝이 가로로 채워지는 경우 (1)` = 3인 것에서 아이디어를 얻었다.  
이후 `asymtiling(n) = (asymtiling(n-2) + asymtiling(n-4))%MOD`으로 asymtiling()을 정의했다.  
`비대칭인 경우의 수 = 전체 경우의 수 - 대칭인 경우의 수` 였는데 우변이 음수가 될 수 있기 때문에, `(전체 경우의 수 - 대칭인 경우의 수 + MOD) % MOD`로 문제를 풀었다.  

  
## 나의 코드

<details>
<summary>ASYMTILING - 내 코드</summary>
<div markdown="1">

```
#include <stdio.h>
#include <string.h>
#include <iostream>
#include <utility>
#include <vector>
#include <algorithm>
#include <climits>

#ifdef _MSC_VER
#define _CRT_SCURE_NO_WARNINGS
#endif

using namespace std;
const int MOD = 1000000007;
int cache[101];
int asymcache[101];
int n;
int tiling2(int n);
int asymtiling(int n);
int main()
{
    ios::sync_with_stdio(false);
    cin.tie(NULL);
    int iters;
    cin >> iters;
    vector<int> answer;
    
    for (int i = 0; i < iters; i++)
    {
        // 메모이제이션할 메모리 초기화
        memset(cache, 0, sizeof cache);
        memset(asymcache, 0, sizeof asymcache);
        
        cin >> n;
        answer.push_back((tiling2(n)-asymtiling(n)+MOD)%MOD);
    }
    for (int i = 0; i < iters; i++)
    {
        cout << answer[i] << endl;
    }
    return 0;
}

int tiling2(int n)
{
    if (n <= 2)
        return (n > 0 ? n : 0);
    int &ret = cache[n];
    if( ret != 0)
        return ret;
    ret = (tiling2(n - 2) + tiling2(n - 1) ) % MOD;
    return ret;
}

int asymtiling(int n) {
    if (n<=3)
        return (n==2?n:1);
    int& ret = asymcache[n];
    if( ret != 0)
        return ret;
    ret =  (asymtiling(n-2)+asymtiling(n-4))%MOD;
    return ret;
    
}
```  

</div>
</details>  


## 책의 풀이
책에서는 두 가지 방법을 이용했다.  
- 좌우 대칭인 경우의 수 세기
- 좌우 비대칭인 경우의 수 세기
  
좌우대칭인 경우의 수를 셀 때 나는 양 끝의 타일이 어떻게 놓이는 지 생각했지만, 책에서는 정가운데의 타일이 어떻게 놓이는지 생각했다.  
n이 홀수일 때는 정가운데의 타일이 세로로 놓이는 경우만 좌우대칭이며, 그 경우의 수는 2개의 2x(n-2) 타일이 같은 배치를 갖는 경우의 수이다. 따라서 tiling2(n/2)이다.  
n이 짝수일 때는 정가운데의 두 개의 타일이 가로로 놓일 수도 있고, 그렇지 않을 수도 있다. 전자는 경우의 수가 tiling2(n/2-1)이고, 후자는 tiling2(n/2)이다.
따라서 tiling2(n)을 구한 뒤, n에 따라 tiling2(n/2)와 tiling2(n/2-1)을 적절히 빼주었다.  
책의 풀이에 따라 코딩하던 중, tiling2(n)에서 n<0일 때 1을 리턴해야 함을 발견했다. 
  
좌우 비대칭인 경우의 수를 셀 때는 양 끝의 타일이 어떻게 놓이는 지 생각했다. 
- 양 끝이 세로인 경우
  - 양 끝이 세로로 같으므로, 그 안의 2x(n-2) 사각형을 비대칭으로 채워야한다. 이 경우의 수는 asymtiling(n-2)이다.
- 양 끝이 가로인 경우
  - 양 끝이 가로로 같으므로, 그 안의 2x(n-4) 사각형을 비대칭으로 채워야한다. 이 경우의 수는 asymtiling(n-4)이다.
- 왼쪽만 세로인 경우
  - 양 끝이 비대칭이므로, 그 안의 2x(n-3) 사각형은 대칭이어도 상관 없다. 이 경우의 수는 tiling2(n-3)이다.
- 오른쪽만 세로인 경우
  - 양 끝이 비대칭이므로, 그 안의 2x(n-3) 사각형은 대칭이어도 상관 없다. 이 경우의 수는 tiling2(n-3)이다.
네 가지 경우의 수를 더해줌으로써 좌우 비대칭인 경우의 수를 직접 구했다.


## 책의 코드 1

<details>
<summary>TRIPATHCNT - 책의 코드</summary>
<div markdown="1">

  
```
#include <stdio.h>
#include <string.h>
#include <iostream>
#include <utility>
#include <vector>
#include <algorithm>
#include <climits>

#ifdef _MSC_VER
#define _CRT_SCURE_NO_WARNINGS
#endif

using namespace std;
const int MOD = 1000000007;
int cache[101];
int asymcache[101];
int n;
int tiling2(int n);
int asymtiling(int n);
int main()
{
    ios::sync_with_stdio(false);
    cin.tie(NULL);
    int iters;
    cin >> iters;
    vector<int> answer;
    
    for (int i = 0; i < iters; i++)
    {
        // 메모이제이션할 메모리 초기화
        memset(cache, 0, sizeof cache);
        memset(asymcache, 0, sizeof asymcache);
        
        cin >> n;
        answer.push_back(asymtiling(n));
    }
    for (int i = 0; i < iters; i++)
    {
        cout << answer[i] << endl;
    }
    return 0;
}

int tiling2(int n)
{
    if (n <= 2)
        return (n > 0 ? n : 1);
    int &ret = cache[n];
    if( ret != 0)
        return ret;
    ret = (tiling2(n - 2) + tiling2(n - 1) ) % MOD;
    return ret;
}

int asymtiling(int n) {    
    int& ret = asymcache[n];
    if( ret != 0)
        return ret;
    ret = tiling2(n);
    if (n%2==1)
        ret = (ret - tiling2(n/2) + MOD)%MOD;
    else {
        ret = (ret - tiling2(n/2) + MOD)%MOD;
        ret = (ret - tiling2(n/2-1) + MOD)%MOD;
    }
    return ret;    
}
```
</div>
</details>  
  

## 책의 코드 2

<details>
<summary>TRIPATHCNT - 책의 코드</summary>
<div markdown="1">

  
```
#include <stdio.h>
#include <string.h>
#include <iostream>
#include <utility>
#include <vector>
#include <algorithm>
#include <climits>

#ifdef _MSC_VER
#define _CRT_SCURE_NO_WARNINGS
#endif

using namespace std;
const int MOD = 1000000007;
int cache[101];
int asymcache[101];
int n;
int tiling2(int n);
int asymtiling(int n);
int main()
{
    ios::sync_with_stdio(false);
    cin.tie(NULL);
    int iters;
    cin >> iters;
    vector<int> answer;
    
    for (int i = 0; i < iters; i++)
    {
        // 메모이제이션할 메모리 초기화
        memset(cache, 0, sizeof cache);
        memset(asymcache, 0, sizeof asymcache);
        
        cin >> n;
        answer.push_back(asymtiling(n));
    }
    for (int i = 0; i < iters; i++)
    {
        cout << answer[i] << endl;
    }
    return 0;
}

int tiling2(int n)
{
    if (n <= 2)
        return (n > 0 ? n : 0);
    int &ret = cache[n];
    if( ret != 0)
        return ret;
    ret = (tiling2(n - 2) + tiling2(n - 1) ) % MOD;
    return ret;
}

int asymtiling(int n) {    
    if (n<=2)
        return 0;
    int& ret = asymcache[n];
    if( ret != 0)
        return ret;
    
    ret = asymtiling(n-2)%MOD;
    ret = (ret + asymtiling(n-4) )%MOD;
    ret = (ret + tiling2(n-3) )%MOD;
    ret = (ret + tiling2(n-3) )%MOD;

    return ret;    
}
```
</div>
</details>  
  
## 기억할 점
- 이 문제처럼 %MOD를 구하는 과정에서 뺄셈이 들어가면, 음수%MOD가 실행되지 않도록 MOD를 더한뒤 %MOD를 실행해줘야 한다.
- 이 문제는 대칭/비대칭인 두 가지 경우의 수로 나눴다. 또한 asymtiling() 내부에서 경우의 수를 나눌 때도 아래의 속성이 성립해야 한다.
  - 모든 경우는 이 선택지들에 포함됨
  - 어떤 경우도 두 개 이상의 선택지에 포함되지 않음

## 개선할 점
- 내 코드에는 직접 계산한 기저사례가 많지만, 코드가 제일 짧고 경우의 수도 가장 적게 나눴기 때문에 만족스럽다.