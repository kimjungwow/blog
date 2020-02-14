---
title: "Algospot :: WORDCHAIN (단어 제한 끝말잇기)"
date: 2020-02-14 20:01:00 -0400
categories: 알고리즘_문제풀이 
tags: Algospot DFS
---

# 단어 제한 끝말잇기 ( [WORDCHAIN](https://algospot.com/judge/problem/read/WORDCHAIN) , 하)

## 문제  

주어지는 단어들을 중복 없이 모두 이용해서 끝말잇기를 해야 한다. 끝말잇기가 불가능한 경우, `IMPOSSIBLE`을 출력한다. 

## 나의 풀이
알파벳들을 정점으로 생각하고, 주어지는 단어들을 간선으로 생각했다. 예를 들어 `dragon`이라는 단어가 주어지면, 정점 d에서 정점 n으로 향하는 간선이 1개 주어지는 것이다. 즉, 이 문제는 **방향 그래프에서 오일러 회로 또는 오일러 트레일이 존재하는지를 묻는 문제이다.**  

방향 그래프이며, 두 정점 사이에 여러 개의 간선이 있을 수 있으므로 `int vertices[26][26]`에 두 정점 사이의 간선의 개수를 저장하였다. 또한, `inV[26]`과 `outV[26]`에 각 정점으로 들어가거나 각 정점에서 나가는 간선의 개수를 미리 계산해두었다. 방향 그래프에서 오일러 회로가 존재하려면 모든 정점에서 나가는 간선의 개수와 들어오는 간선의 개수가 같아야 한다. 방향 그래프에서 오일러 트레일이 존재하려면 나가는 간선의 개수가 들어가는 간선의 개수보다 1개 많은 정점이 딱 1개, 들어가는 간선의 개수가 나가는 간선의 개수보다 1개 많은 정점이 딱 1개 존재하며, 나머지 정점들은 모두 나가는 간선의 개수와 들어오는 간선의 개수가 같아야한다. 위 두 경우를 제외하면 끝말잇기가 불가능하다.  

하지만 계속 오답으로 채점되어서 책을 참고해서 해결했다.

## 책의 풀이
나는 방향 그래프를 정확히 그린 뒤, `findEuler()`를 각 정점에서 호출하며 오일러 회로 혹은 오일러 트레일을 계산하려 했다.  

하지만 책에서는 `findEuler()`는 한 번만 호출하면서, 간선으로 연결된 다른 정점이 확인될 시, `while(vertices[index][i]>0)`로 그 정점으로 향하는 모든 간선을 소모한 뒤 다른 정점을 탐색했다. `while`문을 이용하여 모든 간선을 소모하였기 때문에, 한 번의 `findEuler()`만 호출한 뒤 경로를 뒤집어 줌으로써 하나의 컴포넌트의 모든 간선을 지나는 오일러 트레일을 얻을 수 있었다.  

또한 한 번의 `findEuler()`를 통해 얻은 경로의 길이가 적절하지 않은 경우, 그래프에 컴포넌트가 여러 개가 있기 때문에 끝말잇기가 불가능함을 알 수 있다. 예를 들어, 단어가 `aa, cc`로 주어지면, 각 컴포넌트에는 오일러 회로가 존재하지만 두 단어를 모두 사용하며 끝말잇기를 진행할 수 없기 때문에, `IMPOSSIBLE`을 출력해야 한다.  

그리고 나는 오일러 트레일을 찾는 경우, 하나의 간선을 적절하게 추가해준 뒤 오일러 회로를 찾는 문제로 바꿔서 풀려고 했다.  

하지만 추가해준 간선에 해당하는 단어가 존재하지 않기 때문에, 추가해준 간선이 오일러 회로의 중간에 있는 경우 제대로 된 끝말잇기가 아닐 수 있다. 따라서, 나가는 간선이 들어오는 간선보다 하나 더 많은 정점에서 `findEuler()`를 호출함으로써 해당 정점에서 시작하는 오일러 트레일을 찾아야 한다.  

