---
title: "Algospot :: BLOCKGAME (블록 게임)"
date: 2020-01-21 20:42:00 -0400
categories: 알고리즘_문제풀이 
tags: Algospot
---

# 블록 게임 ( [BLOCKGAME](https://algospot.com/judge/problem/read/BLOCKGAME) , 중)

## 문제
5x5칸에 L 모양으로 구성된 3칸짜리 블럭 혹은 2칸짜리 블럭 하나를 게임판에 겹치지 않게 놓을 때, 마지막으로 놓는 사람이 이기는 게임이다.  
게임판의 상태가 주어질 때 이번 차례인 사람이 승리할 수 있는 방법이 있는지 확인해야 한다.  

## 나의 풀이
게임판의 상태를 보고 이번 차례인 사람이 승리할 수 있는 방법이 있는지 확인하는 함수 `didIWin(int gameBoard)`를 만들었다.
- 기저 사례
  - L 모양의 3칸짜리 블럭이나 2칸짜리 블럭을 놓을 수 없는 경우 : 이번 차례인 사람이 졌으므로 false를 리턴한다.
- 재귀 호출
  - `bool lose = false`를 선언한다.
  - 3칸 짜리 블럭 `┘`, `┌`, `└`, `┐` 각각을 놓을 수 있다면, 해당하는 부분을 놓은 것으로 게임판을 변경한 뒤, `lose |= !didIWin(int newgameBoard)`를 실행한다.
    - 다음 차례 사람이 항상 지는 경우, 이번 차례 사람이 무조건 이길 수 있는 방법을 찾은 것이므로, 리턴할 변수 `lose`를 `true`로 설정한다. 
    - 다음 차례 사람이 항상 이기는 경우, `lose`의 값을 바꾸지 않는다.
  - 2칸 짜리 블럭 가로 모양, 세로 모양 각각을 놓을 수 있다면, 해당하는 부분을 놓은 것으로 게임판을 변경한 뒤, `lose |= !didIWin(int newgameBoard)`를 실행한다.
  
시간 제한이 2초이기 때문에 메모이제이션을 통해 속도를 향상시켜야 했다. 게임판은 5X5이므로, 25비트로 표현할 수 있다. 따라서 4바이트(32비트) `int`타입 변수에 이미 블록이 놓인 경우 해당 비트를 1, 안 놓인 경우를 0으로 설정하였다.  
특정 칸에 블록이 놓여있는지를 판단하는 함수 `isNotSharp(int gameBoard, int x, int y)`와, 특정 칸에 블록을 놓을 때 사용할 함수 `makeOne(int x, int y)`를 만들었다.  
메모리제한이 65536kb이고 `(1<<25)X(1Byte)<65536000`이므로, 1바이트 `int8_t`을 인자로 갖는 `int8_t cache[1<<25]`에 계산값을 저장하였다. `didIWin()`은 `true` 혹은 `false`를 리턴하며, `cache[]`의 특정 인덱스가 계산된 값을 가지는지 아닌지를 판단하기 위해 `cache[]`를 -1로 초기화해야 했다. 따라서 0과 1만 표현할 수 있는 `bool`이 아닌 `int8_t`을 사용했다.  
실행 시간을 단축시키기 위해, 게임판을 90도씩 반복해서 돌리거나 좌우 대칭/ 상하 대칭인 게임판을 생각해도 승패는 달라지지 않는 점을 이용하려 했다. `cache[]`의 값을 참고할 때, 현재 게임판을 90도씩 돌린 게임판, 좌우 대칭/상하 대칭인 게임판에 대한 계산값이 있는지를 모두 확인하였다. 하지만 시간초과를 해결하지 못해서, 편법이지만 `memset(cache, -1, sizeof cache)`를 테스트 케이스의 사이에 실행하지 않음으로써 해결했다.

## 내 코드

<details>
<summary>BLOCKGAME - 내 코드</summary>
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
#include <bitset>

