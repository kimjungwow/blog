---
title: "CASYS :: JaewookWoo"
date: 2021-07-02 18:07:00 +0900
categories: Study
tags: 논문
---

# Deep neural network obfuscator for machine learning as a service in presence of cache side-channel attacks = 캐시 부채널 공격이 존재하는 서비스로서의 기계 학습을 위한 심층 신경망 난독화기

#### Timing side-channel attacks
언제 security-critical operation이 시작/끝나는지 직접적으로 알기는 어렵지만, shared hardware resources인 TLB와 cache를 이용해 간접적으로 알 수 있다.

#### Cache side-channel attacks
victim이 shared cache에 접근할 때, cahce hit과 cache miss는 처리 시간이 다른 점을 이용함
- Timing-based attack에서는 victim의 security-critical operation 실행 시간과, 평균 실행시간을 비교한다.
  - EVICT + TIME attack : 관심 있는 부분을 cache에서 evictr시켜서, victim이 그 캐시 부분에 접근하면 cache miss가 발생해 실행 시간이 길어지게 함
  - Cache Collision attack : victim이 같은(?) memory line에 접근하면 cache hit이 발생해 실행 시간이 짧아짐
- Access-based attack은 victim의 security-critical operation 이후에 각 memory access time을 측정한다.
  - PRIME(준비시키다) + PROBE attack은 미리 몇 cache set을 준비해놓고, victim이 security-critical operation을 끝낸 뒤 다시 아까 준비해두었던 cache set에 접근한다. 이 때 접근 시간이 길어졌다면, victim이 해당 cache set에 새로운 내용을 덮어쓴 것이다.
  - FLUSH + RELOAD attack은 attacker가 victim과 data와 memory를 공유해야 한다. 먼저 특정 데이터를 캐시에서 미리 FLUSH 시켜놓은 뒤, victim이 security-critical operation을 끝낸 후에 다시 데이터에 접근한다 (RELOAD). 이 때 접근 시간이 짧아졌다면, victim이 해당 data에 접근해서 cache에 써놓았다는 것이다.
  
#### FLUSH + RELOAD

FLUSH + RELOAD attack은 커널이 memory footprint를 줄이려고 하는 page sharing을 활용한다.

Memory deduplication (데이터 중복 제거)
- 서로 다른 프로세스가 같은 page를 중복적으로 가지는 경우, 이를 하나로 합친다.
- 하지만 악의적인 프로세스가 해당 페이지를 수정할 수 있기 때문에, 합칠 때 copy-on-write로 합치고, 이후에 어떤 프로세스가 수정하면 수정된 내용을 담을 따로 페이지를 할당한다.
- L3 cache는 physically indexed and physically tagged cache (PIPT)여서, 다른 프로세스의 virtual address를 몰라도 추적할 수 있다.

#### TLB side-channel attacks

virtual address에서 physical address로의 매핑을 담는 Translation Look-aside Buffer (TLB)도 여러 프로세스가 공유하기 때문에 side-channel attack에 이용할 수 있다.


