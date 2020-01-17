---
title: "Algospot :: TICTACTOE (틱택토)"
date: 2020-01-17 20:28:00 -0400
categories: 알고리즘_문제풀이 
tags: Algospot
---

# 틱택토 ( [TICTACTOE](https://algospot.com/judge/problem/read/TICTACTOE) , 하)

## 문제
틱택토는 3X3 게임판에서 번갈아 `x`와 `o`를 그리며 같은 도형을 세 개 연속으로 그리면 이기는 게임이다. 항상 `x`가 선이며, 가로, 세로, 대각선 방향에 상관 없이 연속 세 개를 그리면 이긴다.  
현재 게임 상태가 주어질 때, `x`가 이기는 상황이면 `x`를, `o`가 이기는 상황이면 `o`를, 양쪽 다 최선을 다하면 비기는 상황이면 `TIE`를 출력해야 한다.  

## 나의 풀이
우선 게임판의 상태를 보고 누구의 차례인지 판단하는 `isXTurn()`을 만들었다. `isXTurn()`이 있기 때문에, 현재 게임판의 상태만 인풋으로 전달하면 되고, 누구의 차례인지는 따로 전달할 필요가 없다. 이후 게임판의 상태가 주어졌을 때 한 쪽의 승리로 이미 끝났는지를 판단하는 `isFinished()`와 게임판이 이미 꽉 찼는지를 판단하는 `allDone()`을 만들었다. 게임판이 꽉 안 찼지만 한 쪽이 이겼을 수 있기 때문에 두 개의 함수를 만들었으며, `allDone()`이 참이지만 `isFinished()`가 0이면(어느 쪽의 승리도 아니면) 게임은 비긴 것이다.  
위의 함수들을 이용하여 주어진 게임판을 보고 현재 차례 플레이어가 이길지 비길지 질지를 판단하는 `ticTacToe()`를 만들었다. 비어 있는 곳에 자신의 돌을 놓아보고, 한 번이라도 자신이 무조건 이긴다면 (**자신의 돌을 놓은 새로운 게임판 newboard에 대해 `ticTacToe(newboard)==-1`인 경우**) 자신이 100% 이길 수 있는 수가 있는 것이므로 1을 리턴한다. 비어 있는 고셍 자신의 돌을 놓았지만 모두 자신이 무조건 진다면 자신이 이기거나 비길 수 있는 수가 없으므로 -1을 리턴한다. 그렇지 않다면 0을 리턴한다.  
## 나의 코드

<details>
<summary>TICTACTOE - 내 코드</summary>
<div markdown="1">

