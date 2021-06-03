---
title: "CASYS :: Beyond the socket"
date: 2021-06-03 14:51:00 +0900
categories: Study
tags: 논문
---

## 1 Introduction
좋은 성능을 위해 multi-GPU가 필요하지만, 기존의 single-GPU app을 multi-GPU 용으로 바꿔야하는 귀찮음이 있다.

CPU-GPU 혹은 GPU-GPU interconnect도 계속 연구되고 있다. (기존에는 PCIe만 썼지만 이제는 NVLink도 쓰는 등)

## 2 Motivation and Background

GPU 내 transistor 밀도가 증가하면서 GPU의 성능이 계속 증가해왔지만,  증가 속도가 느려지고 있다.

3D die-stacking은 전력, 에너지, 냉각 등 문제가 있다. 대신 CPU world에서 성공적이었던 multi-socket GPUs를 이용해 GPU 성능 챙기려 함

모든 GPU를 따로 생각하는 discrete GPUs는 single kernel을 가속하기에는 좋지 않음 (app을 수정해야 해서)

이 논문에서는 NUMA penalty 줄이고자 두 가지 시도
- GPU에 연결된 switch를 조사해, 들어가는 것/나가는 것 두 방향을 나눠 asymmetric bandwidth를 제공
  - interconnect를 효율적으로 쓰면, local memory access 대비 remote access 수가 줄어서 성능 향상
- interconnect link 상 traffic 줄이고자 NUMA-aware GPU cache를 제안
  - NUMA에서는 모든 cache miss의 latency가 같은 것이 아니다. slower/faster NUMA zones의 cache capacity를 조절하는 (그에 따라 hit rate가 바뀜) NUMA-aware cache architecture 제안




