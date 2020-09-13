---
title: "Research :: On Micro-Kernel Construction"
date: 2020-09-13 18:45:00 +0900
categories: Study
tags: Study OS 논문
---

On micro-kernel construction, J Liedtke, SOSP 1995.
를 읽고 기억할 내용을 적었다.

1. Rationale

- Kernel : OS에서 나머지 SW에게 mandatory + common인 부분
- Micro-Kernel : Kernel을 최소화 하는 것 (최대한 kernel 바깥에 구현하기)
- Micro-Kernel을 잘 구현하면 이점
    1. 시스템 구조가 더욱 modular
    1. (다른 user program처럼) 서버가 micro-kernel의 메커니즘 사용 가능하여, 서버의 고장이 isolated
    1. 시스템이 더욱 flexible + tailorable (특정 목적에 적용시킬 수 있는) = 다양한 API가 시스템에 존재 가능

2. Some Micro-kernel concepts

- Address space
    - grant, map, flush operation들은 micro-kernel 내부에 구현됨

- Threads and IPC
    - 각 쓰레드마다 갖는 thread state에 address space가 포함된다. address space의 오류를 막기 위해 thread 개념도 micro-kernel 안에 넣는 것이 편리하다.
    - 따라서 cross-address-space communication인 IPC(inter-process communication)도 micro-kernel 안에 있어야 함. 여기서 IPC는 subsystems간 소통의 방법이자, address space처럼 foundation of independence이다. grant와 map에서 granter/mapper와 receiver간 소통을 위해 IPC 사용된다.

- Unique Identifiers
    - thread 혹은 다른 것에 매길 unique identifier(uid)가 필요함. local communication 시 sender/receiver 명확히 하기 위함. local communicaiton에 암호화를 사용하는 것은 너무 비싸기 때문에 UID면 충분하다. 

