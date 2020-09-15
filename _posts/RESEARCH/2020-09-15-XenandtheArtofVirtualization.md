---
title: "Research :: Xen and the art of virtualization"
date: 2020-09-15 17:45:00 +0900
categories: Study
tags: Study OS 논문
---

Xen and the art of virtualization
Paul Barham, Boris Dragovic, Keir Fraser, Steven Hand, Tim Harris,
Alex Ho, Rolf Neugebauer, Ian Pratt, Andrew Warfield, SOSP 2003.
를 읽고 기억할 내용을 적었다.

# Abstract

- **컴퓨터의 리소스를 나눠 쓰기 위한 virtualization**. resource isolation이나 performance guarantee가 잘 되지 않느 것이 문제이다.
- Xen : VM을 잘 추상화해서, OS들이 쉽게 포팅될 수 있게 함 -> **좋은 performance** + functionality 유지하며 resource를 나눠쓰는 virtualization을 잘 구현함

# 1. Intruduction

- VM의 목표는 컴퓨터 리소스를 나눠쓰는 것이며, 아래의 세 가지가 잘 충족되어야 함
    1. 각 VM은 다른 VM (성능 등)에 영향을 주거나 받으면 안 됨 
    1. 다양한 OS + 다양한 app도 사용 가능해야 함
    1. performance overhead 적어야 함

- **VM의 필요성** : VM 대신 standard OS를 사용하면 한 프로세스가 다른 프로세스(의 성능)에 영향 미치는 것을 막지 못함. 
- standard OS에서 performance isolation을 지원하는 기능을 넣으면, 모든 resource usage를 그에 걸맞는 process가 책임지는지 확신하기 어렵다.
- **Xen** 비슷한 방식 사용하지만, process-level이 아닌 physical resource level에서 multiplexing한다. physical resource multiplexing은 다양한 OS가 coexist하기 좋음.


