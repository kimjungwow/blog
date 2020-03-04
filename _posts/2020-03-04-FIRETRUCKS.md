---
title: "Algospot :: FIRETRUCKS (소방차)"
date: 2020-03-04 22:05:00 +0900
categories: 알고리즘_문제풀이 
tags: Algospot Shortest_Path_Problem
---

# 소방차 ( [FIRETRUCKS](https://algospot.com/judge/problem/read/FIRETRUCKS) , 중)

## 문제  
도시의 각 지점들의 위치와 두 지점을 연결하는 도로별 통행 소요 시간이 주어진다. 화재가 발생한 장소와 소방서의 위치가 주어질 때, 각 화재 장소에 소방차가 도착하기까지 걸리는 시간의 합을 구해야 한다.  

소방서에서는 화재가 발생하지 않으며, 각 소방서에는 소방차가 충분히 있다.  

## 나의 풀이
화재 지점과 소방서의 위치가 여러 개가 주어진다. 각 화재 지점마다 가장 가까운 소방서를 골라야 하기 때문에, 각 화재 지점을 출발점으로 하여 여러번 다익스트라 알고리즘을 실행했다.  

하지만 위 방법은 너무 느려서 최적화가 필요했다. 우선순위 큐를 사용하지 않고 매번 반복문을 통해 아직 방문하지 않은 정점 중 가장 가까운 정점을 찾았지만, 역시 느렸다. 이는 `E<=(V^2)/2`이므로 당연한 결과이다.
각 출발점마다 다익스트라 알고리즘을 실행할 때, 출발점에서 모든 소방서로의 최단 거리를 구하면 `while`문을 즉각 종료하고 가장 가까운 소방서와의 최단 거리를 리턴함으로써 코드를 최적화해주었다.


## 나의 코드

<details>
<summary>FIRETRUCKS - 나의 코드 (55m) </summary>
<div markdown="1">

```
#include <bits/stdc++.h>
using namespace std;
int e,v,n,m;
set<pair<int,int>> graph[1002];
vector<int> fs, fire;
int place[1002];
int dijkstra(int start) {
    priority_queue<pair<int,int>> q;
    bool visited[1002];
    memset(visited,false, sizeof visited);
    int dist[1002];
    memset(dist,-1,sizeof dist);
    q.push(make_pair(0,start));
    dist[start]=0;
    int here=1001;
    dist[here]=100002;
    int done=0;
    while(!q.empty()) {
        if(done>=m) break;
        pair<int,int> t=q.top();
        q.pop();
        if(-t.first>dist[t.second]) continue;
        visited[t.second]=true;
        if(place[t.second]==2) done++;
        int curr=-t.first;
        for (auto x:graph[t.second]) {
            if(visited[x.second]) continue;
            if(dist[x.second]==-1||dist[x.second]>curr+x.first) {
                dist[x.second]=curr+x.first;
                q.push(make_pair(-curr-x.first,x.second));
            }
        }
    }
    int ret=100002;
    for (int i=0;i<fs.size();i++) {
        ret=min(ret,dist[fs[i]]);
    }
    return ret;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(NULL);
    int tc;
    cin>>tc;
    while(tc--) {
        memset(place,0,sizeof place);
        fs.clear();
        fire.clear();
        cin>>v>>e>>n>>m;
        for (int i=1;i<=v;i++) graph[i].clear();
        while(e--) {
            int a,b,w;
            cin>>a>>b>>w;
            graph[a].insert(make_pair(w,b));
            graph[b].insert(make_pair(w,a));
        }
        while(n--) {
            int p;
            cin>>p;
            place[p]=1;
            fire.push_back(p);
        }
        int temp_m=m;
        while(temp_m--) {
            int p;
            cin>>p;
            place[p]=2;
            fs.push_back(p);
        }
        int ret=0;
        for (auto x:fire) {
            ret+=dijkstra(x);
        }
        cout<<ret<<endl;
    }
}

```
</div>
</details>  


## 책의 풀이  
역시 각 소방서에서 혹은 각 화재 지점에서 다익스트라 알고리즘을 실행하면 너무 느리다. **책에서는 그래프에 가상의 시작점을 추가한 뒤, 이 가상의 점과 모든 소방서를 가중치가 0인 간선으로 이어주었다. 그 후 이 가상의 점을 출발점으로 하여 모든 화재지점까지의 최단 거리를 구했다. 한 번의 다익스트라 알고리즘만으로 모든 소방서에서 동시에 다익스트라 알고리즘을 수행하는 꼴이다.**

## 책의 코드

<details>
<summary>FIRETRUCKS - 책의 코드  </summary>
<div markdown="1">

```
#include <bits/stdc++.h>
using namespace std;
int e,v,n,m;
set<pair<int,int>> graph[1002];
vector<int> fs, fire;
int place[1002];
int dijkstra(int start) {
    priority_queue<pair<int,int>> q;
    bool visited[1002];
    memset(visited,false, sizeof visited);
    int dist[1002];
    memset(dist,-1,sizeof dist);
    q.push(make_pair(0,start));
    dist[start]=0;
    while(!q.empty()) {
        pair<int,int> t=q.top();
        q.pop();
        if(-t.first>dist[t.second]) continue;
        visited[t.second]=true;
        int curr=-t.first;
        for (auto x:graph[t.second]) {
            if(visited[x.second]) continue;
            if(dist[x.second]==-1||dist[x.second]>curr+x.first) {
                dist[x.second]=curr+x.first;
                q.push(make_pair(-curr-x.first,x.second));
            }
        }
    }
    int ret=0;
    for (int i=0;i<fire.size();i++) {
        ret+=dist[fire[i]];
    }
    return ret;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(NULL);
    int tc;
    cin>>tc;
    while(tc--) {
        memset(place,0,sizeof place);
        fs.clear();
        fire.clear();
        cin>>v>>e>>n>>m;
        for (int i=0;i<=v;i++) graph[i].clear();
        while(e--) {
            int a,b,w;
            cin>>a>>b>>w;
            graph[a].insert(make_pair(w,b));
            graph[b].insert(make_pair(w,a));
        }
        while(n--) {
            int p;
            cin>>p;
            place[p]=1;
            fire.push_back(p);
        }
        int temp_m=m;
        while(temp_m--) {
            int p;
            cin>>p;
            fs.push_back(p);
            graph[0].insert(make_pair(0,p));
            graph[p].insert(make_pair(0,0));
        }
        cout<<dijkstra(0)<<endl;
    }
}

```
</div>
</details>  


## 기억할 점
책의 코드는 수행시간이 52ms이고, 내 코드는 수행시간이 1860ms이다.  
**책에서 가상의 시작점을 추가한 뒤 모든 소방서와 가중치가 0인 도로로 이어준 아이디어를 기억해야겠다.**
