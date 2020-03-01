---
title: "VSCode에서 bits/stdc++.h 사용하기"
date: 2020-03-01 12:37:00 -0400
categories: 알고리즘_공부 
tags:
---

## bits/stdc++.h
코드포스의 솔루션을 보니 헤더파일은 `#include <bits/stdc++.h>`만 포함하고 있었다. `stdc++.h`를 이용하면 헤더 파일을 추가할 시간을 줄일 수 있다. 하지만 불필요한 헤더파일들도 컴파일하기 때문에 시간이 오래 걸린다.


## VSCode에서 bits/stdc++.h 사용하기
나는 `gcc version 6.3.0`을 사용하며, `g++ -c -o $1.o $1.cpp \ g++ -o $1 $1.o -lm`을 이용해 컴파일 한다. gcc를 사용하면 `bits/stdc++.h`를 사용할 수 있다.  
우선 `bits/stdc++.h`를 [다운로드](https://raw.githubusercontent.com/gcc-mirror/gcc/master/libstdc%2B%2B-v3/include/precompiled/stdc%2B%2B.h)한 뒤, 이를 `C:\MinGW\lib\gcc\mingw32\6.3.0\include\c++\bits`에 넣으면 `bits/stdc++.h`를 사용할 수 있다.