---
title: "CASYS :: Efficient Data Protection for Distributed Shared Memory Multiprocessors"
date: 2021-05-24 16:51:00 +0900
categories: Study
tags: 논문
---

## 5 PROCESSOR-PROCESSOR DATA PROTECTION FOR DSM

![Imgur](https://imgur.com/KxHunsl.png)

기존에는 (a)처럼 overhead 크지만, 미리 PAD 만들면 encrypt/decrypt overhead hide 가능함.

authentication overhead(verify MAC)을 hide하기 위해서는 Galois Counter Mode (GCM) 사용.

GCM은 counter를 이용하기 때문에 마찬가지로 PAD 미리 만들어두면 authentication overhead 줄일 수 있음.

GHASH라는 a short chain of Galois Field Multiplications and XORs, each of which can be performed in one cycle 만 계산하면 됨

## 5.1 Private Counter Stream

모든 sender processor / receiver processor pair마다 counter 따로 가짐.

64bit counter value, 512bit pre-generated encryption pad, 128bit pre-generated authentication pad, 1bit valid bit으로 1+64+512+128 = 705bits

valid bit란 pad가 pre-generated지만 not consumed임을 확인하는 용도임.

processor개수가 N이면 (2XNX705/8)bytes per processor

sender
- valid bit이 set이면, 바로 pad를 encrypt/authenticate에 사용
- valid bit이 cleared이면 (a pad half-miss) 현재 만들고 있는 pad가 사용가능해질 떄 까지 기다림. (이 경우는 드물다)

receiver는 받은 pad의 counter와 pre-generated pad의 counter가 같은지도 확인함
-> 따라서 sender가 보낸 순서대로 receiver에게 도착하지 않으면 (드문 경우지만), pad를 만들기 위해 기다리는 시간이 늘어나 성능 감소

## 5.2 Shared Counter Stream

Private scheme에서는 processor 수 커지면 storage overhead가 늘어남을 해결하기 위한 scheme

sender가 모든 receiver에 대해 하나의 counter를 사용함. 그에 따라 receiver는 불연속적인 counter를 받을 확률이 커져서, counter hit rate가 감소함

## 5.3 Cached Counter Stream

앞선 두 scheme은 고정된 수의 processor에서만 사용 가능한 scalability 문제가 있었는데, 이를 해결하기 위한 scheme

각 프로세서는 나머지 모든 프로세서가 아닌, 일부 프로세서와만 communicate하는 경우가 많기 때문에 send/receive table을 캐시로 대체하면 성능이 좋아짐 + 기존 캐시와 다르게 write back이 필요 없고 그냥 버리면 됨

문제는 cache miss 발생시 counter를 어떻게 할지
- receiver는 sender가 보낸 counter 사용
- sender는 지금까지 자기가 사용한 가장 큰 counter (maxCtr)를 기억해둬서, 그것보다 1큰 값을 사용

Cached scheme은 연속적인 counter value 받을 확률이 커서 shared보다는 성능이 좋지만, private보다는 성능이 안 좋음.
대신 scalability 해결

## 5.4 Detecting Replay Attacks

old message/MAC 사용하는 replay attack을 막기 위해, message 속 counter가 receive table 속 counter보다 작으면 replay attack으로 취급하면 안 된다. sender가 보낸 메시지가 다른 순서로 도착하는 out-of-order message delivery 때문이다.

