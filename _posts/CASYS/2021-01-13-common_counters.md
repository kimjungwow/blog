---
title: "CASYS :: Common Counters"
date: 2021-01-13 16:51:00 +0900
categories: casys
tags: 논문
---

## Common Counters: Compressed Encryption Counters for Secure GPU Memory

Context
- Hardware-based trusted execution은 secure cloud computing에 중요함
- 하지만 hardware-based memory protection은 CPU에만 최적화되었고, GPU에는 최적화되지 않음

Gap
- GPU execution environment를 고려한 이전 논문들
  - Gravition은 GPU가 trusted memory 사용하는 경우만 다룸 -> 이 논문에서는 untrusted memory도 고려함 -> GPU의 메모리를 보호하는 방법을 고려
  - HIX는 direct physical attack을 고려하지 않음


Innovation
- GPU의 write pattern은 CPU의 패턴과 다르기 때문에, common counter를 이용하면 counter cache miss rate를 줄일 수 있음
  - MAC보다 counter cache miss가 성능에 더 영향이 큼
  - common counter는 counter cache miss rate만을 향상시킴
- GPU write pattern
  - 처음에 한 번만 쓰거나 (initial data transfer from CPU)
  - the number of writes tends to be uniform
  - NVBit을 이용해 관측
- *Common Counter Status MAP (CCSM)*
  - tells whether the requested address can be served by a common counter
  - *initial write once* | kernel execution 마다 수정됨
  - Number of distinct counters도 수가 매우 적음
    - CCSM을 GPU-wide로 사용하며, segment당 4 bits 사용 -> invalid + 15개의 valid common counters
    - common counter set은 per-context
    - CCSM 속 entry의 값은 index to the common counter set for the context
  - scanning overhead를 줄이기 위해 updated memory region을 기록함 (1 bit per 2MB region)
- LLC miss handling flow
  - data와 counter를 동시에 request함
  - counter request시 CCSM Cache에서 hit인지 확인 -> miss면 DRAM (hidden memory) 확인
  - valid index인지 확인
    - 맞다면 common counter 이용
    - 아니라면 counter cache를 다시 확인
  - counter등을 이용해 얻은 OTP로 DRAM에서 가져온 encrypted data를 decrypt
- 결과
  - common counter는 counter cache에만 집중하므로, efficient data MAC encryption technique (Syngery)와 함께 사용할 시 성능이 더욱 개선됨
  - 보통 Ratio of LLC misses served by common counters가 높을 수록 더 성능 개선 
  - 보통 common counter는 counter cache size에 영향 적게 받음 -> 대부분의 miss가 CCSM에서 처리되어서
  - Scanning overhead는 무시할만함


질문
- kernel execution이란? + 왜 kernel execution마다 CCSM이 update 되도록 했는지
- 코드는 공개되는지
- 아이디어가 어디서 시작했는지
  - GPU에서 성능이 잘 안 나와서? 아니면 요즘 트렌드여서?
  - GPU의 write pattern을 확인할 생각을 어떻게 하게 되었는지
