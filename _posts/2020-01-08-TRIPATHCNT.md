---
title: "TRIPATHCNT (삼각형 위의 최대 경로 개수 세기)"
date: 2020-01-08 20:24:00 -0400
categories: 알고리즘
tags: AOJ
---

# 삼각형 위의 최대 경로 개수 세기 ( [TRIPATHCNT](https://algospot.com/judge/problem/read/TRIPATHCNT) , 중)

## 문제
삼각형 위의 최대 경로의 길이만 구하던 [TRIANGLEPATH](https://algospot.com/judge/problem/read/TRIANGLEPATH)에서 난이도를 높혀, 최대 길이를 갖는 경로의 개수를 구해야 한다.  

## 나의 풀이
TRIANGLEPATH에서 했듯이 path2()를 이용하여 각 위치 별 최대 경로의 길이를 얻을 수 있다.  
처음에는 path2()를 통해 최대 경로의 길이만 얻고, 다시 처음의 input 삼각형의 (1,1)에서 시작해 아래로 내려가며 합이 최대 경로의 길이가 되는 경로의 개수를 찾으려 했다.  
하지만 array of map을 사용하며 코드가 복잡해지고, 메모리 초과 오류가 발생하며 잘 풀리지 않았다.  
그래서 책을 살짝 보고 path2(1,1)를 실행하면 cache[101][101]에는 각 인덱스에서 시작하는 경로의 최대 길이가 저장되므로, 이를 이용하면 된다는 아이디어를 얻었다.  
  
count(x,y)가 (x,y)에서 시작해 아래로 내려가는 경로 중 최대 길이를 갖는 경로의 수라고 하자.  
그러면 문제의 정답은 count(1,1)이다.  
기저 사례로 가장 아래 행에 대해서는 count()에서 1을 리턴하였다.  
처음의 input을 block[101][101]에 저장하고, 각 인덱스에서 시작하는 경로의 최대 길이를 cache[101][101]에 저장하였다.  
따라서 `cache[x][y] - block[x][y] == cache[x+1][y]`이면 같은 열의 다음 행으로 내려가도 최대 길이를 갖는 경로 위에 있는 것이다.  
마찬가지로 `cache[x][y] - block[x][y] == cache[x+1][y+1]`이면 다음 열의 다음 행으로 내려가도 최대 길이를 갖는 경로 위에 있는 것이다.  
count()의 계산 값은 result[101][101]에 메모이제이션하며 count(1,1)을 구했다.  
  
## 나의 코드

<details>
<summary>LIS - 내 코드</summary>
<div markdown="1">

```
#include <stdio.h>
#include <string.h>
#include <iostream>
#include <utility>
#include <vector>
#include <algorithm>

#ifdef _MSC_VER
#define _CRT_SCURE_NO_WARNINGS
#endif

using namespace std;
int block[101][101];
int result[101][101];
int cache[101][101];
int path2(int row, int col);
int n;
int count(int row, int col);

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
        memset(block, 0, sizeof block);
        memset(result, -1, sizeof result);
        memset(cache, 0, sizeof cache);

        cin >> n;
        // 주어진 input을 block[101][101]에 저장
        for (int x = 1; x <= n; x++)
        {
            for (int y = 1; y <= x; y++)
                cin >> block[x][y];
        }
        
        // 각 인덱스에서 시작하는 최대 경로의 길이를 cache[101][101]에 저장
        path2(1, 1);

        // cache[101][101]와 block[101][101]을 이용하여, (1,1)에서 출발하는 최대 길이 경로의 개수를 구함
        answer.push_back(count(1, 1));

    }
    for (int i = 0; i < iters; i++)
        cout << answer[i] << endl;
    return 0;
}

int count(int row, int col)
{
    if (result[row][col] != -1)
        return result[row][col];
    int &ret = result[row][col];
    ret = 0;
    if (row == n)
        ret = 1;
    else
    {
        // if 조건문이 성립하면 같은 열의 다음 행으로 내려가도 최대 길이의 경로에 있는 것이다.
        if (cache[row][col] - block[row][col] == cache[row + 1][col])
            ret += count(row + 1, col);
        // if 조건문이 성립하면 다음 열의 다음 행으로 내려가도 최대 길이의 경로에 있는 것이다.
        if (cache[row][col] - block[row][col] == cache[row + 1][col + 1])
            ret += count(row + 1, col + 1);
    }
    return ret;
}


int path2(int row, int col)
{
    if (cache[row][col] != 0)
        return cache[row][col];
    int &ret = cache[row][col];
    ret = block[row][col];
    if (row != n)
        ret += max(path2(row + 1, col), path2(row + 1, col + 1));
    return ret;
}
```  

</div>
</details>  


## 책의 풀이
책의 풀이 path2()를 통해 얻은 cache[101][101]을 이용하지만, block[101][101]은 이용하지 않는다.  
count(x,y)의 값을 구할 때, `cache[x+1][y]`와 `cache[x+1][y+1]`의 크기를 비교하여 count(x+1,y)와 count(x+1,y+1)을 더할지 말지를 정한다.  

## 책의 코드

<details>
<summary>LIS - 책의 코드</summary>
<div markdown="1">

  
```
#include <stdio.h>
#include <string.h>
#include <iostream>
#include <utility>
#include <vector>
#include <algorithm>

#ifdef _MSC_VER
#define _CRT_SCURE_NO_WARNINGS
#endif

using namespace std;
int block[101][101];
int result[101][101];
int cache[101][101];
int path2(int row, int col);
int n;
int count(int row, int col);

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
        memset(block, 0, sizeof block);
        memset(result, -1, sizeof result);
        memset(cache, 0, sizeof cache);

        cin >> n;
        // 주어진 input을 block[101][101]에 저장
        for (int x = 1; x <= n; x++)
        {
            for (int y = 1; y <= x; y++)
                cin >> block[x][y];
        }
        
        // 각 인덱스에서 시작하는 최대 경로의 길이를 cache[101][101]에 저장
        path2(1, 1);

        // cache[101][101]와 block[101][101]을 이용하여, (1,1)에서 출발하는 최대 길이 경로의 개수를 구함
        answer.push_back(count(1, 1));

    }
    for (int i = 0; i < iters; i++)
        cout << answer[i] << endl;
    return 0;
}

int count(int row, int col)
{
    if (row == n)
        return 1;
    int &ret = result[row][col];
    if(ret!=-1)
        return ret;
    ret = 0;
    if (cache[row+1][col]>=cache[row+1][col+1])
        ret += count(row+1,col);
    if (cache[row+1][col]<=cache[row+1][col+1])
        ret += count(row+1,col+1);
    

    return ret;
}


int path2(int row, int col)
{
    if (cache[row][col] != 0)
        return cache[row][col];
    int &ret = cache[row][col];
    ret = block[row][col];
    if (row != n)
        ret += max(path2(row + 1, col), path2(row + 1, col + 1));
    return ret;
}
```
</div>
</details>  
  
## 개선할 점
count(x,y)의 값을 구할 때, 나는 `cache[x][y]-block[x][y]`와 `block[x+1][y]`, `block[x+1][y+1]`을 비교했다.  
하지만 책은 `cache[x+1][y]`와 `cache[x+1][y+1]`의 크기를 비교하여 count(x+1,y)와 count(x+1,y+1)을 더할지 말지를 정했다.

내가 `cache[x+1][y]`와 `cache[x+1][y+1]`을 비교하지 않고, `cache[x][y]-block[x][y]`를 따진 것은 서로 다른 최대 길이 경로가 같은 행에서 다른 cache값을 가질 수도 있다고 생각했기 때문이다. → (A)  
  
예를 들어 block이 다음과 같으면,
```
1
1 2
2 1 1
1 1 1 1
```  
cache는 다음과 같다.  
```
5
4 4
3 2 2
1 1 1 1
```  
  
이 때, 경로 (1-1-2-1)은 cache에서 (5-4-3-1)이지만, 경로 (1-2-1-1)은 cache에서 (5-3-2-1)이다. 이 때 count(1,1)=5이다.  
  
흥미로운 점은 count(3,1)=3이고, count(3,2)=count(3,3)=2이지만, (3,1), (3,2), (3,3) 세 점 모두 최대 길이 경로에 속한다.  
count(2,1)을 구할 때는, (3,1)로 가는 경우만 최대 경로이고, count(2,2)를 구할 때는 (3,2), (3,3) 둘 다 최대 경로이므로, 이 경우 (A)은 참이지만, 이는 문제되지 않는다.  
  
(A)에 해당하는 한 경우에는 `cache[x+1][y]`와 `cache[x+1][y+1]`을 비교해도 되는 것만 보였다.  
실제로,
`cache[x][y]`  
` = max ( cache[x+1][y] + block[x][y], cache[x+1][y+1] + block[x][y]`  
` = block[x][y] + max (cache[x+1][y], cache[x+1][y+1])`  
이므로 `cache[x][y]`는 `max(cache[x+1][y], cache[x+1][y+1])`에 의해 결정된다. → (B)  
그래서 책처럼 `cache[x+1][y]`와 `cache[x+1][y+1]`만 비교해도 된다.  
쉽게 (B)를 보일 수 있는데 (A)처럼 잘못 생각한 것이 아쉽다.  

각 인덱스에 대하여 path2()와 count()가 한 번씩 실행되므로 시간복잡도는 나의 풀이와 책의 풀이 모두 O(n^2)이다.  