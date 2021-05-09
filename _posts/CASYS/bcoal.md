---
title: "CASYS :: BCOal"
date: 2021-04-27 16:51:00 +0900
categories: Study
tags: 논문
---

GPU는 CPU보다 throughput이 월등히 높아서 점점 널리 쓰이고 있다. 그에 따라 GPU가 민감한 데이터들을 많이 다루게 된다.

### Background
warp의 threads들은 같은 instruction을 실행한다

Memory bandwidth는 GPU의 performance-critical shared resource

GPU memory bandwidth optimization techniques 예시
- Access coalescing
  - Warp의 LD/ST unit이 그 warp 내의 서로 다른 스레드에서 발생한 multiple memory requests를 합치는 **intra-warp coalescing**
  - 쉽게 coalesced accesses를 예측할 수 있으면 취약하기 때문에, RCOal은 각 스레드의 sub-warp를 랜덤하게 설정하여 coalesced accesses를 예측하기 어렵게 했다.
- Caching
- Merging
  - cache miss가 발생하면 memory access는 **miss-status holding registers (MSHRs)** 에 기록된다
  - 같은 SM의 서로 다른 warp에서 같은 cache block에 대한 cache-missed coalesced accesses를 MSHR이 합친다 **inter-warp merging**
  - MSHRs은 같은 warp에서 시간 차를 두고 redundant accesses를 보내면 이것도 합친다 **intra-warp merging**

GPU에서의 AES는 plaintext를 multiple parallel threads가 나눠서 암호화한다.

**Baseline attack** : attacker가 암호화하는데 소요된 시간과 cache request 횟수를 통해 암호를 해독하는 경우를 고려한다. **Side channel attack**

**Baseline defense** : RCoal은 thread의 subwarp를 무작위로 정해서 coalesced accesses를 예측하기 어렵게 했다. 그에 따라 coalesced accesses 개수와 execution time의 상관관계가 옅어져서 공격하기 어려워진다. 문제는 성능이 안좋아지고 SM과 메모리 사이에서 데이터가 많이 이동하게 된다.

## Motivation and Analysis

RCoal에서는 보안이 강해지면 성능은 안 좋아진다. MSHR이나 Cache가 합칠 수 있는 accesses들을 합쳐버리면, 다시 coalesced accesses 개수와 execution time의 상관관계가 뚜렷해지기 때문에 보안이 약해진다. 따라서 RCoal에서는 MSHR과 Cache를 사용하지 못해 성능이 안 좋다.

(Cache가 모든 경우 hit이면 execution time이 constant여서 상관 없지만, cache miss가 발생하면 MSHR이 accesses들을 merge 시켜버리기 때문에 key를 예측할 수 있게 된다)

## BCoal

coalesced accesses의 개수를 사전에 정해둔 값 (bucket)과 같게한다. 이 때 MSHR이 merge를 못하도록 **unique** additional padded accesses를 추가한다. BCoal의 목표는 attacker가 real coalesced accesses의 수와 observed execution time의 상관관계를 알기 어렵게 하는 것이다.

bucket을 1개만 쓰면 coalesced accesses의 개수가 일정해져서 제일 보안적으로는 좋지만, additional padded accesses가 많아져 성능에 안 좋다.따라서 여러 개의 bucket을 쓰는데, 예를 들어 BCoal(1,16)은 최종 coalesced accesses의 개수가 1 혹은 16이 되도록 한다.

**padded accesses가 execution time에 끼치는 영향이 real access와 같도록**, 같은 address range를 향하게 해야한다.