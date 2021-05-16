---
title: "CASYS :: Cosmic"
date: 2021-05-16 16:51:00 +0900
categories: Study
tags: 논문
---


Q3 : **Using Stack** CPU가 aggregation/networking 담당하고 accelerator가 gradient 계산하게 해 it maximizes system-wide resource utilization as well as portability to different accelerator boards.

Contribution을 정리한 intro 마지막 문장 :  To this end, this work not only contributes the full stack of CoSMIC, but also defines a new multithreaded accelerator architecture, a novel communication-aware scheduling and mapping algorithm, and a lean and specialized system software for thread management and system orchestration.

Conclusion : accelerator을 최적화하기 위해 system stack를 모두 신경써야함. specialize each layer + offers a cohesive hardware-software
solution.

## 2 Distribution Learning

COSMIC stack은 HW design/system SW programming 신경쓰찌 않고 다양한 learning algorithm을 잘 하는 accelerator-augmented distrubuted system 제공 => performance, generality(gradient descent optimizer 쓰는 알고리즘들에 모두 잘 적용됨), programmability 모두 챙김

CoSMIC facilitates programming by exposing a math-oriented DSL to programmers to express various learning algorithms as stochastic optimization problems.