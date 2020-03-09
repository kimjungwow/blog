---
title: "Codeforces 후기 :: Round #620 Div.2"
date: 2020-02-29 14:17:00 -0400
categories: 알고리즘_문제풀이 
tags: Codeforces Contest
---

# Codeforces Round #620 Div.2  
코드포스에서 진행하는 2시간짜리 대회에 처음으로 참가했다.  
![Standing](https://i.imgur.com/ByJxleF.jpg)
![Score](https://imgur.com/IPCSn0W.jpg)  
2시간동안 총 7문제 중 4문제를 풀었고, 첫 Rating은 1616(Blue)을 받았다.

## [Problem A](https://codeforces.com/contest/1304/problem/A)   
각각 x와 y의 위치에 있는 두 마리의 토끼가, 서로를 향해 1초마다 a, b씩 이동한다. 이 때 두 마리의 토끼가 같은 시간에 같은 지점에서 만날 수 있는지, 있다면 그 때 까지 몇 초가 걸리는지를 출력해야 한다.

## Problem A : 나의 풀이 : 3m
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
쉬운 문제이지만 출제자의 의도를 확인하기 위해 Solution을 확인했다. 기본 아이디어는 내 생각과 같았다. 
- 솔루션에서는 `#include <bits/stdc++.h>` 하나만으로 `cin`과  `cout`을 사용했다. 나도 불필요한 코드 복붙을 줄여야 겠다.  
- `testcase` 개수가 주어질 때, `for(int i=0;i<testcase;i++)`대신 `while(testcase--)`를 쓰면 더 빠르다.
- 태그
  - math
  - *800

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

## [Problem B](https://codeforces.com/contest/1304/problem/B)   
`pop`, `noon`처럼 뒤집어도 똑같은 문자열은 `palindrome`이라고 한다. m글자로 이루어진 n개의 단어가 주어질 때, 주어진 단어들을 이어서 만들 수 있는 가장 긴 `palindrome`의 길이와 그 `palindrome`을 출력해야 한다.  

## Problem B : 나의 풀이: 11m
n 개의 단어를 이어서 가장 긴 `palindrome`을 만들 때, 사용될 수 있는 단어는 아래의 조건 중 하나를 만족해야 한다.  
- n이 홀수이며, 정 가운데에 있는 단어는 `palindrome`이어야 한다.  
- 앞에서부터 k번째 단어는 뒤에서부터 k번째 단어와 서로 뒤집으면 같은 단어여야 한다.  
따라서 각 단어가 `palindrome`인지, 그리고 뒤집으면 다른 단어와 같아지는지 먼저 판단했다. 그 후, `palindrome`인 단어가 있다면 그 단어를 가운데에 두고 (없다면 빈 문자열을 가운데에 두고), 뒤집으면 같은 단어들을 앞과 뒤에 붙여나가며 가장 긴 `palindrome`을 만들었다.


## Problem B : 나의 코드

<details>
<summary> Problem B - 나의 코드 </summary>
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
int n,m;
using namespace std;
bool itself[100];
int palin[100];
int main()
{
    ios::sync_with_stdio(false);
    cin.tie(NULL);
    
    
    cin>>n>>m;
    vector<string> vec;
    memset(palin,-1,sizeof palin);
    memset(itself,false,sizeof itself);
    for (int i=0;i<n;i++) {
        string temp;
        cin>>temp;
        vec.push_back(temp);
    }
    int selv=-1;
    for (int i=0;i<n;i++) {
        string reversed = vec[i];
        reverse(reversed.begin(),reversed.end());
        if(vec[i].compare(reversed)==0) {
            itself[i]=true;
            selv=i;
            continue;
        }
        for (int j=i+1;j<n;j++) {
            if(vec[j].compare(reversed)==0) {
                palin[i]=j;
                palin[j]=i;
                break;
            }
        }
    }
    string ans="";
    if(selv!=-1) ans=vec[selv];
    for (int i=0;i<n;i++) {
        if(palin[i]==-1) continue;
        else {
            ans = vec[i]+ans+vec[palin[i]];
            palin[palin[i]]=-1;
            palin[i]=-1;
        }
    }
    cout<<ans.length()<<"\n"<<ans;
}

```
</div>
</details>  

## Problem B : 출제자의 풀이
- `vector`를 사용한 기존의 코드는 `O(n^2*m)`이지만, `std::set`과 같이 탐색에 `O(logn)`이 걸리는 데이터 구조를 사용하면 `O(nmlogn)`이 된다.  
- 단어 자체가 `palindrome`이면 `string mid`로 지정하고, 이외의 경우는 `string left, right`에 붙여나가며 `std::set dict`에서 지우는 아이디어가 흥미로웠다.  
- 태그
  - brute force
  - constructive algorithms
  - greedy
  - implementation
  - strings
  - *1100

## Problem B : 출제자의 코드

<details>
<summary> Problem B - Solution </summary>
<div markdown="1">

```
#include <bits/stdc++.h>
using namespace std;

const int MAX_N = 100;
string s[MAX_N];

int main()
{
	set<string> dict;
	int n, m, i;
	cin >> n >> m;
	for (i = 0; i < n; i++)
	{
		cin >> s[i];
		dict.insert(s[i]);
	}
	vector<string> left, right;
	string mid;
	for (i = 0; i < n; i++)
	{
		string t = s[i];
		reverse(t.begin(), t.end());
		if (t == s[i])
			mid = t;
		else if (dict.find(t) != dict.end())
		{
			left.push_back(s[i]);
			right.push_back(t);
			dict.erase(s[i]);
			dict.erase(t);
		}
	}
	cout << left.size() * m * 2 + mid.size() << endl;
	for (string x : left)
		cout << x;
	cout << mid;
	reverse(right.begin(), right.end());
	for (string x : right)
		cout << x;
	cout << endl;
}
```
</div>
</details>  


## [Problem C](https://codeforces.com/contest/1304/problem/C)  
손님들의 수, 레스토랑의 초기 온도, 레스토랑에 각 손님들의 도착 시간, 선호하는 최저 온도와 최고 온도가 주어진다. 에어컨을 조작하여 온도를 분당 1도씩 올리거나 내리거나 유지되도록 할 수 있다. 모든 손님들이 도착했을 때 레스토랑의 온도가 선호하는 온도 내여서 만족할 수 있는지를 출력해야 한다.

## Problem C : 나의 풀이: 15m
`int maxT, minT`를 초기 온도로 설정한다. 이 후, k번째 손님과 k-1번째 손님의 도착 시간 차이가 `t`라면, `maxT+=t; minT-=t;`를 한다. 기존 온도의 범위에서, t초동안 최고 온도는 maxT까지 올라갈 수 있고 최저 온도는 minT까지 내려갈 수 있다. 이 후, k번째 손님의 선호 온도 범위와 `[minT,maxT]`가 겹치는지 확인한다. 한 번이라도 겹치지 않으면 모두를 만족시키는데 실패한 것이다. 겹치는 경우에는 k번째 손님의 선호 온도 범위가 `l[k], h[k]`로 주어질 때, `maxT=min(maxT,h[k]); minT=max(minT,l[k]);`를 한다. k번째 손님을 만족시킬 수 있는 범위로 `[minT,maxT]`를 바꾸는 것이다.

## Problem C : 나의 코드

<details>
<summary> Problem C - 나의 코드 </summary>
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
int n,m;
using namespace std;
int t[100],h[100],l[100];
int main()
{
    ios::sync_with_stdio(false);
    cin.tie(NULL);
    
    int testcases;
    cin>>testcases;
    for (int g=0;g<testcases;g++) {
        cin>>n>>m;
        int maxT=m,minT=m;
        for (int i=0;i<n;i++) {
            cin>> t[i] >>l[i]>>h[i];
        }
        bool pos=true;
        for (int i=0;i<n;i++) {
            int gone=t[i];
            if(i>0) gone=t[i]-t[i-1];
            
            maxT+=gone;
            minT-=gone;
            
            if(maxT<l[i]||minT>h[i]) {
                pos=false;
                break;
            } else {
                maxT=min(maxT,h[i]);
                minT=max(minT,l[i]);
            }
        }
        cout<<(pos?"YES":"NO")<<"\n";
    }
}

```
</div>
</details>  

## Problem C : 출제자의 풀이
나의 풀이와 동일하다.
- 태그
  - dp
  - greedy
  - implementation
  - sortings
  - two pointers
  - *1500

## Problem C : 출제자의 코드

<details>
<summary> Problem C - Solution </summary>
<div markdown="1">

```
#include <bits/stdc++.h>
using namespace std;

const int MAX_N = 100;

int t[MAX_N], lo[MAX_N], hi[MAX_N];

int main()
{
	int tc;
	cin >> tc;
	while (tc--)
	{
		int n, m, i;
		cin >> n >> m;
		for (i = 0; i < n; i++)
			cin >> t[i] >> lo[i] >> hi[i];
		int prev = 0;
		int mn = m, mx = m;
		bool flag = true;
		for (i = 0; i < n; i++)
		{
			mx += t[i] - prev;
			mn -= t[i] - prev;
			if (mx < lo[i] || mn > hi[i])
			{
				flag = false;
				break;
			}
			mx = min(mx, hi[i]);
			mn = max(mn, lo[i]);
			prev = t[i];
		}
		if (flag)
			cout << "YES\n";
		else
			cout << "NO\n";
	}
}
```
</div>
</details>  


## [Problem D](https://codeforces.com/contest/1304/problem/D)  
`<`와 `>`로 이루어진 문자열이 주어진다. 문자열의 길이가 n-1이면, 1부터 n까지의 수로로 이루졌으며, k번째 수와 k+1번째 수의 관계는 문자열의 k번째 부등호와 같은 수열들을 생각하자. 그 중 LIS (longest increasing subsequence)가 가장 긴 경우와 가장 짧은 경우의 수열을 출력해야 한다.


## Problem D : 나의 풀이 : 47m
LIS의 길이가 가장 긴 경우는, 문자열에 `<`가 k개 있다면 LIS의 길이를 k+1개로 만드려 했다. 문자열을 앞에서부터 훑으며, `small=0; big=n;`으로 시작했을 때 `<`가 나올 때마다 `small++;`를 넣고, 그렇지 않으면 `big--`을 넣었다.  
LIS가 가장 짧은 경우에는, 우선 문자열을 뒤에서 부터 훑었다. `<`가 연속적으로 나온 경우, 연속된 `<`의 가장 첫 `<` 앞에 `small++`를 넣고, 나머지는 1씩 큰 수를 넣었다. 이후 다시 앞으로 훑으며 `<`를 찾았다. 그 과정이 끝나면 `>`에 해당하는 부분만 비어있는데, 이는 반대로 앞에서부터 뒤로 훑으며 `>`를 찾았다. 연속된 `>` 중 첫 `>`의 앞에 `big--`를 넣고, 이후에는 1씩 작은 수를 넣었다. 지금 생각하면 내 코드이 `putInTwo()`는 필요없는 함수다.

## Problem D : 나의 코드

<details>
<summary> Problem D - 나의 코드 </summary>
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
int n;
string word;
void putIn(int index, int& num, vector<int>& LIS);
void putInTwo(int index, int& num, vector<int>& LIS);
int main()
{
    ios::sync_with_stdio(false);
    cin.tie(NULL);
    
    int testcases;
    cin>>testcases;

    for (int t=0;t<testcases;t++) {
        cin>>n>>word; 
        int small=1,big=n;
        vector<int> bigLIS=vector<int>(n,0);
        vector<int> smallLIS=vector<int>(n,0);
        
        for (int g=1;g<n;g++) {
            if(word[g-1]=='<') {
                bigLIS[g-1]=small++;
            } else {
                bigLIS[g-1]=big--;
            }
        }

        bigLIS[n-1]=big;

        small--;
        int temp=big;
        big++;
        small=1;big=n;
        for (int g=n-1;g>0;g--) {
            if(word[g-1]=='<'&&smallLIS[g-1]==0) {
                putIn(g-1,small,smallLIS);
            } 
        }
        for (int g=1;g<n;g++) {
            if(word[g-1]=='>'&&smallLIS[g-1]==0) {
                smallLIS[g-1]=big--;
            }
        }
        smallLIS[n-1]=big;
        for (int i=0;i<n;i++)
            cout<<smallLIS[i]<<" ";
        cout<<"\n";
        for (int i=0;i<n;i++)
            cout<<bigLIS[i]<<" ";
        cout<<"\n";
    }
    
}

void putInTwo(int index, int& num, vector<int>& LIS) {
    if(index<n-1&&word[index+1]=='>') {
        putInTwo(index+1,num,LIS);
    }
    LIS[index]=num--;
}

void putIn(int index, int& num, vector<int>& LIS) {
    if(index>0&&word[index-1]=='<') {
        putIn(index-1,num,LIS);
    }
    LIS[index]=num++;
}

```
</div>
</details>  

## Problem D : 출제자의 풀이
- 출제자 역시 연속된 `<` 혹은 연속된 `>`에 주목했다.  
  - 가장 짧은 LIS를 갖는 경우는, 연속된 `<` 개수의 최댓값+1로 LIS의 길이를 만들고자 했다. 이를 위해, 앞에서 부터 훑으며 `>`가 나오면, 그 앞의 모든 `<`들에 대하여 n에서 1씩 줄어드는 수를 채웠다. 즉, 각각은 연속되었지만 서로 분리된 `<`들의 모임이 있다면, 오른쪽의 연속된 `<`에 있는 수들이 왼쪽의 연속된 `<`에 있는 수들보다 작도록 했다.  
  - 마찬가지로 가장 긴 LIS를 갖는 경우는, 앞에서부터 훑다가 `<`가 나오면 앞에서부터 0부터 1씩 늘어나며 `>`를 채웠다. 즉 두 개의 분리된 연속 `>`중 왼쪽의 `>` 모임에 있는 수들이 오른쪽의 `>` 모임에 있는 수보다 작도록 했다.  
- 인상적인 코드
  - `for (i = 0; i < n; i++) cout << ans[i] << (i == n - 1 ? '\n' : ' ');`
  - 배열의 각 값 사이에 빈칸과 `\n`을 한 번에 표현했다.


- 태그
  - constructive algorithms
  - graphs
  - greedy
  - two pointers
  - *1800

## Problem D : 출제자의 코드

<details>
<summary> Problem D - Solution </summary>
<div markdown="1">

```
#include <bits/stdc++.h>
using namespace std;

const int MAX_N = 200000;
int ans[MAX_N + 5];

int main()
{
	int tc;
	cin >> tc;
	while (tc--)
	{
		int n, i, j;
		string s;
		cin >> n >> s;

		int num = n, last = 0;
		for (i = 0; i < n; i++)
		{
			if (i == n - 1 || s[i] == '>')
			{
				for (j = i; j >= last; j--)
					ans[j] = num--;
				last = i + 1;
			}
		}
		for (i = 0; i < n; i++)
			cout << ans[i] << (i == n - 1 ? '\n' : ' ');

		num = 1, last = 0;
		for (i = 0; i < n; i++)
		{
			if (i == n - 1 || s[i] == '<')
			{
				for (j = i; j >= last; j--)
					ans[j] = num++;
				last = i + 1;
			}
		}
		for (i = 0; i < n; i++)
			cout << ans[i] << (i == n - 1 ? '\n' : ' ');
	}
}
```
</div>
</details>  


## 기타
Problem E는 풀다가 시간이 부족해서 포기했고, F는 손도 못 댔다.