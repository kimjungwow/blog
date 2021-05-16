---
title: "CASYS :: Cosmic"
date: 2021-05-16 16:51:00 +0900
categories: Study
tags: 논문
---

## 1 Introduction

paragraph 1 : distributed, accelerator 합치기 어렵( 단순 스택은 cpu에 최적화)

Q1 : layer별로 역할을 나눴고, 중요한 op는 partial gradient, aggregate, # of processed datas

Q2 : 이미 HW는 compute/storage resource 충분하지만, 기존 learning 알고리즘은 (DFG) 이를 활용못함. 그래서 CosMIC은 MIMD 활용해 여러 개의 learning algorithm 계산 (last layer, resource 잘 활용)
FPGA와 P-ASIC 차이에 따른 code generation 차이 설명 (기초 지식 설명 부족?)
template : 2차원 matrix computing unit -> data dependency나 within-thread communication이 scalability 제한 안 하도록
+ 그 외 Scalability 개선한 이야기들

Q3 : **Using Stack** CPU가 aggregation/networking 담당하고 accelerator가 gradient 계산하게 해 it maximizes system-wide resource utilization as well as portability to different accelerator boards.

Contribution을 정리한 intro 마지막 문장 :  To this end, this work not only contributes the full stack of CoSMIC, but also defines a new multithreaded accelerator architecture, a novel communication-aware scheduling and mapping algorithm, and a lean and specialized system software for thread management and system orchestration.

Conclusion : accelerator을 최적화하기 위해 system stack를 모두 신경써야함. specialize each layer + offers a cohesive hardware-software
solution.

## 2 Distribution Learning

COSMIC stack은 HW design/system SW programming 신경쓰지 않고 다양한 learning algorithm을 잘 하는 accelerator-augmented distrubuted system 제공 => performance, generality(gradient descent optimizer 쓰는 알고리즘들에 모두 잘 적용됨), programmability 모두 챙김

CoSMIC facilitates programming by exposing a math-oriented DSL to programmers to express various learning algorithms as stochastic optimization problems.

### 2.1 Learning as Stochastic Optimization

ML 알고리즘이란 loss function을 줄이는 것이 일반적이며, loss function을 줄이기 위해 통상 gradient가 음의 방향으로 빠르게 줄어드는 것에 주목하는 Stochasitc Gradient Descent (SGD) 이용한다. Cosmic에서는 프로그래머가 사용하고자 하는 알고리즘에 따라 gradient of loss function만 명확히해주면 된다.

### 2.2 Parallelizing Stochastic Optimization

SGD 계산 시 여러 노드가 각자 계산한 결과를 average로 합치는 것을 통해 병렬화 가능. Cosmic에서는 2.1의 gradient와, aggregation operator, mini-batch size만 programmer가 알려주면 된다.

## 3 COSMIC SYSTEM SOFTWARE

시중에 나온 node들을 대상으로 잡음

### Execution and acceleration flow

각 node 내에서도 데이터를 나눠서 gradient를 계산한 뒤 local aggregation을 마치고, 그것들을 또 aggregate하는 sigma node에게 보낸다. (sigma node가 아닌 node는 delta node)

한 sigma node에 일이 몰리지 않게 여러 계층의 sigma nodes 사용

### Task assignment in the system software.

aggregation이 gradient 계산보다 쉽기 때문에, gradient 계산은 accelerator가 하고, aggregation은 CPU에서 해서 resource utilization 높임. 앞에서 얘기한 대로 accelerator 내에서도 local aggregation을 함

### Internal thread pools for networking and aggregation

기존의 Thread cr4eation/scheduling/context switch overhead 줄이고자
- alleviate the need to create an active thread for each connection, limiting the number of active threads 
- reuse threads for different connections, mitigating the cost of context switching
- use a producer-consumer semantics between the two thread pools specializing their scheduling


## 4 THE COSMIC STACK

### 4.1 Programming Layer

프로그래머가 high level language를 이용해 gradient, aggregation 등 수학적 표현을 가능하게 해주며, 이를 Dataflow Graph (DFG)로 바꿈. **Translator : Model Specification -> DFG**

### 4.2 Compilation Layer

(잘 이해 못함)

Compiler : DFG -> Operation Schedule/Map. 더 나중 단계인 Planner로부터의 dependency도 있음

### 4.3 System Layer

Section 3에서 얘기한 sigma/delta node

### 4.4 Architecture Layer

**P-ASIC이나 FPGA에 대한 사전지식을 많이 요하는듯. 예를 들면 Planner step은 FPGA의 context에 대해서만 간결히 다룸 (4.4 첫문단 마지막문장)**

2차원에서 각 차원 PE의 수를 조절하며 Single-thread와 parallelism의 성능 balance 조절

## 5 TEMPLATE ARCHITECTURE

다양한 workload + parallelism 챙기기 위한 template arhictecture = 고정되지 않은 architecture

DFG 내에서 paralleism하면 depednecy 때문에 최대 성능 제한 -> independent partial gradient update를 서로 다른 thread가 계산하는 방식으로 multi-threading

### 5.1 Accelerator Organization

같은 row에 있는 PE들에게 한 번에 data를 줄 수 있게, column 수는 memory bandwidth에 따라 정해진다. 또한 worker thread에게 PE를 row granularity(단위)로 할당한다.

dot product, SUM 등은 intermediate results가 많은 PEs에 전달되어야 하기 때문에, communication이 bottleneck이 되지 않도록 해야한다. 따라서, Cosmic에서는 PE들을 연결하는 데 3 계층의 connectivity를 사용함 -> 더 자주 communicate하는 PE를 빠른 것으로 연결한듯

5.1의 PE design부터