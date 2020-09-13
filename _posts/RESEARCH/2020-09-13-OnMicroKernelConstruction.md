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