```
#include <stdio.h>
#include <string.h>
#include <iostream>
#include <utility>
#include <vector>
#include <algorithm>
#include <list>
#include <string>
#include <climits>
#include <bitset>

#ifdef _MSC_VER
#define _CRT_SCURE_NO_WARNINGS
#endif
using namespace std;
int ticTacToe(char givenBoard[3][3]);
int isFinished(char givenBoard[3][3]);
bool isXTurn(char givenBoard[3][3]);
bool allDone(char givenBoard[3][3]);
void printBoard(int answer, char givenBoard[3][3]);
char board[3][3];

int main()
{
    int testcases;
    ios::sync_with_stdio(false);
    cin.tie(NULL);
    cin >> testcases;
    for (int i = 0; i < testcases; i++)
    {
        memset(board, 0, sizeof board);
        for (int j = 0; j < 3; j++)
        {
            for (int k = 0; k < 3; k++)
            {
                cin >> board[j][k];
            }
        }
        int turnWin = ticTacToe(board);
        if (turnWin == 0)
            cout << "TIE" << endl;
        else if (turnWin * (isXTurn(board) ? 1 : -1) == 1)
            cout << "x" << endl;
        else
            cout << "o" << endl;
    }

    return 0;
}



// Return if current turn player can win : 1
// if current turn player can at least draw : 0
// if current turn player always loses : -1
int ticTacToe(char givenBoard[3][3])
{
    if (isFinished(givenBoard) != 0 || allDone(givenBoard))
    {
        return isFinished(givenBoard);
    }
    char turn = isXTurn(givenBoard) ? 'x' : 'o';
    bool alwaysLose = true;
    for (int i = 0; i < 3; i++)
    {
        for (int j = 0; j < 3; j++)
        {
            if (givenBoard[i][j] != '.')
                continue;
            givenBoard[i][j] = turn;
            int answer = ticTacToe(givenBoard);
            givenBoard[i][j] = '.';
            if (answer == -1)
            {
                return 1;
            }
            else if (answer == 0)
                alwaysLose = false;
        }
    }
    return (alwaysLose ? -1 : 0);
}

// Return whether board is full with 'x' and 'o'
bool allDone(char givenBoard[3][3])
{
    int ret = 0;
    for (int i = 0; i < 3; i++)
    {
        for (int j = 0; j < 3; j++)
        {
            if (givenBoard[i][j] == '.')
                ret++;
        }
    }
    return ret == 0;
}

int isFinished(char givenBoard[3][3])
{
    int winner = 0;
    // Check horizontally
    for (int j = 0; j < 3; j++)
    {
        int score = 0;
        for (int k = 0; k < 3; k++)
        {
            if (givenBoard[j][k] == '.')
                break;
            score += givenBoard[j][k] == 'x' ? 1 : -1;
        }
        if (abs(score) == 3)
        {
            winner = score == 3 ? 1 : -1;
            break;
        }
    }
    if (winner == 0)
    {
        // Check vertically
        for (int j = 0; j < 3; j++)
        {
            int score = 0;
            for (int k = 0; k < 3; k++)
            {
                if (givenBoard[k][j] == '.')
                    break;
                score += givenBoard[k][j] == 'x' ? 1 : -1;
            }
            if (abs(score) == 3)
            {
                winner = score == 3 ? 1 : -1;
                break;
            }
        }
    }
    if (winner == 0)
    {
        // Check diagonally
        int score = 0;
        for (int j = 0; j < 3; j++)
        {
            if (givenBoard[j][j] == '.')
                break;
            score += givenBoard[j][j] == 'x' ? 1 : -1;
        }
        if (abs(score) != 3)
        {
            score = 0;
            for (int j = 0; j < 3; j++)
            {
                if (givenBoard[j][2 - j] == '.')
                    break;
                score += givenBoard[j][2 - j] == 'x' ? 1 : -1;
            }
        }
        if (abs(score) == 3)
        {
            winner = score == 3 ? 1 : -1;
        }
    }
    return winner * (isXTurn(givenBoard) ? 1 : -1);
}

// Return whether it's X's turn
bool isXTurn(char givenBoard[3][3])
{
    int x = 0, o = 0;
    for (int i = 0; i < 3; i++)
    {
        for (int j = 0; j < 3; j++)
        {
            if (givenBoard[i][j] == 'x')
                x++;
            else if (givenBoard[i][j] == 'o')
                o++;
        }
    }
    return x == o;
}

void printBoard(int answer, char givenBoard[3][3])
{ // for debug
    cout << endl
         << "******************" << endl;
    for (int i = 0; i < 3; i++)
    {
        for (int j = 0; j < 3; j++)
        {
            cout << givenBoard[i][j] << " ";
        }
        cout << endl;
    }
    cout << endl
         << "Answer : " << answer << endl
         << "***********" << endl;
    return;
}
```  

</div>
</details>  


## 책의 풀이
틱택토는 게임판이 아주 작기 때문에 메모이제이션을 하지 않아도 제한시간에 걸리지 않았다. 하지만 게임판의 상태를 정수로 변환하여 게임판의 상태에 따른 `ticTacToe()`의 결과를 캐시에 저장해두면 더욱 소요 시간이 짧아질 것이다. 게임판의 9개의 자리에는 `o`, `x`, `.` 세 가지 문자가 들어갈 수 있으므로, 9자리 3진수 정수로 변환하면 된다.

## 책의 코드

<details>
<summary>LIS - 책의 코드</summary>
<div markdown="1">

  
```

```
</div>
</details>  
  
## 개선할 점
