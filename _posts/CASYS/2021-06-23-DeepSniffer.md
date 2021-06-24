---
title: "CASYS :: DeepSniffer"
date: 2021-04-25 16:51:00 +0900
categories: Study
tags: 논문
---

# DeepSniffer: A DNN Model Extraction Framework Based on Learning Architectural Hints (ASPLOS 2020)

## 1 Introduction

P1 : DNN이 다양한 분야에 사용된다

P2 : DNN에서 model architectures (number of layers, layer connection topology, layer types, and the layer dimension sizes)는 중요하다 -> 노출되면 adversarial attack, model extraction attack이 쉬워진다.

P3 : model extraction attack 관련 이전 algorithm-level 연구들은 계산 시간이 오래 걸리고 resource overhead가 큰 것이 문제였다.

P4 : P3의 한계점 때문에, (algorithm-level 대신) architecture-level model extraction이 연구되고 있다.

P5 : 하지만 architecture-level model extraction 관련 이전 연구들에는 아래와 같은 문제들이 있다.
- layer type이나 DNN architecture 등 victim model에 대한 priori model knowledge가 필요하다
- generality와 robustness 부족 : diverse runtime system optimization and architecture-level noises 에서는 잘 동작하지 않음
- Incomplete model extraction : dimension size와 neuron number 중 하나만 알아내기 때문에, extracted information과 end-to-end attack effectiveness 간의 관계를 보이기 어려움

P6 : DeepSniffer는 P5의 세 가지 문제를 해결하는 framework이며, 세 단계로 구성됨 : run-time
layer sequence identification, layer topology reconstruction, and dimension size estimation.
특히, layer sequence prediction과 dimension size prediction을 분리해 generality와 robustness 챙김.

## 2 Background and Challenges

### 2.1 Model Characteristics

P1: Model extraction attack에 사용되는 DNN model characteristics
- **network architecture** : layer depth and types, connection topologies between layers, and layer dimensions (including the number of channels, feature map size, weight kernel size, stride, and padding in each layer)
- parameters (training process 내 stochastic gradient descent 동안 계속 바뀜) : the weights, biases, and Batch Normalization (BN) parameters
- hyper-parameters : the configurations during training, including the learning rate, regularization factors, and momentum coefficients, etc.

P2 : Adversarial attack의 시작은 Model extraction 이며, 특히 network architecture가 중요한 정보이다.

### 2.2 Model Extraction Techniques

P1, P2, P3 : Intro의 P4, P5, P6 내용 반복

## 3 Attack Model and Arch-Hints

P1 : threat model : attacker가 edge device의 GPU platform에 physically access 가능

P2 : GPU platform 구성 : PCIE(CPU, GPU 연결), GDDR5 memory bus(GPU, Device memory 연결), DDR memory bus(CPU, Host memory 연결)

P3 : 아래 P4, P5에서 두 가지 attack scenario 제시할 것

P4 : Scenario-1 (Side-channel attack) : EM side-channel attack을 통해 read and write memory access volumne (R_v, W_v), kernel execution time(Exe_Lat)를 얻고, -> victim model은 chained DNN

P5 : Scenario-2 (Bus snooping attack) : memory bus, PCIe event를 monitor함. GDDR memory bus의 mem access trace를 관찰하여 kernel read/write access volumne(R_v, W_v)와 memory address trace를 알아냄. PCIe bus에서 kernel이 시작/종료될 때 존재하는 control message를 관찰하여 kernel execution latency(Exe_Lat)도 알아냄 -> victim model은 complex DNN

P6 : Bus snooping은 실용적이고 low-cost attack -> 접근하는 address만 알 수 있다고 가정 -> 데이터는 어차피 못 보기 때문에, data가 encrpyt 되어도 attack 가능함

## 4 Observation and Design Overview

P1 : 다음 문단 소개

P2 : **DNN System Stack Noises** : DNN system의 workflow는 다음과 같다.
- DNN model의 network architecture를 최적화해 framework-level computational graph of layers 만듦.
- high-level computational graph abstraction을 hardware primitives of run-time layer execution sequence로 만듦
- run-time hardware primitive libraries가 layer type에 맞는 well-optimized kernel sequence를 만듦
- 그 kernel sequence가 hardware platform에서 실행되는데, 이 때 memory access pattern이나 kernel execution latency 같은 architectural hints가 나옴

P3 : 이 architectural hints로부터 model architecture를 파악하는 것은 아래 두 noise 때문에 어려움
- **Architectural noise** : shuffling address mapping, complex memory hierarchy 등의 comprehensive memory system optimization 때문에, 정확히 dimension size를 예측하기 위한 complete memory traces를 식별하기 어려움
- **System noise** : layer와 kernel이 일대일 매핑되는 것이 아니기 때문에, kernel sequence로부터 layer 수나 layer boundary를 파악하기 어려움

P4 : **Observations** 
- **Run-time kernel implementations vary** across different models and even across time for the same model
- The kernel sequences of different layers have a **static execution order** related to the original computational graph of a DNN model.

P5 : **Design Overview**
DeepSniffer는 run-time layer sequence identification을 알아내서, 그를 통해 layer topology reconstruction과 dimension size estimation 진행함

P6 : run-time layer sequence identification : kernel-grained architectural hint sequence로부터 run-time layer sequence를 계산함. 이 때 dimension size parameter 정확히 몰라도 괜찮아서, generality와 robustness챙김

P7 : Layer topology reconstruction : bus snooping scenario에서, read-after-write(RAW) memory access pattern를 이용하는데, not complete but partial memory trace만 있어도 되서 실용적임

P8 : Dimension size prediction : ReLU kernel이 보통 large read cache miss 가지므로, ReLU read volume 통해 dimension size 예측하지만, 정확하지 않다. 하지만 이 work에서는 dimension size가 정확하지 않아도 효율적인 end-to-end adversarial attack 가능하다.

## 5 DeepSniffer Design

### 5.1 Run-time Layer Sequence Identification

#### 5.1.1 Problem Formalization

