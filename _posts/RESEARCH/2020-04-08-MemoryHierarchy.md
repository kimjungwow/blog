---
title: "Research :: MemoryHierarchy"
date: 2020-04-08 19:45:00 +0900
categories: Study
tags: Study Architecture
---
H&P textbook Appendix B를 읽고 정리했다.  

# B.1 Introduction

- cache miss
  - latency : block의 첫 word 받는데 걸리는 시간
  - bandwidth : 나머지 내용들을 받는데 걸리는 시간
- out of order execution시, cache miss발생하면 더 늦게 들어온 I 중 관련 없는 것은 stall 안 되고 계속 실행 가능.
- address space는 일정 개수의 blocks로 이루어진 page로 구성된다. 필요한 page가 cache와 main memory에 없으면 palt 발생하여, disk로부터 전체 page를 가져온다. 이는 아주 오래걸려서 stall되지 않으며, SW가 담당한다.
- cache와 main memory의 관계는 main memory와 disk의 관계와 비슷하다. 또한 cache보다 register가 작고 빠르다
- memory hierarchy는 작은 메모리가 빠르고, locality 때문에 성능을 개선한다
- cache 성능 계산시 사실 read와 write의 비율을 따로 생각해야하지만, 편의상 그렇지 않는다.
- miss rate를 `miss/memory_reference` 혹은 `miss/I`로 가능. 
  - 후자는 independent to HW implementation(일부 HW는 실제 commit된 I의 2배가까이를 fetch함->전자로 계산하면 miss rate 낮아짐)
  - 후자의 단점 : architect dependent(I당 memory access 평균횟수가 다름)
- address
  - address tag는 모든 block 확인
  - index는 set 정함
  - offset은 data with block (block 내에서 위치인듯)
- Pseudo LRU
  - 하나의 set이 모두 가득차면, 각 block이 accessed 될 때 turn on bit
  - 만약 모든 block이 turned on이면, exception을 통해 turn off bit
  - evict시 꺼져있는 bit 선택
- memory access는 대부분 read임
  - processor는 read는 기다리고 write는 기다리지 않으므로, read를 최적화하는 것이 중요함
  - cache 의 block 는 bit 비교함과 동시에 읽기 시작할 수있어서 최적화하기 좋음. 또한 정해자 바이트가 없어서 미리 읽어두는 방식으로 최적화 가능
  - write는 비교 끝난 뒤 시작 가능하며, 정해진 바이트만 수정 가능해 최적화 어렵

- writing to cache
  - *Write-through* : cache의 block에 적을 때 low-level cache에도 적음
    - 구현이 쉽고 data coherence 유지 쉬움
  - *Write-back* : cache에만 적고, replaced 될 때 main memory에 적음
    - dirty bit을 이용해 데이터가 수정되었는지 확인.
    - 여러 번 수정되어도 한 번만 메인 메모리에 적어서 memory bandwidth, power 절약
  - Write-through 기다리는 대신 (write stall) 우선 *write buffer*에 적은 뒤, 버퍼의 내용을 write함과 동시에 다른 일도 함. 물론 write buffer가 있어도 write stall이 생길 수 있다.
  - write miss
    - *Write allocate* : write miss 시 block을 allocated받는다. 이어서 read/write시 cache hit. *Write-back*이 주로 사용
    - *No-write allocate* : write miss 시 cache에 적지 않고 main memory에 직접 적는다. 이어서 read/write시 cache miss (read한 이후에는 cache hit). *Write-through*가 주로 사용. (여러 번의 write시 어차피 main memory에 적어야 하기 때문에)

- Intel Operaton 예시
  - Block
    - Block address : 34 bits
        - Tag : 25 bits (valid bit 포함)
        - Index : 9 bits
    - Block offset : 6 bits
  - 64bytes word로 구성된 set이 512=2^9개 있다. set이 512개여서 Index는 9bits이지만, word가 64bytes이므로 정확히 원하는 8bytes를 얻기 위해 추가적으로 3비트가 필요하다. 따라서 Index 9 bits에 더해 block offset의 6 bits 중 3 bits를 사용해 원하는 8bytes에 접근한다.
  - victim이 수정되어있다면 (dirty bit ON), *write buffer*를 사용하듯 *victim buffer*에 넣은 뒤 다른 작업과 병렬적으로 진행하며 main memory에 수정된 값을 적음

- Instruction cache와 Data cache를 분리해놓아서 miss rate 줄임

# B.2 Cache Performance

