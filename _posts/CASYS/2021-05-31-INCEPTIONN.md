---
title: "CASYS :: A Network-Centric Hardware/Algorithm Co-Design to Accelerate Distributed Training of Deep Neural Networks"
date: 2021-05-31 16:51:00 +0900
categories: Study
tags: 논문
---

## Abstract

distributed에서 gradient comm시 빠르고 정확히 하기 위한 방법

## intro
p1
dnn 속도 늘리는 accelerator, distributed는 graduent communication cost큼. worker and aggregator. 이 논문은 NIc에 data compression accelerator 넣어 comm cost reduce

p2
not general but network centric techniques for gradient comm.
three observations.

p3
from observations, most gradients are between minus one and one. So, compress only gradients in that range leads less loss

p4
from observations, graduent comm is big because node communicates with only one node. It burdens aggregator node. So, worker nodes themselves communicate gradient in circular manner.
p5 성능 향상 요약

2.a
p1 : gradient 줄이는 방향으로 weight modify
p2 : datset너무 크면 gpu memory에 다 못넣으니, 데이터셋중 일부 (batch)만 쓰는 stochastic gradient descent
p3,4 : dataset을 각 worker node가 나눠 가지고, aggregator가 각 worker node들에게서 받은 값을 이용해 새 weight  계산후 worker node들에게 전달. 이 comm overhead을 줄이고자 함

2.b
comm overhead 해결 필요

3.
p1
기존의 그라디언트 압축 알고리즘은 오버헤드 큼. 두 가지 인사이트: 그라디언트는 loss를 다소 덜 걱정해도 되고, 대부분의 절댓값 1 이하

3.A
기존 압축 방식은 느리다. 따라서 aggressive 압축 방식 제안. 또한 그라디언트는 iteration마다 쌓이지 않아 loss 조금 있어도 정확성 해치지 않는 점을 이용해, g의 LSB를 줄이는 방식 이용

3.b
그라디언트 대부분 0 근처임을 이용.
그러디언트가 한 방향으로만 소통되는 문제 해결방법부터 논의할것

4ㅋ. gradient 나눠 계산해 calculation overhead solve. gradient comm 방향 두 개로 하고 aggregation node 없애 커뮤니케이션 오버헤드 해결.

5
compression
exp을 127오 고정해, exp 표시하는 비트 필요 없게 함. 얼마나 exp바뀌었는지도 별도로 기억하며, 그 양에 따라 얼마나 압축되는지가 달라진다
원래 값이 너무작으면 아예 저장 안하고0으로 취급

decompression은 당연한 얘기들

6 처음 두 문단

기존 압축 방식은 정확도, 커뮤니케이션 시간만 평가하고 전체 실행 시간 평가 안하는게 문제+ gpu에서 가능한 압축 방식 필요. 이를 위해 hw based compression 다룰것.

### 6.A.Accelerator Architecture and Integration with NIC

#### NIC architecture

단순히 압축/해체 알고리즘만 생각한게 아니고, 그것을 실제로 NIC와 integrate하는 방법을 고민함 + 그 과정에서 생기는 어려움 (floating number 다루는데 TCP/IP pcaket 이용해야함 + lossy compression을 껐다 켰다 할 수 있어야 함)을 논의할 것

#### Compression Engine

특정 패킷이 압축 대상인지 판단하는 Type of Service 필드를 헤더 앞쪽에 넣어서 빠르게 판단함.
Vector, Compression Blocks 개념들

#### Decompression Engine

패킷이 압축 해제 대상인지 판단 + 2개의 burst를 포함할 수 있는 burst buffer + 다양한 사이즈의 variable size

### 6.B. APIs for Lossy Compression of Gradients

압축/해제 대상인지 판단하기 위해 ToS를 0x28로 설정하는 API -> linux NW kernel 손 안 댐


### 7.A
사용한 dnn models 설명

### 7.B Distributed DNN Training Framework

X

### 7.C. Measurement Hardware Setup

X

## 8. Evaluation

### 8.A. Performance Improvement with INCEPTIONN

p1 : 기존의 workker/aggregator (WA) 디자인에서는 dnn model에 상관없이 communication하는데 70% 이상 시간이 소요되서, 명백히 bottleneck임

p2 : INCEPTIONN (aggregator 없앰) + Compression 적용할 때 training time 비교 -> 성능 향상

p3 : INC는 communication time 특히 잘 줄이며, gradient summation이 모든 노드에서 진행되므로 computation time도 잘 줄임

p4 : Compression도 성능 향상에 도움됨. 기존의 WA에 compression만 적용해도 성능 향상

### 8.B. Effect of INCEPTIONN Compression on Final Accuracy

Inceptionn compression 적용해도, 기존의 WA와 같은 수준의 정확성으로 수렴할 때 까지 소요되는 epoch가 거의 비슷하다 -> 즉 상당히 정확하다


### 8.C. Evaluation of INCEPTIONN Compression Algorithm

p1 기존의 naive한 compression 방식은 압축률이 매우 적으며, 정확도 손실도 크다
p2,3 하지만 이 논문의 compression 방식은 압축률 좋고 정확도도 좋다
p4 gradient 압축률이 꼭 communication time 줄이는데 정비례하지 않음 -> 2^-10의 작은 에러 사용하면 패킷 수나 NW stack overhead가 변하지 않기 때문임 -> (FUTUER WORK?)

### 8.D. Scalability Evaluation of INCEPTIONN Training Algorithm

p1 node 수와 관련된 것은 gradient exchange 뿐이니 그것만 비교하였다.
p2 INC에서는 WA와 달리 ndoe 늘려도 gradient exchange time이 linealy increase가 아닌 constant이며 + 그 이유를 수식적으로 설명함


## 10. Conclusion

이 논문은 distributed training에서 bottleneck인 communication을 새 알고리즘 + higher speed NW fabric을 활용해 해결함 + 하드웨어도 알고리즘과 코디자인 -> (1) in-network acceleator for compression (2) gradient-centric distributed training


