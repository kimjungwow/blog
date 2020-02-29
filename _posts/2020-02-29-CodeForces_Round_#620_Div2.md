---
title: "Codeforces 후기 :: Round #620 Div.2"
date: 2020-02-29 14:17:00 -0400
categories: 알고리즘_문제풀이 
tags: Codeforces Contest
---

# Codeforces Round #620 Div.2  
코드포스에서 진행하는 2시간짜리 대회에 처음으로 참가했다.  
[Standing](https://imgur.com/ByJxleF)
[Score](https://imgur.com/IPCSn0W)  
2시간동안 총 7문제 중 4문제를 풀었고, 첫 Rating은 1616(Blue)을 받았다.

## [Problem A](https://codeforces.com/contest/1304/problem/A)   
각각 x와 y의 위치에 있는 두 마리의 토끼가, 서로를 향해 1초마다 a, b씩 이동한다. 이 때 두 마리의 토끼가 같은 시간에 같은 지점에서 만날 수 있는지, 있다면 그 때 까지 몇 초가 걸리는지를 출력해야 한다.

## Problem A : 나의 풀이
`(y-x)%(a+b)==0`이면 같은 시간에 같은 지점에서 만날 수 있으며, 그 때까지 걸리는 시간은 `(y-x)/(a+b)`초이다.  

## Problem A : 나의 코드

<details>
<summary> Problem A - 나의 코드 </summary>
<div markdown="1">

```
#include <stdio.h>
#include <string.h>
#include <iostream>
#include <utility>
#include <vector>
#include <algorithm>
#include <string>
#define fi first
#define se second

#ifdef _MSC_VER
#define _CRT_SCURE_NO_WARNINGS
#endif

using namespace std;
int main()
{
    ios::sync_with_stdio(false);
    cin.tie(NULL);
    
    int testcases;
    cin>>testcases;
    for (int t=0;t<testcases;t++) {
        int x,y,a,b;
        cin>>x>>y>>a>>b;
        int diff=y-x;
        if(diff%(a+b)==0) cout<<diff/(a+b)<<"\n";
        else cout<<"-1"<<"\n";
    }
}

```
</div>
</details>  

## Problem A : 출제자의 풀이
쉬운 문제이지만 출제자의 의도를 확인하기 위해 Solution을 확인했다.  
- 솔루션에서는 `#include <bits/stdc++.h>` 하나만으로 `cin`과  `cout`을 사용했다. 나도 불필요한 코드 복붙을 줄여야 겠다.  
- `testcase` 개수가 주어질 때, `for(int i=0;i<testcase;i++)`대신 `while(testcase--)`를 쓰면 더 빠르다.

## Problem A : 출제자의 코드

<details>
<summary> Problem A - Solution </summary>
<div markdown="1">

```
#include <bits/stdc++.h>
using namespace std;

int main()
{
	int tc;
	cin >> tc;
	while (tc--)
	{
		int x, y, a, b;
		cin >> x >> y >> a >> b;
		cout << ((y - x) % (a + b) == 0 ? (y - x) / (a + b) : -1) << endl;
	}
}
```
</div>
</details>  











## Problem A : 나의 풀이
 

## Problem A : 나의 코드

<details>
<summary> Problem A - 나의 코드 </summary>
<div markdown="1">

```


```
</div>
</details>  

## Problem A : 출제자의 풀이


## Problem A : 출제자의 코드

<details>
<summary> Problem A - Solution </summary>
<div markdown="1">

```

```
</div>
</details>  

