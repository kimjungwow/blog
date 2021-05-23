---
title: "CASYS :: Bitfusion"
date: 2021-05-08 16:51:00 +0900
categories: Study
tags: 논문
---


# Intro

DNN의 유행에 따라 accelerator가 연구되고 있지만, DNN의 특성은 잘 고려되지 않고 있다.

DNN의 특성 3가지를 이용해 computation과 communication 줄일 수 있는 bit-flexible accelerator 제안

- DNN은 대부분 massively parallel multiply-adds.
- 이 operation의 bidwidth 줄여도 정확도 문제 없음 
- 하지만 정확도 유지하려면 DNN간 혹은 layer간 bidwidth가 크게 달라짐

BitFusion의 motivation
- bit-level operation은 mul, add에 비례해, bit width 조절 시 bit-level operation 4배 줄임
- On/Off chip memory에서 읽고 쓰는 bit width도 조절 시 에너지 효율 증가 및 효율적으로 storage capacity 사용
- 여러 DNN에서 bit width 줄여도 정확하다는 이전 연구들에서 시작 -> 하지만 한 DNN 안에서도 다양한 bid width를 사용해야 정확하므로, bit-flexible accelerator 제안

BitFusion의 contribution
- Dynamic bit-level fusion and decomposition 
- Microarchitecture design for bit-level composability -> 여러 bitwidth 사용가능한 Processing Engine인 BitBricks 제안
- Hardware-software abstractions for bit-flexible acceleration -> bit-level fusion capability에서 이득을 얻기 위한 Fusion-ISA 제안

## Bit Fusion Architecture

#### Bit-level flexibility via Dynamic Fusion
![Imgur](https://imgur.com/ft4yIwb.png)

하나의 Fusion unit에 그림처럼 16개의 BitBricks가 물리적 2차원 배열 형태로 들어가 있다.

(a) 각 BitBricks는 1bit binary(0,1) 혹은 2bit ternary(-1,0,+1) multiply-add operation 수행한다.

BitBricks들끼리 논리적으로 Fused-PE를 이루기도 하는데, 하나의 Fusion unit 안에 (b),(c),(d)처럼 다양한 개수의 Fused-PE가 존재할 수 있으며, 그에 따라 bit-width가 달라진다.

예를 들어 (b)는 모두 binary/ternary이고, (c)는 2bits for weight(너비), 8bits for inputs(높이), (d)는 weight과 input 모두 8bits이다.

Fusion unit 내에 Fused-PE 개수가 같아도 배열에 따라 bit-width for weights and inputs가 달라진다.

#### Accelerator Organization

![Imgur](https://imgur.com/FREwqjt.png)
Fusion unit들이 systolic array 형태로 모여있음. 

그에 따라 control overhead가 적어져, 많은 BitBricks를 넣을 수 있음 (control에 필요한 area를 최소화한다는 의미인듯)

전체 Fused-PEs array는 하나의 compute unit처럼 동작하는데, 이 때 bitwidth를 바꿀 수 있다는 것이 BitFusion의 장점이다.

systolic array여서 각 fusion unit마다 필요한 것은 Weight buffer뿐이고, input은 row-wise shared되며 psum은 column-wise flowed

또한 원하는 bitwidth로 input, weight data를 Fusion Unit에 주기 위해 buffer뒤에는 multiplexer가 있다.

이를 통해 on-chip SRAM, register file 접근 횟수를 줄여서 에너지 효율적이다.

## Bit Fusion Microarchitecture

Bit Fusion 원리 : 모든 power-of-2 bidwidth 곱셈은 2비트 곱셈으로 decomposed 가능하다.

따라서 2비트 곱셈만 (signed unsigned 모두) 지원하면 된다.

