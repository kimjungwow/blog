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