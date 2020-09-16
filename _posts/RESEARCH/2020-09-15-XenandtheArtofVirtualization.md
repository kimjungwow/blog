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

**컴퓨터의 리소스를 나눠 쓰기 위한 virtualization**. resource isolation이나 performance guarantee가 잘 되지 않느 것이 문제이다.
Xen : VM을 잘 추상화해서, OS들이 쉽게 포팅될 수 있게 함 -> **좋은 performance** + functionality 유지하며 resource를 나눠쓰는 virtualization을 잘 구현함

# 1. Intruduction

VM의 목표는 컴퓨터 리소스를 나눠쓰는 것이며, 아래의 세 가지가 잘 충족되어야 함
1. 각 VM은 다른 VM (성능 등)에 영향을 주거나 받으면 안 됨 
1. 다양한 OS + 다양한 app도 사용 가능해야 함
1. performance overhead 적어야 함

**VM의 필요성** : VM 대신 standard OS를 사용하면 한 프로세스가 다른 프로세스(의 성능)에 영향 미치는 것을 막지 못함. 
standard OS에서 performance isolation을 지원하는 기능을 넣으면, 모든 resource usage를 그에 걸맞는 process가 책임지는지 확신하기 어렵다.
**Xen** 비슷한 방식 사용하지만, process-level이 아닌 physical resource level에서 multiplexing한다. physical resource multiplexing은 다양한 OS가 coexist하기 좋음.

# 2. Xen : Approach & Overview

이전에는 *full virtualization* (실제 장비와 노출된 virtual HW가 같음)을 사용했지만, OS가 virtual resource 뿐 아니라 실제 resource도 봐야 성능이 좋아지기도 한다. 그래서 Xen은 *paravirtualization* (VM abstraction)을 통해 이 단점을 극복 -> Guest OS 수정이 필요할 수 있지만, app은 수정 필요 X 
Xen의 approach
1. app의 binary 수정하지 않아도 Xen에서 동작해야함
1. Xen에서 사용하는 하나의 OS에서 여러 개의 app이 동작할 수 있어야 함
1. full virtualization 말고 paravirtualization이 필요함 (성능 + resource isolation 위해)
1. 하지만 guest OS에서의 영향을 완전히 숨기는 것(hiding)은 성능과 correctness를 해칠 수 있ㅇ므

Denali와 비교했을 때 Xen의 특징
- Xen은 수정되지 않은 app들을 실행하는 OS 약 100개를 한 machine에 담는 것이 목표이다. 각 guest OS별로 페이징이 진행되어 다른 guest OS의 성능에 영향을 끼치지 않도록 함.
- Xen은 physical address를 guest OS에게 직접적으로 보여줌. 이 때 correctness + performance 이슈도 잘 챙김. (hypervisor가 관리)


용어
- Xen은 guest OS의 supervisor code보다도 높은 privilege level 가지므로, **hypervisor**
- Xen이 host하는 OS : **guest operating system**
- guest OS가 실행되는 VM : **domain**

## 2.1 The Virtual Machine Interface

### 2.1.1 Memory management

Xen의 Memory management는 x86 기준 : tagged TLB나 software-managed TLB가 사용되면 메모리를 가상화하기 수월하지만, x86은 어려운 케이스이다. TLB miss 발생시 processor가 HW의 page table structure를 walk한다.
(i) HW의 page table에 필요한 내용이 있어야 성능이 좋고, (ii) untagged TLB이므로 transferring execution시 TLB flush가 필요하다.
따라서 guest OS가 HW page table을 할당 및 관리 하고 (Xen의 도움을 최소한으로 받고) + Xen은 모든 address space의 top에 64MB section을 가짐으로써 hypervisor mode에 들어가고 나갈 때 TLB flush가 필요 없도록 함. 이 떄 top 64MB section은 Xen만 접근 가능.
page table 할당은 guest OS가 하지만, update하려면 Xen의 검사를 받아야 함 -> 서로 다른 OS의 메모리에 mapping 못함

### 2.1.2 CPU

Guest OS가 hypervisor 수정 못하게 하고 + 다른 guest OS에 영향을 못 끼치게 하기 위해서는 -> guest OS는 hypervisor보다 낮은 privilege level 가져야 함.
대부분의 processor는 단 2개만의 privilege level 제공 -> guest OS는 낮은 level을 사용하되, app과 다른 address space를 사용함으로써 virtual privilege level을 사용
xv6는 Ring 0~3으로 4개의 privilege level -> guest OS는 Ring 1 사용
exception handler는 Ring 0 (Xen)에 있음
가장 많이 발생하는 exception : system call, page fault
1. system call은 각 guest OS가 직접 접근할 수 있는 주소를 저장 (syscall은 수정되지 않기 때문에) 가능
1. page fault address는 Ring 0에서만 사용 가능하기 때문에, Ring 0을 통해 해결되어야 함.
Xen의 "double faults" : exception handler의 주소를 접근할 때 page fault 발생하는 경우 : guest OS 종료시킴?

safety?

## 2.2 The Cost of Porting an OS to Xen

XP는 Linux보다 Xen에 포팅하기 위해 수정해야할 것이 더 많다. (Linux는 Page table 접근 관련 매크로가 많아서 사용 가능)

## 2.3 Control and Management

- Xen에서는 기본적인 operation만 다루고, guest OS에서 policy 등 high level을 다룬다.

# 3 Detailed Design

## 3.1 Control Transfer: Hypercalls and Events

- Hypercalls
    - guest OS -> Xen
    - sync
- Events
    - Xen -> guest OS
    - async
    - callback handler가 event 처리시 interrupt를 껏다 키듯 Xen-readable SW flag를 설정한다.


## 3.3 Subsystem Virtualization

### 3.3.3 Virtual address translation

별도의 shadow page table을 두는 대신, Xen은 guest OS에게 MMU page table의 read-only access를 부여함. update는 Xen이 담당함.





