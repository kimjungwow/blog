---
title: "Algospot :: DICTIONARY (고대어 사전)"
date: 2020-01-27 01:31:00 -0400
categories: 알고리즘_문제풀이 
tags: Algospot DFS
---

# 고대어 사전 ( [DICTIONARY](https://algospot.com/judge/problem/read/DICTIONARY) , 하)

## 문제
사전 순서로 주어지는 단어들을 보고 알파벳들의 순서를 출력해야 한다.  
예를 들어, `ba, aa` 순서로 단어들이 주어지면, 알파벳 `b`가 알파벳 `a`보다 순서가 빠르다.  
## 나의 풀이
나는 그래프를 그리는 동시에 사이클이 있는지 확인하려 해서 코드가 복잡해졌다. 또한 소요 시간도 길었다.  
`dfs()`를 하나씩 실행하는 `dfsAll()`에서도 사이클이 있는지 확인하는 과정이 있어서 소요 시간이 길었다.  
또한 `dfs()`가 종료될 때 현재 정점을 추가한 뒤 마지막에 뒤집는 방식이 아닌, 해당 정점으로 들어가는 간선이 없는 정점들을 하나씩 지우는 방식을 사용했기 때문에 시간초과가 떴다.

## 나의 코드

<details>
<summary>DICTIONARY - 나의 코드</summary>
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
int n, testcases;
vector<string> words;
bool order[27][27];
bool taken[27];
bool invalid;
void makeGraph();
void dfs(int index);
void dfsall();
string answer;

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(NULL);
    
    cin >> testcases;
    for (int x=0;x<testcases;x++) {
        cin>>n;
        words.clear();
        for (int c=0;c<n;c++) {
            string temp;
            cin>>temp;
            words.push_back(temp);
        }
        memset(order, false, sizeof order);
        memset(taken, false, sizeof taken);
        invalid=false;
        makeGraph();
        
        if(invalid) {
            cout<<"INVALID HYPOTHESIS"<<endl;
        } else {
            dfsall();

        }
    }
    return 0;
}

void makeGraph() {
    for (int i=1;i<n;i++) {
        for (int j=0;j<=words[i].length();j++) {
            if(j==words[i].length()) {
                invalid=true;
                return;
            } else if (words[i-1].length()==j) {
                break;
            } else if (words[i-1][j]==words[i][j]) {
                continue;
            } else {
                order[words[i-1][j]-'a'][words[i][j]-'a']=true;
                break;
            }
        }
    }
    return;
}

void dfs(int index) {
    if(taken[index])
        return;
    taken[index]=true;
    for (int i=0;i<26;i++) {
        if(order[index][i])
            dfs(i);
    }
    answer.push_back((char)('a'+index));
}
void dfsall() {
    string last;
    last="";
    for (int i=0;i<26;i++)
        {
            bool check=false;
            for (int j=0;j<26;j++) {
                if(order[j][i]) {
                    check=true;
                    break;
                }
            }
            if(check) continue;
            answer="";
            dfs(i);
            reverse(answer.begin(), answer.end());
            // cout << answer;
            last = last+answer;
        }
    if(last.length()<26) {
        cout<<"INVALID HYPOTHESIS"<<endl;
    } else {
    cout<<last<<endl;
    }
    return;
}
```
</div>
</details>  
## 책의 풀이
우선 `makeGraph()`라는 함수로 사이클의 유무를 신경쓰지 않고 의존성 그래프를 그렸다. 그 후, 각 단어를 바로 앞의 단어와만 비교했다. 예를 들어, A, B, C가 주어지면 A와 B, B와 C만 비교하고 A와 C는 비교하지 않았다.  
**`dfsAll()`이 실행된 결과를 뒤집어서 얻은 결과에서, 역방향 간선이 있는지 확인함으로써 사이클의 유무를 판단하는 아이디어가 간결했다. 내 풀이 처럼 2중 for문을 사용해 root 노드들에 대해서만 먼저 실행할 필요 없이, 1중 for 문을 사용한 뒤 뒤집어주면 된다.**  여러가지 반례를 신경쓰지 않고 일반적으로 해결한 좋은 풀이였다.  
## 책의 코드

<details>
<summary>DICTIONARY - 책의 코드</summary>
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
int n;
string words[1001];
bool vertices[27][27];
bool visited[27];
string dfs(int node);
bool isLoop;
int visit;
void makeGraph();
int main()
{
    int testcases;

    ios::sync_with_stdio(false);
    cin.tie(NULL);
    cin >> testcases;

    for (int c = 0; c < testcases; c++)
    {
        cin >> n;
        for (int w = 0; w < n; w++)
        {
            cin >> words[w];
        }
        memset(vertices, false, sizeof vertices);
        memset(visited, false, sizeof visited);
        makeGraph();
        bool invalid = false;
        bool existLoop = false;
        string temp = "";
        for (int a = 0; a < 26; a++)
        {
            if (!visited[a])
                temp = temp + dfs(a);
        }
        reverse(temp.begin(), temp.end());
        for (int p = 1; p < temp.length(); p++)
        {
            if (vertices[temp[p] - 'a'][temp[p - 1] - 'a'])
                temp = "INVALID HYPOTHESIS";
        }
        cout << temp << endl;
    }

    return 0;
}

void makeGraph()
{
    for (int w = 1; w < n; w++)
    {
        for (int i = 0; i < words[w].size(); i++)
        {
            if (words[w - 1].size() <= i)
                break;
            if (words[w - 1][i] != words[w][i])
            {
                vertices[words[w - 1][i] - 'a'][words[w][i] - 'a'] = true;
                break;
            }
        }
    }
}

string dfs(int node)
{

    visited[node] = true;
    string ans;
    char ret = 'a';

    for (int a = 0; a < 26; a++)
    {
        if (vertices[node][a] && (!visited[a]))
        {
            ans = ans + (dfs(a));
            vertices[node][a] = false;
        }
    }
    ans.push_back(ret + node);
    return ans;
}
```
</div>
</details>  
  
## 기억할 점
`dfs()`가 종료될 때 현재 정점을 추가한 뒤 마지막에 뒤집는 방식을 사용해야 한다.
