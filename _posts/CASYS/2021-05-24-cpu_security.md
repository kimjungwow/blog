---
title: "CASYS :: Efficient Data Protection for Distributed Shared Memory Multiprocessors"
date: 2021-05-24 16:51:00 +0900
categories: Study
tags: 논문
---

## 5 PROCESSOR-PROCESSOR DATA PROTECTION FOR DSM

![Imgur](https://imgur.com/KxHunsl.png)

기존에는 (a)처럼 overhead 크지만, 미리 PAD 만들면 encrypt/decrypt overhead hide 가능
authentication overhead(verify MAC)을 hide하기 위해서는 Galois Counter Mode (GCM) 사용. GCM은 counter를 이용하기 때문에 마찬가지로 PAD 미리 만들어두면 authentication overhead 줄일 수 있음. GHASH라는 a short chain of Galois Field Multiplications and XORs, each of which can
be performed in one cycle 만 계산하면 됨

## 5.1 Private Counter Stream

모든 sender processor / receiver processor pair마다 counter 따로 가짐.

64bit counter value, 512bit pre-generated encryption pad, 128bit pre-generated authentication pad, 1bit valid bit으로 1+64+512+128 = 705bits

valid bit란 pad가 pre-generated지만 not consumed임을 확인
processor개수가 N이면 (2XNX705/8)bytes per processor

sender
- valid bit이 set이면, 바로 pad를 encrypt/authenticate에 사용
- valid bit이 cleared이면 (a pad half-miss) 현재 만들고 있는 pad가 사용가능해질 떄 까지 기다림. (이 경우는 드물다)

receiver는 받은 pad의 counter와 pre-generated pad의 counter가 같은지도 확인함
-> 따라서 sender가 보낸 순서대로 receiver에게 도착하지 않으면 (드문 경우지만), pad를 만들기 위해 기다리는 시간이 늘어나 성능 감소

## 5.2 Shared Counter Stream