#ifdef _MSC_VER
#define _CRT_SCURE_NO_WARNINGS
#endif

using namespace std;
vector<string> board;
int8_t cache[1 << 25];
int getIndex(int gameBoard);
bool didIWin(int gameBoard);
bool possible(int gameBoard);
int main()
{
    ios::sync_with_stdio(false);
    cin.tie(NULL);
    int iters;
    cin >> iters;
    memset(cache, -1, sizeof cache);
    for (int i = 0; i < iters; i++)
    {
        
        
        board.clear();        
        int startBoard = 0;
        for (int x = 0; x < 5; x++)
        {
            string temp;
            cin >> temp;
            board.push_back(temp);
            for (int y = 0; y < 5; y++)
            {
                startBoard *= 2;
                if (board.back()[y] == '#')
                    startBoard+=1;
            }
            
        }
        cout << (didIWin(startBoard) ? "WINNING" : "LOSING") << endl;
    }
    return 0;
}

bool isNotSharp(int gameBoard, int x, int y)
{
    return (gameBoard & (1 << (5 * (4 - x) + 4 - y))) == 0;
}
int makeOne(int x, int y)
{
    return (1 << (5 * (4 - x) + (4 - y)));
}

bool didIWin(int gameBoard)
{


    if (!possible(gameBoard))
    {
        return false;
    }
    int8_t &ret = cache[getIndex(gameBoard)];
    if (ret != -1)
    {
        return ret == 1 ? true : false;
    }
    bool lose = false;
    for (int x = 0; x < 5; x++)
    {
        if(lose) break;
        for (int y = 0; y < 5; y++)
        {
            if(lose) break;
            if (y < 4)
            {
                if (isNotSharp(gameBoard, x, y) && isNotSharp(gameBoard, x, y + 1))
                {
                    int newGameBoard = (gameBoard | makeOne(x, y) | makeOne(x, y + 1));
                    if (!lose && x < 4 && isNotSharp(gameBoard, x + 1, y))
                        lose |= !didIWin(newGameBoard | makeOne(x + 1, y));
                    if (!lose && x < 4 && isNotSharp(gameBoard, x + 1, y + 1))
                        lose |= !didIWin(newGameBoard | makeOne(x + 1, y + 1));
                    if (!lose && x > 0 && isNotSharp(gameBoard, x - 1, y + 1))
                        lose |= !didIWin(newGameBoard | makeOne(x - 1, y + 1));
                    if (!lose && x > 0 && isNotSharp(gameBoard, x - 1, y))
                        lose |= !didIWin(newGameBoard | makeOne(x - 1, y));

                    lose |= !didIWin(newGameBoard);
                }
            }
            if (x < 4)
            {
                if (!lose && isNotSharp(gameBoard, x, y) && isNotSharp(gameBoard, x + 1, y))
                {
                    lose |= !didIWin(gameBoard | makeOne(x, y) | makeOne(x + 1, y));
                }
            }
        }
    }
    ret = lose ? 1 : 0;
    return lose;
}

int getIndex(int gameBoard)
{
    int index = 0;
    for (int i = 0; i < 5; i++)
    {
        for (int j = 0; j < 5; j++)
        {
            index *= 2;
            index += isNotSharp(gameBoard, i, j) ? 0 : 1;
        }
    }
    return index;
}

bool possible(int gameBoard)
{
    bool lose = false;
    for (int x = 0; x < 5; x++)
    {
        for (int y = 0; y < 5; y++)
        {
            if (y < 4)
            {
                if (isNotSharp(gameBoard, x, y) && isNotSharp(gameBoard, x, y + 1))
                {
                    lose = true;
                    return true;
                }
            }
            if (x < 4)
            {
                if (isNotSharp(gameBoard, x, y) && isNotSharp(gameBoard, x+1, y))
                {
                    lose = true;
                    return true;
                }
            }
        }
    }
    return lose;
}

