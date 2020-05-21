---
title: "BAEKJOON :: Social Distancing (18877)"
date: 2020-05-21 23:44:00 +0900
categories: 알고리즘_문제풀이 
tags: BAEKJOON
---

# Social Distancing ( [18877](https://www.acmicpc.net/problem/18877) )

## 문제

소가 몇마리 있는지 나타내는 N과 mutually-disjoint intervals의 개수 M이 주어진다. 소는 양 끝점을 포함한 M개의 intervals 위에만 올라갈 수 있을 때, 각 소 사이의 최단 거리 D의 최댓값을 찾아야 한다.

## 나의 풀이

M개의 mutually-disjoint intervals중 가장 작은 인터벌의 작은 수를 A, 가장 큰 인터벌의 큰 수를 B라고 하자. 그 때 D는 `(B-A)/(n-1)` 이하이다. A부터 B까지 겹치지도 떨어지지도 않게 M개의 intervals들이 있는 경우에 D는 `(B-A)/(n-1)`이다. 만약 D가 이 값보다 크다면, 양 끝에 있는 소끼리의 거리가 B-A보다 커지기 때문에 모순이다.

따라서 `D=(B-A)/(n-1)`인 경우부터 시작해 D에 대하여 Binary Search를 시도했다. 처음에 `d=(B-A)/(n-1), diff=d, finish=false`에 대하여 `BinarySearch(d,diff,finish)`를 실행한다. `BinarySearch()`에서는 우선 N마리의 소를 d 간격으로 놓는 것이 가능한지 판단한다. M개의 인터벌 중 가장 작은 인터벌의 가장 작은 지점부터 시작하여, 인터벌 위의 지점 중 d이상 떨어졌지만 앞의 소와 가장 가까운 점을 계속해서 선택해 나간다. 그렇게 N마리의 소를 놓을 수 있다면, 더 큰 d로 시도하기 위해 `BinarySearch(d+diff/2,diff/2,finish)`를 시도한다. N마리의 소를 놓을 수 없다면, 더 작은 d로 시도하기 위해 `BinarySearch(d-diff/2,diff/2,finish)`를 시도한다. 마지막에는 `diff==1`인 경우가 연속해서 두 번 있을 수 있는데, 이 때 `diff/2==0`이어서 그 특수한 경우를 다루기 위해 `finish`변수를 사용한다. `diff==1`이면 `diff/2`로 빼는 대신 다시 `diff=1`을 사용하고, 연속해서 두 번 `diff==1`이면 `finish==true`로 설정하는 식이다.

2중 while문이 있지만, 처음에 `left=N-1`로 설정되고 while문의 조건문은 `left>0`을 포함하고 있기 때문에 하나의 `binarySearch()` 호출은 `O(N)`이다. `diff`를 반씩 줄여나가므로 전체 시간복잡도는 `O(NlgN)`이다.

## 나의 코드

<details>
<summary>Social Distancing - 나의 코드 (소요시간 : 48m)</summary>
<div markdown="1">
  
```
#include <bits/stdc++.h>
using namespace std;
int n,m,a,b,maxD,tryD, ans;
vector<pair<int,int>> grass;
int binarySearch(int d, int diff, bool finish) {
    int left=n-1, last=grass.front().first;
    if(d<=0||diff<=0||d>maxD) return 0;
    vector<pair<int,int>>::iterator it=grass.begin();
    while(left>0 && it!=grass.end()) {
        while(left>0&&last+d <= (it->second)) {
            left--;
            last = max(last+d, (it->first));
        }
        it++;
    }
    bool putfinish;
    int newdiff;
    if(diff==1) {
        newdiff=1;
        putfinish=true;
    } else {
        newdiff=diff/2;
        putfinish=false;
    }
    if (finish) {return 0;}
    else if(left==0) {
        ans=max(ans,d);
        binarySearch(d+newdiff,newdiff,putfinish);
        return d;
    }
    else return binarySearch(d-newdiff,newdiff,putfinish);
}
int main() {
    cin>>n>>m;
    
    while(m--){
        cin>>a>>b;
        grass.push_back({a,b});
    }
    sort(grass.begin(),grass.end());
    maxD=(grass.back().second-grass.front().first+1)/(n-1);
    tryD=maxD;
    ans=0;
    binarySearch(maxD,maxD,false);
    cout<<ans;
    
    return 0;
}

```
</div>
</details>

## 좋은 풀이

```
while(lo<hi) {
    mid=(lo+hi)/2;
    if(ok(mid)) {
        answer=mid;
        lo=mid-1;
    } else {
        hi=mid+1;

    }
}
```

`diff`를 계속 파라미터로 전달하기 보다는, main함수 안에서만 `lo`, `hi`, `mid`를 이용해 해결할 수 있다. 특히, `lo`와 `hi`를 수정할 때 1을 빼고 더해줌으로써 더욱 최적화 할 수 있다.



## 아쉬운 점
- `binarySearch()`에서 처음 `last`변수를 `grass.front().first`이 아닌 `0` 으로 설정해서 디버깅하는데 오래 걸렸다.