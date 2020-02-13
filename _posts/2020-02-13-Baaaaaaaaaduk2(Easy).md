---
title: "BAEKJOON :: Baaaaaaaaaduk2(easy) (16988)"
date: 2020-02-11 17:31:00 -0400
categories: 알고리즘_문제풀이 
tags: BAEKJOON
---

# Baaaaaaaaaduk2(easy) ( [16988](https://www.acmicpc.net/problem/16988) )

## 문제
Baaaaaaaaaduk2에서는 기존의 바둑과 다르게, 각 플레이어가 한 번에 두 개의 바둑돌을 둘 수 있다. 현재의 바둑판이 주어질 때, 2개의 바둑돌을 둬서 죽일 수 있는 상대 돌의 최대 갯수를 구해야 한다.  

## 나의 풀이
DFS처럼 내가 바둑돌 2개를 둘 수 있는 경우를 모두 따져보았다. 그 후, 각 경우마다 내가 뺏을 수 있는 상대의 바둑돌을 셌다. 이 때 죽게되는 상대의 바둑돌들이 속한 컴포넌트들은, 내가 놓는 2개의 바둑돌과 맞닿아있다. 따라서, 내가 놓은 바둑돌 2개의 상하좌우에서만 죽일 수 있는 바둑돌이 있는지 확인했다.  
DFS로 바둑돌 2개를 둘 수 있는 경우를 따지는데 O(nm)이고, 각 경우마다 죽일 수 있는 바둑돌을 판단하는데 O(nm)이다. 따라서 O((nm)^2)이지만, 실제로는 내가 죽일 수 있는 바둑돌이 바둑판에 많이 존재하지 않아서 실행시간이 8ms밖에 걸리지 않았다.  
  
## 나의 코드

<details>
<summary>Baaaaaaaaaduk2(easy) - 나의 코드 (소요시간 : 45m)</summary>
<div markdown="1">

  
```
#include <stdio.h>
#include <vector>
#include <algorithm>
#include <cmath>
#include <iostream>
#include <cstdint>
#include <cstring>

#ifdef _MSC_VER
#define _CRT_SCURE_NO_WARNINGS
#endif
using namespace std;
int dx[4] = {1,0,-1,0}, dy[4]={0,-1,0,1};
int n,m, score, name=3, big=-1;
void dfs(int x, int y);
bool isEmpty(int x, int y);
int board[20][20];
int tempBoard[20][20];
int scores[400];
bool components[400];
int main()
{
    ios::sync_with_stdio(false);
    cin.tie(NULL);
    cin>>n>>m;
    memset(board,0,sizeof board);
    
    for (int y=0;y<n;y++) {
        for (int x=0;x<m;x++) {
            cin>>board[y][x];
        }
    }
    
    for (int i=0;i<n*m;i++) {
        int y1=i/m, x1=i-y1*m;
        if(board[y1][x1]!=0) continue;
        board[y1][x1]=1;
        for (int j=i+1;j<n*m;j++) {
            name=3;
            int y2=j/m, x2=j-y2*m;
            if(board[y2][x2]!=0) continue;
            memset(tempBoard,0,sizeof tempBoard);
            memset(components,true,sizeof components);
            memset(scores,0,sizeof scores);
            board[y2][x2]=1;
            memcpy(tempBoard,board,sizeof board);
            for (int t=0;t<4;t++)
            {
                
                dfs(x2+dx[t],y2+dy[t]);
                name++;
                
                dfs(x1+dx[t],y1+dy[t]);
                name++;
            }
            memcpy(board,tempBoard,sizeof board);
            board[y2][x2]=0;
            int sum=0;
            for (int l=3;l<name;l++) {
                if(components[l]) sum+=scores[l];

            }
            if(big<sum) big=sum;

        }
        board[y1][x1]=0;
    }
    cout<<big<<endl;
    return 0;
}

void dfs(int x, int y) {
    if(x<0||y<0||x>=m||y>=n) return;
    if(board[y][x]!=2) return;
    if(isEmpty(x-1,y)||isEmpty(x+1,y)||isEmpty(x,y-1)||isEmpty(x,y+1))
    {
        components[name]=false;
        return;
    }
    scores[name]+=1;
    board[y][x]=name;
    for (int i=0;i<4;i++)
        dfs(x+dx[i],y+dy[i]);
}

bool isEmpty(int x, int y) {
    bool ans;
    if(x<0||y<0||x>=m||y>=n) ans=false;
    else ans= board[y][x]==0;
    return ans;
}

```
</div>
</details>