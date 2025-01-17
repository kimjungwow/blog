---
title: "Algospot :: Linear Data Structure"
date: 2020-03-11 21:31:00 +0900
categories: 알고리즘_공부
tags: Algospot Linear_Data_Struture
---

# Linear Data Structure (선형 자료 구조) : AOJ 18단원  

## 동적 배열 (Dynamic array)

동적 배열은 선언할 때 배열의 크기를 지정해서 그 이상의 자료를 넣을 수 없는 배열과 다르지만, 내부적으로는 배열을 이용한다. 따라서 동적배열은 배열과 마찬가지로 원소들은 메모리의 연속된 위치에 저장되며(캐시의 효율성과 직결), 주어진 위치의 원소를 반환하거나 변경하는 데는 `O(1)`이 걸린다. 하지만 배열과는 다르게 배열의 크기를 변경하는 `resize()`연산이 걸리며 (`O(n)`소요 ) 주어진 원소를 배열의 맨 끝에 추가함으로써 크기를 1 늘리는 `append()`연산이 가능하다 (`O(1)` 소요).

동적 배열은 `new`등의 연산으로 할당받은 고정 길이 배열인 동적으로 할당 받은 배열(dynamically allocated)을 사용한다. 크기가 바뀌어야 할 때는 단순하게 새 배열을 동적으로 할당받고 기존 원소들을 복사하며, 새 배열을 참조하도록 바꿔치기 한다. 따라서 동적 배열 클래스는 `int size; ElementType* array`를 저장하고 있다.  

새 배열을 할당받고 기존 자료를 복사하는데는 배열의 크기에 비례하는 시간이 걸리므로 `resize()` 연산을 `O(N)`에 구현할 수 있다. 하지만 `append()`를 호출될 때마다 `resize()`를 호출하면 `append()`도 `O(N)`이 되어버린다. 따라서 메모리를 할당받을 때 배열의 크기가 커질 때를 대비해서 여유분의 메모리를 미리 할당받아둔다. 그리고 배열이 이미 할당한 메모리에 꽉 찼을 떄 더 큰 메모리를 할당받아 배열의 원소를 그 쪽으로 전부 옮긴다.  

이미 할당받은 메모리의 크기를 배열의 용량(capacity), 그리고 실제 원소의 수를 배열의 크기(size)라고 한다. 만약 현재 배열의 크기가 용량보다 작다면 `append()`연산은 size를 1 늘리고 그 위치에 새 값을 할당하는 것으로 간단히 구현할 수 있다. 문제는 미리 할당해 둔 메모리가 꽉 찼을 때다. 더 큰 새 배열을 동적으로 할당받고 새 배열에 기존 배열의 내용을 모두 복사한 다음 배열에 대한 포인터를 바꿔치기해야 한다. 기존의 용량 N보다 M만큼 큰 새 배열을 만든다면, `O(N+M)`이 소요된다.

### 동적 배열의 재할당 전략

`append()`를 수행하는데 가끔 `O(N+M)`이고, 대부분은 `O(1)`이다. 이 때, `O(N+M)`이 얼마나 자주인지에 따라 `append()`의 시간 복잡도가 정해진다. 현재 가진 원소에 비례해서 여유분을 확보하면, 선형 시간에 `append()`를 구현할 수 있게 된다. 예를 들어 동적 배열의 용량을 2배씩 늘려나간다면, 마지막 k번째 복사에야 우리가 원하는 N개를 모두 복사하게 된다. 이 때, k-1번째 복사에서는 N/2개를, k-2번째 복사에서는 N/4개를 복사하며, (1/2+1/4+1/8 ...)는 1보다 작기 때문에, 전체 복사의 수는 2N보다 작다. 따라서 `append()`는 선형시간에 구현가능하다.

### 표준 라이브러리의 동적 배열 구현

`append()` 연산을 여러 번 수행할 때 배열의 최종 크기가 얼마인지 미리 짐작할 수 있다면, 동적 배열의 용량을 미리 `reserve()`로 늘려놓음으로써 최적화할 수 있다.

## 연결 리스트

배열에서 원소를 삭제하려면 해당 위치 뒤에 있는 원소들을 하나씩 앞칸으로 옮겨야하기 때문에 평균적으로 원소의 개수에 선형 시비례하는 시간이 걸립니다. 하지만 연결 리스트(Linked List)는 `struct ListNode {int element; ListNode *prev, *next}`꼴로써 삭제는 `O(1)`이다. 연결 리스트에서 i번째 노드를 찾아내려면 리스트의 머리(head) 혹은 꼬리(tail)에서부터 찾아나가야 한다. 따라서 i번째 노드를 찾는데는 리스트의 길이에 선형 비례한다. 하지만 삽입, 삭제는 삽입/삭제할 노드와 이전/이후 노드만 바꾸면 되므로 `O(1)`이다.

### 연결 리스트 응용 연산 (C++ : STL의 list)

#### 잘라붙이기 연산

C++에서는 `splice()`를 통해 리스트를 다른 리스트의 중간에 집어 넣을 수 있다. 하지만 이 때 몇 개의 원소가 추가되는지 모르므로, 리스트의 길이가 궁금하면 선형 시간의 반복문을 수행하며 직접 세주어야 한다.

#### 삭제했던 원소 돌려놓기

양방향 연결 리스트의 잘 알려지지 않은 장점으로, 한 번 삭제했던 원소를 제자리에 쉽게 돌려놓을 수 있다. 노드를 삭제할 때 이전/이후 노드의 포인터만 바꾸지, 삭제한 노드의 포인터들은 자기가 원래 있던 위치를 그대로 기억하고 있다. 이를 이용해서 삭제했던 노드를 다시 복구할 수 있지만, 이전노드나 이후 노드 또한 삭제된 상태에서 수행하면 리스트를 망가뜨리게 된다. 따라서 삭제한 순서의 반대로 복구가 이루어져야 한다. 이는 사용자 인터페이스나 조합 탐색 (답의 한 조각을 만들고, 현재의 상태를 갱신한 뒤 나머지를 재귀 호출로 해결하며, 그 뒤에 다시 원래의 상태로 돌리는 경우)에서 유용하게 쓰일 수 있다. 

## 동적 배열과 연결 리스트의 비교

동적 배열과 연결 리스트의 가장 큰 차이점은 삭제/삽입, 그리고 임의의 원소에 접근하는데 걸리는 시간이다. 삽입/삭제할 일이 없거나 배열의 끝에서만 하면 될 경우에는 동적 배열을 사용해야 좋다. 
