---
title: "Research :: MemoryHierarchyDesign"
date: 2020-04-17 14:45:00 +0900
categories: Study
tags: Study Architecture
---
H&P textbook Chapter 2를 읽고 정리했다.  

# 2.1 Introduction

- mh(memory hierarchy) : locality + 메모리는 작아야 더 빠르다는 사실에 근거
- server는 제일 아래에 disk쓰지만 mobile device는 flash memory
- mh에서 AMAT보다 power consumption도 중요해짐. 특히 PMD(personal mobile device) 에서.
- both miss rate and miss per Instruction don't factor in miss rate. 그래서 AMAT이용. 하지만 AMAT에서 miss penalty는 miss동안 다른 I 실행하는 등의 방법으로 tolerate가능하므로, 역시 execution time을 정확히 반영하지는 못함
- six basic cache operation
  - Increase block size: reduce compulsory misses and the number of tags. increase miss penalty, conflcit misses, capacity misses.
  - bigger cache : decreases capacity misses. longer hit time, higher cost and power.
  - Increase associativity : reduce conflict misses, increases hit time and power
  - Multilevel cache to reduce miss penalty : fast first level, large second level
  - Read misses를 write보다 우선시하여 miss penalty 줄임
  - page offset을 사용하여, avoid address translation during indexing of the cache to reduce hit time
    - L1 cache의 구조나 size에 제약이 생기지만, critical path에서 TLB를 뺄 수 있는 장점떄문에 사용
- Server와 Desktop에서 processor architecture는 비슷. workload와 performance가 다름.
  - Desktop은 latency from memory hierarchy에 집중
  - Server는 memory bandwidth도 신경써야 함

# 2.2 Ten Advanced Optimizations of Cache Performance
- AMAT에 있는 hit time, miss rate, miss penalty에 추가로 cache bandwidth, power consumption을 고려할 것
- *Reducing the hit time* : power consumption 개선
  - Small and simple first-level cache 
    - Critical timing path in a cache hit
      - Addressing the tag memory using the index portion of the address
      - Comparing the read tag value to the address
      - Setting the multiplexor to choose the correct data item if the cache is set associative
    - Direct-mapped cache에서는 tag check하며 바로 데이터 전송 가능함. associativity 작을 수록 더 적은 line access하기 때문에 power 적게 듦
    - 기술이 발전했음에도 L1 cache를 키우면 clock rate가 커지기 때문에, L1 cache는 작은 사이즈 유지 중. cache size 키우기보다는 associativity 늘리기를 선택. 하지만 associativity를 늘리면 address aliases가 제거되어 hit time이 길어질 수도 있음. 또한 cache size/associativity 정할 때 power consumption도 고려함 (associativity 크면 여러 개의 tag를 읽어야 해서 power 많이 필요)
    - 대부분의 processor의 hit time이 2 cycle이상이어서 더 긴 hit time이 영향이 크지 않음. 또한 TLB를 critical path에서 제외하기 위해 대부분의 L1 cache는 virtually indexed -> cache size는 `associativit X page size` 이하. 또한 multi-threading 사용시 conflict misses가 증가하므로 associativity 키우는 것이 적절

  - way-prediction 
    - 미리 하나의 block을 예측 : 한 번의 tag 비교만 필요. 혹시 틀렸으면 다음 cycle에 다른 blocks 비교. 2-way/4-way에서 주로 사용. processor가 너무 빠르면 one cycle stall로 유지하기도 어려움 (clock cycle 짧아서?) -> 유지 못하면 way prediction penalty 너무 큼.
    - way selection : way prediction bit을 이용해 어느 block을 access할지 예측하기 : 맞으면 power 줄이지만, 틀리면 단순히 tab 비교가 아닌 access가 반복되기 때문에 시간 손해 큼. 또한 cache access를 pipeline 하기 어려워짐.


- *Increasing cache bandwidth* : power consumption 개선하는 정도 불분명
  - Pipelined caches
    - cache accesses를 pipeline함. hit time이 여러 clock cycles되며 길어졌지만 (miss penalty 커짐), high bandwidth and fast clock time 제공 가능.
  - multibanked caches
    - cache가 여러 개의 cache인것처럼 사용. 각 address가 mod N의 bank에 할당되도록 하는 sequential interleaving이 대표적
  - nonblocking caches
    - *nonblocking cache*/*lockup-free cache* : miss 동안에도 cache hit 제공 (hit under miss) - miss 중에도 다른 일 하며 miss penalty 줄임. 또한 여러 miss를 overlap하여 "hit under multiple misses" or "miss under miss" 가능하면 miss penalty 더 줄일 수 있음.
    - 상황에 따라 associativity를 늘리는 것보다 miss under miss 가능케 하는 것이 더 성능 향상 시키기도 함
    - nonblocking caches에서는 miss시 stall 발생하는 것이 아니므로 성능 계산이 어려움. miss penalty는 miss 횟수가 아니고 processor가 stall한 것의 non-overlapped time이다.
- *Reducing the miss penalty* : power consumption 개선에 영향 적음
  - Critical word first
    - critical word first(block 전체가 아닌, 필요한 word를 우선 요청) and early restart(missed word 도착하자마자 processor에 전달하여 execution 진행)
    - block size가 커야 성능향상 큼.
    - rest of block 중 새로운 것이 또 요청되면 miss penalty 계산 어려워짐.
  - Merging write buffers
    - write buffer는 하나의 word를 여러번 쓰는것보다 multiword를 한번 쓰는것이 더 효율적이어서 성능개선함. write buffer에 쓰기 전에, 하나의 block으로 합칠 수 있는지 확인함.
- *Reducing the miss rate*
  - Compiler optimizations : power consumption 개선, HW 사용 X
    - loop interchange : loop 순서 바꿔 row wise로 하여 cahe block use
    - blocking : matrix multiplication에서 block으로 계산 : cache misses 줄임
- *Reducing the miss penalry or miss rate via parallelism* : power consumption 악화.
  - Hardware prefetching
    - miss 시, 그 뒤 것을 stream buffer에 넣음. memory bandwidth를 최대한 사용하며 이득 보는 것
  - Compiler prefetching 
