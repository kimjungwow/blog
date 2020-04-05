---
title: "CodeJam 후기 :: Qualification Round 2020"
date: 2020-04-06 02:32:00 +0900
categories: 알고리즘_문제풀이 
tags: CodeJam Contest Google
---

# CodeJam Qualification Round 2020
구글에서 진행하는 27시간짜리 Code Jam에 처음으로 참가했다. 40,698명 중 97등으로 마쳤다. 
![Imgur](https://i.imgur.com/OEsTiZt.jpg)  
27시간 중 13시간을 사용해서 5문제를 모두 풀었다. 늦게 일어나서 3시간 30분 늦게 시작한 것이 아쉽다.

## [Vestigium](https://codingcompetitions.withgoogle.com/codejam/round/000000000019fd27/000000000020993c)   

주어진 NxN 행렬의 trace 값과, 중복된 원소를 가진 열, 행의 개수를 구하는 문제이다. 행렬은 1~N의 수들로 구성된다.

## Vestigium : 나의 풀이 : 
행과 열을 각각 `O(n^2)`로 구해도 느리지 않았다. 같은 수가 하나의 행(혹은 열)에 2개 이상 있는지 확인하여 중복된 원소의 유무를 판단했다.


## Vestigium : 나의 코드

<details>
<summary> Vestigium - 나의 코드 </summary>
<div markdown="1">

```
#include <bits/stdc++.h>
using namespace std;
int tc,n,b,rea, ret,rs,cs;
int main() {
    cin>>tc;
    for (int tcc=1;tcc<=tc;tcc++) {
        cin>>n;
        rs=0;
        cs=0;
        ret=0;
        int m[101][101], cache[101];
        for (int r=1;r<=n;r++) {
            memset(cache,false,sizeof cache);
            bool isRow=false;
            for (int c=1;c<=n;c++) {
                cin>>rea;
                m[r][c] = rea;
                if(cache[rea]) {
                    isRow=true;
                } else {
                    cache[rea]=true;
                }
            }
            if(isRow) rs++;
        }
        for (int c=1;c<=n;c++) {
            memset(cache,false,sizeof cache);
            bool isCol=false;
            for (int r=1;r<=n;r++) {
                rea=m[r][c];
                if(cache[rea]) {
                    isCol=true;
                } else {
                    cache[rea]=true;
                }
                if(c==r) ret+=rea;
            }
            if(isCol) cs++;
        }
        

        cout<<"Case #"<<tcc<<": "<<ret<<" "<<rs<<" "<<cs<<endl;
    }

}

```
</div>
</details>  




## [Nesting Depth](https://codingcompetitions.withgoogle.com/codejam/round/000000000019fd27/0000000000209a9f)   
0~9의 숫자로 이루어진 문자열이 주어진다. 각 수 i는 왼쪽에 닫히지 않은 i 개의 `(`를, 오른쪽에 그에 짝지어지는 i개의 `)`를 가지는 식으로 바꾼 뒤 출력해야 한다. 예를 들면, `111000`은 `(111)000`이 되어야 한다.

## Nesting Depth : 나의 풀이 

문자열을 앞부터 훑으며, 현재 열려있는 `(`의 개수를 파악했다. 그 개수가 현재 확인 중인 수 i보다 작다면 부족한만큼 `(`를 추가하고, 크다면 넘치는 만큼 `)`를 추가했다.

## Nesting Depth : 나의 코드

<details>
<summary> Nesting Depth - 나의 코드 </summary>
<div markdown="1">

```
#include <bits/stdc++.h>
using namespace std;
int tc;
int main() {
    cin>>tc;
    for (int tcc=1;tcc<=tc;tcc++) {
        string s, ret;
        cin>>s;
        int curr=0, diff;
        for (auto x: s) {
            if(curr<(x-'0')) {
                diff=(x-'0')-curr;
                string par(diff,'(');
                ret += par;
                curr += diff;

                
            } else if (curr>(x-'0')) {
                diff=curr-(x-'0');
                string par(diff,')');
                ret += par;
                curr -= diff;

            }

            ret.push_back(x);
        }
        string par(curr,')');
        ret += par;

        

        cout<<"Case #"<<tcc<<": "<<ret<<endl;
    }

}

```
</div>
</details>  





## [Parenting Partnering Returns](https://codingcompetitions.withgoogle.com/codejam/round/000000000019fd27/000000000020bdf9)   

각 업무의 시작 시간과 끝 시간이 주어지며, Cameron과 Jamie가 그 일을 나누어 하려 한다. 한 사람이 동시에 2가지 이상의 일을 할 수 없을 때, 두 사람에게 일을 적절히 나눌 수 있는지, 있다면 어떻게 나누면 되는 지를 구해야 한다.

## Parenting Partnering Returns : 나의 풀이 :

Greedy Method를 이용하면 쉽다. 우선 시작 시간을 기준으로 업무들을 정렬한뒤, Cameron과 Jamie에게 일을 배정한다. 각 사람이 가장 마지막에 배정받은 업무의 종료 시간과, 새롭게 배정할 업무의 시작 시간을 비교해서 겹치는지 판단한다. 두 사람 모두에게 배정할 수 없는 업무가 있다면 불가능한 경우이다.

## Parenting Partnering Returns : 나의 코드

<details>
<summary> Parenting Partnering Returns - 나의 코드 </summary>
<div markdown="1">

```
#include <bits/stdc++.h>
using namespace std;
#define fi first
#define se second

int tc,n,b,rea,rs,cs;
int main() {
    cin>>tc;
    for (int tcc=1;tcc<=tc;tcc++) {
        cin>>n;
        vector<vector<int>> works;
        vector<vector<int>> ori_works;
        for (int i=0;i<n;i++) {
            cin>>rs>>cs;
            works.push_back({rs,cs,i});
        }
        ori_works=works;
        sort(works.begin(),works.end());
        int c=-1,j=-1;
        string ret;
        bool impos=false;
        vector<pair<int,char>> corj;
        for (auto x:works) {
            if(c<=x[0]) {
                c=x[1];
                corj.push_back(make_pair(x[2],'C'));
                

            }
            else if (j<=x[0]) {
                j=x[1];
                corj.push_back(make_pair(x[2],'J'));
                
            } else {
                impos=true;
                break;
            }
        }
        if(impos) ret="IMPOSSIBLE";
        else {
            sort(corj.begin(),corj.end());
            for (auto x:corj) {
                ret+=x.se;

            }

        }
        cout<<"Case #"<<tcc<<": "<<ret<<endl;
    }

}
```
</div>
</details>  



## [ESAb ATAd](https://codingcompetitions.withgoogle.com/codejam/round/000000000019fd27/0000000000209a9e)   

0과 1로 이루어진 수열을 맞추는 문제이다. 최대 150번까지 특정 index의 값을 확인할 수 있다. 10번 확인할 때마다 수열이 확률적으로 앞과 뒤가 뒤집히거나, 1의 보수로 바뀌거나, 앞과 뒤가 뒤집히며 1의 보수로 변한다. 수열의 길이는 10 혹은 20 혹은 100이다.

## ESAb ATAd : 나의 풀이 :

**수열의 길이가 N일 때, i번째 수와 N+1-i번째 수를 같이 확인해야 좋다.** 만약 i번째 수와 N+1-i번째 수가 다르다면, 수열이 어떻게 변형되어도 그 둘은 다르다. 마찬가지로 두 수가 같다면, 수열이 어떻게 되어도 그 둘은 같다.  

처음에 1~5, N-4~N번째 수 10개를 확인하면 수열이 변형된다. 이 때, 세 가지 경우로 나눠서 생각할 수 있다.

- 5개의 쌍이 모두 서로 다른 경우 (예 : `1010101010`)
  - 첫 번째 수와 마지막 수가 어떻게 변형되었는지 확인한다. 위 예에서는 원래 1,0이었다.
    - 바뀌지 않는 경우 : 1,0 그대로
    - 뒤집히는 경우 : 0,1로 바뀜
    - 1의 보수가 되는 경우: 0,1로 바뀜
    - 뒤집히며 1의 보수가 되는 경우 : 1,0 그대로
  - 만약 1,0이 0,1로 바뀌었다면, 나머지 수들도 모두 바꾸어줘야 한다. 1,0이 그대로 1,0이라면 나머지 수들도 바꾸지 않아도 된다.
  - 맨 앞의 수만 확인해도 된다. 1개의 수만 확인하면 됨
- 5개의 쌍이 모두 서로 같은 경우 (예 : `1001001001`)
  - 마찬가지로 첫 번째 수와 마지막 수가 어떻게 변형되었는지 확인한다. 위 예에서는 원래 1,1이었다.
    - 1의 보수가 되거나, 뒤집히며 1의 보수가 되는 경우 : 0,0으로 바뀐다. 이 경우는 나머지 수들도 모두 바꿔야 한다.
    - 바뀌지 않거나 뒤집히는 경우 : 그대로 1,1이다. 나머지 수들도 그대로 둬도 된다.
  - 맨 앞의 수만 확인해도 된다. 1개의 수만 확인하면 됨
- 5개의 쌍 중 서로 다른 쌍도 있고 같은 쌍도 있는 경우
  - 우선 같은 쌍을 통해 1의 보수가 되었는지 확인하고, 그 후 다른 쌍을 통해 뒤집혔는지 확인한다.
  - 총 4개의 수를 확인해야 함

이렇게 앞에서 5개의 수, 뒤에서 5개의 수를 알았으면, 그 다음 수들을 확인하면 된다. 5개의 쌍 중 서로 다른 쌍도 있고 같은 쌍도 있는 경우는 수열이 변형된 후 이미 4개의 수를 확인했기 때문에, 6개의 수(3개의 쌍)을 새롭게 확인한다. 나머지 경우에는 1개의 수만을 확인했기 때문에, 8개의 수(4개의 쌍)를 새롭게 확인한다. 짝수여야 의미가 있기 때문에 나머지 한 번은 의미 없는 확인을 한다.

`vector`를 통해 서로 다른 쌍과 같은 쌍을 계속 저장하며, 위의 세 경우로 나누어 판단한다. 사실 앞의 두 경우는 같이 처리해도 된다.

## ESAb ATAd : 나의 코드

<details>
<summary> ESAb ATAd - 나의 코드 </summary>
<div markdown="1">

```
#include <bits/stdc++.h>
using namespace std;
int t,b,s, it, ready;

int singleCheck(int i,vector<int>& v) {
    int s;
    cout<<i<<endl;
    cout.flush();
            
    cin>>s;
    v[i-1]=s;
    return s;
}

bool mir(int i, vector<int>& v) {
    int s,t;
    cout<<i<<endl;
    cout.flush();
            
    cin>>s;
    v[i-1]=s;
    cout<<b+1-i<<endl;
    cout.flush();
    
    cin>>t;
    v[b-i]=t;
    return s==t;
    

}
void repeat_noop(int repeat) {
    for (int i=0;i<repeat;i++) {
        int a=1;
        cout<<a<<endl;
        cout.flush();
        cin>>a;    
    }   
}

void swap(int i, vector<int>& v) {
    int tmp=v[i-1];
    v[i-1]=v[b-i];
    v[b-i]=tmp;
}
int main() {
    
    cin>>t>>b;
    vector<int> v(b,-1);
    
    while(t--) {
        it=0;
        for (int j=1;j<=5;j++) {
            mir(j,v);
        }
        ready=5;
        vector<int> sam, dif;
        for (int j=0;j<5;j++) {
            int i=j;
            if(v[i]==v[b-1-i])
                sam.push_back(i);
            else
                dif.push_back(i);
        }
        while(true) {
            int ch=0;
            if(dif.empty()||sam.empty()) {
                bool changed=false;
                cout<<1<<endl;
                cout.flush();
                cin>>s;
                if(v[0]!=s) {
                    changed=true;
                }
                if(changed) {
                    for(int i=1;i<=ready;i++) {
                        v[i-1]=1-v[i-1];
                        v[b-i]=1-v[b-i];
                    }
                }

                it=ready;
                if(2*it>=b) break;
                else if(2*it+7>b) {
                    for (int j=1;j<=(b-2*it)/2;j++) {
                        int i=it+j;
                        mir(i,v);
                    }    
                    break;
                }
                repeat_noop(1);
                for (int j=1;j<=4;j++) {
                    int i=it+j;
                    mir(i,v);
                    if(v[i-1]==v[b-i])
                        sam.push_back(i-1);
                    else
                        dif.push_back(i-1);
                }
                ready=it+4;
                
            } else { //BOTH
                bool rev=false, com=false;
                int samIdx=sam[0],difIdx=dif[0];
                cout<<samIdx+1<<endl;
                cout.flush();
                cin>>s;
                if(s!=v[samIdx]) com=true;
                cout<<difIdx+1<<endl;
                cout.flush();
                cin>>s;
                if(com) {
                    if(s==v[difIdx]) rev=true;
                } else {
                    if(s!=v[difIdx]) rev=true;
                }
                if(rev) {
                    for (int j=1;j<=ready;j++) {
                        int i=j;
                        swap(i,v);
                    }
                } 
                if(com) {
                    for (int j=1;j<=ready;j++) {
                        int i=j;
                        v[i-1]=1-v[i-1];
                        v[b-i]=1-v[b-i];
                    }
                }
                it=ready;
                
                if(2*it>=b) break;
                else if(2*it+7>b) {
                    for (int j=1;j<=(b-2*it)/2;j++) {
                        int i=it+j;
                        mir(i,v);
                    }    
                    break;
                }

                for (int j=1;j<=4;j++) {
                    int i=it+j;
                    mir(i,v);
                }
                ready=it+4;
            }
        }
        string st;
        for (int i=1;i<=b;i++)
            st += (char)('0'+v[i-1]);
        cout<<st<<endl;
        cout.flush();
        
        char ans;
        cin>>ans;
        if(ans=='N') exit(99);
        

    }
    return 0;

    
}




```
</div>
</details>  



## [Indicium](https://codingcompetitions.withgoogle.com/kickstart/round/000000000019ffc7/00000000001d3f56)   

k와 N이 주어질 때, trace가 k이며 각 행과 열은 중복된 수가 없고, 1~N까지의 수로 이루어진 NxN 행렬을 만들어야 한다.

## Indicium : 나의 풀이 :

예를 들어 4x4 행렬일 때, 대각선의 수가 `(1,2,2,2)`이면 위 조건을 만족하는 행렬을 찾을 수 없다. 1열과 1행에 각각 `2`를 넣어야 하지만 넣을 자리가 없기 때문이다. 따라서, 대각선의 수가 N-1개의 a와 1개의 b로 이루어지면 만들 수 없다.(a!=b) 따라서 `k==N+1||k==N^2-1`이면 불가능하다. 대각선에 넣을 수 들을은 `N-k%N`개의 `k/N`과 `k%N`개의 `(k/N)+1`로 만들되, `k%N==1||k%N==N-1`이라면 더 큰 수 중 하나에 1을 더하고, 더 작은 수 중 하나에서 1을 뺀다.

대각선의 수를 정했으면 나머지를 채우면 된다. 예를 들어 대각선의 수가 `(3,3,3,2,2)`라고 하자. 이 때, `(4,4)`와 `(5,5)`에 `2`가 있으므로, 4열,5열,4행,5행을 무시한 3x3 행렬을 생각하자. 이 3x3행렬의 대각선에는 3이 들어 있고, 이 안에 2를 3개 넣되 같은 행 혹은 열에 들어가지 않도록 해야 한다. 따라서 좌측 상단에서 우측 하단으로 가는 대각선 모양으로 넣으면 된다.(삐져나가면 다시 0열부터 채운다)


## Indicium : 나의 코드

<details>
<summary> Indicium - 나의 코드 </summary>
<div markdown="1">

```
#include <bits/stdc++.h>
using namespace std;
int tc, n, k, rea, ret, rs, cs, covered;
int m[51][51], cache[51], visited[51];

bool cleanPut(int r, int num, vector<bool>& lc) {
    if(r==n) return true;
    bool ret=false;
    for (int c=0;c<n;c++) {
        if(m[r][c]!=0||lc[c]) continue;
        if(ret) break;
        
        lc[c]=true;
        ret |= cleanPut(r+1,num,lc);
        if(ret) {
            m[r][c]=num;
        }
        lc[c]=false;
    }
    return ret;

}

void printM()
{
    for (int i = 0; i < n; i++)
    {
        for (int j = 0; j < n; j++)
        {
            cout << m[i][j] << " ";
        }
        cout << endl;
    }
    return;
}
bool chall(int head, int tail)
{

    int num = m[head][head];
    if (tail - head == n - 1)
    {
        visited[num] = true;
        covered++;
        return true;
    }
    bool nonzero = false, isDone = false;
    int ini = 1;
    int c = 1, r = 0;

    while (ini < n)
    {
        r = 0;
        c = ini;
        if (m[r][c] != 0 || (ini >= head && ini <= tail))
        {
            ini++;
            continue;
        }
        nonzero = false;

        while (r < n)
        {
            if (r == c)
            {
                c++;
            }
            else if (r >= head && r <= tail)
            {
                r = tail + 1;
            }
            else if (c >= head && c <= tail)
            {
                c = tail + 1;
            }
            else if (c == n)
            {
                c = 0;
            }
            else
            {
                if (m[r][c] != 0)
                {
                    nonzero = true;
                    break;
                }
                else
                {
                    r++;
                    c++;
                }
            }
        }
        if (!nonzero)
        {
            r = 0;
            c = ini;
            while (r < n)
            {
                if (r == c)
                {
                    c++;
                }
                else if (r >= head && r <= tail)
                {
                    r = tail + 1;
                }
                else if (c >= head && c <= tail)
                {
                    c = tail + 1;
                }
                else if (c == n)
                {
                    c = 0;
                }
                else
                {
                    m[r][c] = num;
                    r++;
                    c++;
                }
            }
            visited[num] = true;
            isDone = true;
            covered++;
            break;
        }
        else
            ini++;
    }
    return isDone;
}

int main()
{
    cin >> tc;
    for (int tcc = 1; tcc <= tc; tcc++)
    {
        cin >> n >> k;
        memset(visited, false, sizeof visited);
        covered = 0;

        vector<int> v;
        for (int i = 0; i < k % n; i++)
        {
            v.push_back((k / n) + 1);
        }
        for (int i = k % n; i < n; i++)
        {
            v.push_back((k / n));
        }
        if (k == n + 1 || k == n * n - 1)
        {
            cout << "Case #" << tcc << ": IMPOSSIBLE" << endl;
            continue;
        }
        else
        {

            if (k % n == 1)
            {
                if (n == 3)
                {
                    cout << "Case #" << tcc << ": IMPOSSIBLE" << endl;
                    continue;
                }
                v[1] = v[1] + 1;
                v[n - 1] = v[n - 1] - 1;
            }
            else if (k % n == n - 1)
            {
                if (n == 3)
                {
                    cout << "Case #" << tcc << ": IMPOSSIBLE" << endl;
                    continue;
                }
                v[n - 2] = v[n - 2] - 1;
                v[0] = v[0] + 1;
            }
        }

        memset(m, 0, sizeof m);
        for (int i = 0; i < n; i++)
        {
            m[i][i] = v[i];
        }
        int prev = 0;
        bool dd = false;
        for (int i = 1; i < n; i++)
        {
            if (m[prev][prev] != m[i][i])
            {
                if (!chall(prev, i - 1))
                {
                    dd = true;
                    break;
                }
                prev = i;
            }
            if (i == n - 1)
            {
                if (!chall(prev, i))
                {
                    dd = true;
                    break;
                }
            }
        }

        for (int i = 1; i <= n; i++)
        {

            if (visited[i])
                continue;
            int r = 0;
            int c = 1;
            int num = i;
            if (covered == n - 1)
            {
                while (c < n)
                {
                    if (m[r][c] != 0)
                        c++;
                    else
                        break;
                }
                while (r < n)
                {
                    if (r == c)
                    {
                        c++;
                    }
                    else if (c == n)
                    {
                        c = 0;
                    }
                    else if (m[r][c] != 0)
                    {
                        c++;
                    }
                    else
                    {
                        m[r][c] = num;
                        r++;
                        c++;
                    }
                }
                visited[num] = true;
                covered++;
            }
            else
            {
                
                int ini = 1;
                bool nonzero;
                while (ini < n)
                {
                    r = 0;
                    c = ini;
                    if (m[r][c] != 0)
                    {
                        ini++;
                        continue;
                    }
                    nonzero = false;

                    while (r < n)
                    {
                        if (r == c)
                        {
                            c++;
                        }
                        else if (c == n)
                        {
                            c = 0;
                        }
                        else
                        {
                            if (m[r][c] != 0)
                            {
                                nonzero = true;
                                break;
                            }
                            else
                            {
                                r++;
                                c++;
                            }
                        }
                    }
                    if (!nonzero)
                    {
                        r = 0;
                        c = ini;
                        while (r < n)
                        {
                            if (r == c)
                            {
                                c++;
                            }
                            else if (c == n)
                            {
                                c = 0;
                            }
                            else
                            {
                                m[r][c] = num;
                                r++;
                                c++;
                            }
                        }
                        visited[num] = true;
                        covered++;
                        break;
                    }
                    else
                        ini++;

                }
                if(!visited[num]) {

                    vector<bool> lcc(n,false);
                    cleanPut(0,num,lcc);
                }
            }
        }

        if (dd)
            cout << "Case #" << tcc << ": IMPOSSIBLE" << endl;
        else
        {
            cout << "Case #" << tcc << ": POSSIBLE" << endl;
            printM();
        }
    }
}

```
</div>
</details>  
