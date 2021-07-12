---
title: "CASYS :: Ghost Routing"
date: 2021-07-12 23:31:00 +0900
categories: Study
tags: 논문
---

# Ghost Routing to Enable Oblivious Computation on Memory-centric Networks (ISCA 2021)

## 1. Introduction

P1 : 
cloud computing이 발달함에 따라 클라우드에 넣어놓는 데이터를 보호하는 것이 중요해졌다.

특히 physical access가 가능한 attacker는 memory access pattern을 파악할 수 있는데, 이를 막기 위해 **random memory location에 대한 access를 추가하는 Oblivious RAM인 ORAM**이 연구되고 있다.

아직 ORAM memory overhead는 크다.

P2 : 
DRAM의 문제는 memory address encryption이 안 된다는 것인데, 이 논문에서는 memory command 대신 packet을 사용하는 packetized memory interface에서 oblivious computation을 제공하는 방법을 다룰 것이다.

P3 : 이전 연구

P4 : multi-hop network with multiple memory modules 에서 oblivious computation을 지원하는 것이 목표임

P5 : end-to-end encryption을 지원하며 동시에 per-hop latency overhead를 줄이는 microarchitecture 제안함
- intermediate node에서 필요 없는 내용들은 암호화하지 않음

P6 : access pattern을 숨기기 위해 이 논문에서는 ghost packet을 network에 넣는데, packet을 batching하여 overhead를 줄인다.

Contributions
- We propose oblivious computation in a multi-module, memory-centric network by exploiting packetized memory interface and multi-hop network routing.
- We propose distributed, scalable encryption microarchitecture that minimizes per-hop latency while ensuring proper decryption with out-of-order packet arrival at destination.
- We propose Ghost routing that provides secure routing to achieve oblivious computation. Ghost routing exploits existing traffic and ghost packets to minimize performance overhead while still obfuscating the traffic pattern.

## 2. Background & Threat Model

### A. Memory Network & Packetized Interface

![fig1](https://imgur.com/hP4URm2.png)

P1 : 3D로 stacking하는 DRAM

![fig2](https://imgur.com/oqcDsOf.png)

P2 : fig2 (a) 처럼 이전의 ORAM에서는 command와 address가 암호화되지 않았고, packetized interface에서는 fig2 (b)처럼 전체를 암호화했다.

이 논문에서는 fig2 (c) 처럼 route info는 암호화하지 않아 overhead를 줄이려 한다.

### B. Threat Model

P1 : processor과 memory module은 secure하지만, memory module를 잇는 channel과 같이 off-chip components는 insecure이다. 

그에 따라 memory module이 주고 받는 packet을 엿볼 수 있다.

P2 : 이 논문에서는 3D stacked memory를 unsecure로 가정했다.