`findEuler()`의 실행 횟수는 단어의 개수 `n`에 비례하며, 알파벳들의 개수 `A`가 정점의 개수이므로 시간 복잡도는 `O(nA)`이다.  

## 책의 코드

<details>
<summary>WORDCHAIN - 책의 코드</summary>
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
int n;
vector<vector<int>> vertices;
vector<string> words;
vector<int> inV;
vector<int> outV;

vector<int> answer;
vector<int> real;
bool taken[100];
bool cycle=false;
void findEuler(int index);
int inOdd=0, start=0, dest=0,outOdd=0;
int main()
{
    int testcases;

    ios::sync_with_stdio(false);
    cin.tie(NULL);
    cin >> testcases;
    for (int t=0;t<testcases;t++) {
        memset(taken,false,sizeof taken);
        answer.clear();
        cin>>n;
        vertices = vector<vector<int>>(26, vector<int>(26, 0));
        inV = vector<int>(26,0);
        outV = vector<int>(26,0);
        words.clear();
        inOdd=0; start=0; dest=0; outOdd=0;
        real.clear();
        for (int w=0;w<n;w++) {
            string tempWord;
            cin>>tempWord;
            words.push_back(tempWord);
            vertices[tempWord[0]-'a'][tempWord[tempWord.size()-1]-'a']+=1;
            outV[tempWord[0]-'a']+=1;
            inV[tempWord[tempWord.size()-1]-'a']+=1;
            dest=start=tempWord[0]-'a';
        }
        
        for (int i=0;i<26;i++) {
            if(inV[i]>outV[i]) {
                inOdd++;
                dest=i;
            }
            if(outV[i]>inV[i]) {
                outOdd++;
                start=i;
            }
        }
        if(!((inOdd==0&&outOdd==0)||(inOdd==1&&outOdd==1))) cout<<"IMPOSSIBLE"<<endl;
        else {
            findEuler(start);

            for (int o=0;o<answer.size();o++) {
                real.push_back(answer[answer.size()-1-o]);
            }
            answer.clear();
            if(real.size()==n+1) {
                for (int x=1;x<=real.size()-1;x++)
                {
                    for (int r=0;r<n;r++) {
                        if(taken[r]||words[r][0]-'a'!=real[x-1]||words[r][words[r].length()-1]-'a'!=real[x]) continue;
                        cout<<words[r]<<" ";
                        taken[r]=true;
                        break;
                    }
                }
                cout<<endl;
            } else cout<<"IMPOSSIBLE"<<endl;
        } 
    }
}



void findEuler(int index) {
    if(start!=dest&&index==dest) {
        while(vertices[dest][start]>0)
        {
            outV[dest]-=1;
            inV[start]-=1;
            vertices[dest][start]-=1;
            findEuler(start);
        }
    }
    for (int i=0;i<26;i++) {
        while(vertices[index][i]>0)
        {
            outV[index]-=1;
            inV[i]-=1;
            vertices[index][i]-=1;
            findEuler(i);
        }
    }
    answer.push_back(index);
}
  

```
</div>
</details>  

## 기타
- 각 단어를 하나의 정점으로 생각하며, 한 단어의 마지막 글자가 다른 단어의 첫 글자와 같으면 간선으로 연결하는 그래프로 생각할 수 있다. 이 경우에는 모든 정점을 지나는 `해밀토니안 경로(Hamiltonian path)`를 구해야 한다. 하지만 해밀토니안 경로를 빠르게 찾는 방법은 아직 고안되지 않았다.  

## 기억할 점
- `findEuler()`가 종료될 때 현재 정점을 추가한 뒤 마지막에 뒤집는 방식을 사용해야 한다.
- 오일러 회로를 찾을 때,  `while(vertices[index][i]>0)`를 이용하여 각 정점으로 향하는 모든 간선을 소모한 뒤 다른 정점을 탐색하면 `findEuler()`을 한 번만 호출해도 한 컴포넌트의 오일러 회로를 얻을 수 있다.