- Instruction count와 miss rate가 HW와 독립적이라고 memory hierarchy의 performance를 판단하는 근거로 사용하는 것은 부적절하다. **Average Memory Access Time (AMAT)**로 판단하는게 더 낫다. `AMAT = Hit time + Miss rate X Miss penalty`

## Average Memory Access Time and Processor Performance

- cache miss로 인한 AMAT로 processor 성능을 판단하는 것이 적절한가?
  - 모든 memory stall이 cache miss 떄문이 아니다 (memory 사용하는 I/O device 때문일수도 있다)
  - In-of-Order processor에서는 AMAT로 성능 판단 가능. (cahe miss로 인한 stall이 성능에 영향이 크기 때문) : 하지만 AMAT가 작다고 꼭 CPU execution time이 작지 않음
- CPI가 작을 수록, clock rate가 클 수록 CPU execution time 에서 cache영향이 크다

## Miss Penalty and Out-of-Order Execution Processors
  
- Out-of-Order에서는 cache miss가 좀 많아도, miss를 일부 hide하면 성능이 더 좋을 수도 있음.
- Out-of-Order에서는 AMAT개선이 CPU time개선에 직결되지 않을 수 있어서 복잡함

# B.3 Six Basic Cache Optimizations

- Optimizing cache
  - Reducing the *miss rate*
    - larger block size, larger cache size, and higher associativity
  - Reducing the *miss penalty*
    - multilevel caches and giving reads priority over writes
  - Reducing the *time to hit in the cache*
    - avoidng address translation when indexing the cache

- Three categories of miss
  - Compulsory
    - 첫 access는 cache hit일 수 없음. = *cold-start misses* = *first-reference misses* (cache size에 관련 X)
  - Capacity
    - cache이 프로그램 실행 중 필요한 block들을 모두 담기에 작은 경우, capacity miss 발생 (cache 전체 사이즈에 영향)
  - Conflict
    - n-way set-associative에서 특정 set에 n개 초과 request 발생시 conflict miss 발생 (n에 영향)
    - full associative면 conflict miss가 없지만 HW에서 구현하려면 비싸고, 성능도 안 좋을 수 있음
  - Coherence miss도 있지만 아직 다루지 않음

- *thrash* : upper-level cache가 너무 작으면 low-level memory만 있는 것과 같은 성능이 날 수도 있고, 오히려 miss overhead 때문에 성능이 더 안 좋을 수도 있음
- block size를 늘리면 compulsory miss는 줄어들 지라도, 다른 miss가 늘어날 수 있음
- cache size를 늘리면 capacity miss는 줄어들 지라도, 특정 set에 할당되는 reference가 많아져 conflict miss 늘어날 수도 있음
- miss rate를 줄이면 hit time/miss penalty 증가할 수 있으므로 균형을 맞추어야 함

## First Optimization : Larger Block Size to Reduce Miss Rate

- Larger Block size reduces compulsory misses (thanks to spatial locality)
- increase conflict misses 
- increase capacity misses when cache is small
- The increase in miss penalty may outweigh the decrease in miss rate
- block size가 cache size에 비해 너무 커지면 miss rate가 오히려 증가
- high latency, high bandwidth : bigger block size

## Second Optimization: Larger Caches  to  Reduce  Miss  Rate

- Larger cache
  - lower miss rate
  - higher bit time, cost, power

## Third Optimization:  Higher  Associativity to  Reduce  Miss  Rate

- Two rules of thumb
  - 8-way associative is almost effective as full associative
  - direct cache with size N has same miss rate with 2-way cache with size n/2

- higher associativity
  - bigger hit time
  - lower miss rate
  - cache size 커지면 큰 hit time으로 인해 higher associativity의 AMAT 커짐

## Fourth Optimization: Multilevel Caches to Reduce Miss Penalty

- 프로세서는 점점 빨라져서 memory stall을 해결하는 것이 중요해짐
  - cache를 더 빠르게 만들어 메모리도 프로세서와 비슷한 속도로 만들거나
  - cache를 더 크게 만들어서 프로세서와 메모리의 차이를 극복하거나
- first-level cache는 빠른 프로세서의 clock cycle에 부합하는 작은 크기로, second-level cache는 충분히 크게 만드는 multilevel caches
- *Local miss rate* : 단순히 cache misses를 해당 cache에서의 memory accesses로 나눔
- *Global miss rate* : cache misses를 processor에 의해 시작된 memory accesses로 나눔. `L2의 Global miss rate = Miss_rate_L1 X Miss_rate_L2`