```  

</div>
</details>  
  

## 책의 풀이
나는 더 이상 놓을 곳이 없는 상태를 기저 사례로 정했고, 기저 사례인지 확인하기 위해 `possible(int gameBoard)`를 사용했었다. 하지만 책처럼 기저 사례를 따로 구분하지 않아야 시간초과가 뜨지 않았다.  
책에서는 `int8_t`가 아닌 `char`를 사용했지만, 둘 다 1바이트여서 사용한 의도는 같다고 생각한다.  


## 책의 코드

<details>
<summary>BLOCKGAME - 내 코드</summary>
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
#include <bitset>

#ifdef _MSC_VER
#define _CRT_SCURE_NO_WARNINGS
#endif

using namespace std;
vector<string> board;
int8_t cache[1 << 25];
int getIndex(int gameBoard);
bool didIWin(int gameBoard);
int main()
{
    ios::sync_with_stdio(false);
    cin.tie(NULL);
    int iters;
    cin >> iters;
    
    for (int i = 0; i < iters; i++)
    {
        memset(cache, -1, sizeof cache);
        
        board.clear();        
        int startBoard = 0;
        for (int x = 0; x < 5; x++)
        {
            string temp;
            cin >> temp;
            board.push_back(temp);
            for (int y = 0; y < 5; y++)
            {
                startBoard *= 2;
                if (board.back()[y] == '#')
                    startBoard+=1;
            }
            
        }
        cout << (didIWin(startBoard) ? "WINNING" : "LOSING") << endl;
    }
    return 0;
}

bool isNotSharp(int gameBoard, int x, int y)
{
    return (gameBoard & (1 << (5 * (4 - x) + 4 - y))) == 0;
}
int makeOne(int x, int y)
{
    return (1 << (5 * (4 - x) + (4 - y)));
}

bool didIWin(int gameBoard)
{

    int8_t &ret = cache[getIndex(gameBoard)];
    if (ret != -1)
    {
        return ret == 1 ? true : false;
    }
    bool lose = false;
    for (int x = 0; x < 5; x++)
    {
        if(lose) break;
        for (int y = 0; y < 5; y++)
        {
            if(lose) break;
            if (y < 4)
            {
                if (isNotSharp(gameBoard, x, y) && isNotSharp(gameBoard, x, y + 1))
                {
                    int newGameBoard = (gameBoard | makeOne(x, y) | makeOne(x, y + 1));
                    if (!lose && x < 4 && isNotSharp(gameBoard, x + 1, y))
                        lose |= !didIWin(newGameBoard | makeOne(x + 1, y));
                    if (!lose && x < 4 && isNotSharp(gameBoard, x + 1, y + 1))
                        lose |= !didIWin(newGameBoard | makeOne(x + 1, y + 1));
                    if (!lose && x > 0 && isNotSharp(gameBoard, x - 1, y + 1))
                        lose |= !didIWin(newGameBoard | makeOne(x - 1, y + 1));
                    if (!lose && x > 0 && isNotSharp(gameBoard, x - 1, y))
                        lose |= !didIWin(newGameBoard | makeOne(x - 1, y));

                    lose |= !didIWin(newGameBoard);
                }
            }
            if (x < 4)
            {
                if (!lose && isNotSharp(gameBoard, x, y) && isNotSharp(gameBoard, x + 1, y))
                {
                    lose |= !didIWin(gameBoard | makeOne(x, y) | makeOne(x + 1, y));
                }
            }
        }
    }
    ret = lose ? 1 : 0;
    return lose;
}

int getIndex(int gameBoard)
{
    int index = 0;
    for (int i = 0; i < 5; i++)
    {
        for (int j = 0; j < 5; j++)
        {
            index *= 2;
            index += isNotSharp(gameBoard, i, j) ? 0 : 1;
        }
    }
    return index;
}
```  

</div>
</details>  
## 기억할 점
- DP에 항상 기저 사례가 필요한 것은 안디ㅏ.
