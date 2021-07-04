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

P1 : Run-time layer sequence identification problem 설명
- input : architectural hint vectors of kernel sequence X with temporal length of T
- time step t의 kernel feature X_t는 multiple-dimension tuple of arhictectural hints
- goal : train a layer sequence identifier h, ground-truth layer sequence (L\*)과 가장 가까운 run-time layer sequence L를 예측하기

P2 : Run-time layer sequence identification은 kernel model과 layer-sequence model로 이루어짐
- kernel model : architectural hint와 kernel type의 관계
  - **architectural hint vector에는 kernel execution latency, Read volume, write volume, kernel dependency distance가 포함됨**
- layer-sequence model : layer간의 확률 분포
  - speech에서 이야기한 예 : 하나의 linear layer로 이루어진 basic block가 batch normalization을 가지고 있지 않다면, 그 block은 ReLU와 같은 nonlinear layer로 끝남.
**Run-time layer sequence prediction이 speech recognition과 비슷**하기 때문에, DeepSniffer는 speech recognition에서 사용되는 방법들을 활용함

[fig3](https://imgur.com/xTJWhEM.png)

#### 5.1.2 Kernel and Layer Features

##### Architectural Hints of A Single Kernel

P1 :  kernel depedency distance(kdd, bus snooping 시나리오에서만), kernel execution latency, read volume, write volume, input/output data volume ratio
kdd : 어느 커널과, 그 커널 이전의 dependent 커널 중 최대 거리 : layer topology information을 담음

P2 : 하지만 single kernel architectural hint만 이용해서, kernel이 어떤 layer에 속하는지, 즉 layer sequence를 예측하기는 어렵다.

##### Inter-Layer Sequence Context

P3 : **layer사이의 temporal association**을 이용하면 model extraction을 더 잘 할 수 있음.
예를 들어, linear transformation layer가 두 번 연속 나오는 것은 말이 안 되기 때문에, DNN model에서 FC layer 바로 뒤에 Conv layer가 올 확률은 낮다.
- architecture는 iteratively connected basic blocks로 구성됨 (같은 구성이 연속적으로 나온다는 의미인듯?)
- basic block의 구성 순서
  - Conv, FC와 같은 linear operation
  - convergence를 개선하기 위해 batch normalization
  - 그 뒤에
    - ReLU 같은 non-linear transformation 혹은 
    - Pool같은 down-sampling layer 혹은 
    - Add나 Concat 같은 tensor reduction or merge
  
P4 : P3에서 이야기한 layer context가 실제 network architecture design에도 반영되고 있기 때문에, 이 논문의 layer identification에서 사용할 수 있음

#### 5.1.3 Context-aware Layer Sequence Identification

P5 : context-aware layer sequence identifier h에 대해, h(X)가 최대한 target layer sequene 

##### An Example for Layer Sequence Prediction

P6 : kernel sequence의 i번째 frame은, 그것의 kernel architectural hint vector X_i를 가진다. 그 X_i를 통해, i번째 frame이 어떤 Conv, ReLU 등의 kernel일 확률 분포들을 구하는데, 그 확률분포들의 벡터를 K_i라 한다.

P7 : 이전 커널들의 확률 분포를 이용해 조건부 확률을 계산하고, 가장 가능성이 높은 output L을 구한다. 이 때 이 L이 ground-truth layer sequence L\*와 가까워지도록 예측한다.

P8: X_i, X_i+1, X_i+2에 대해 가장 조건부 확률이 높은 layer sequence (예: Conv, ReLU, Pool)을 계산함 -> fig3의 b 참고

### 5.2 Layer Topology Reconstruction

P1 : DeepSniffer는 layer의 memory access pattern을 통해 layer topology를 알아냄.
이제 memory traffic이 어떻게 layer 간의 interconnection을 드러내는지 설명할 것이다.

P2 : 특정 layer의 filter data가 다른 layer의 input data로 사용되면, 두 layer 간에는 interconnection이 있을 것이다.

P3 : **Observation-1**. memory traffic data의 종류는 input images, weight parameters, feature map data 세 가지가 있는데, read-after-write (RAW) memory access pattern은 feature map data에서만 나타난다. feature만 inference 중 수정되기 때문이다.

P4 : **Observation-2**. feature map data는 특히 convergent layer(여러 layer in different branches로부터 data를 받아 input으로 사용)와 divergent layer(여러 layer in different branches에 output 전달)에서 RAW access pattrn 확률 높음

P5 : DeepSniffer는 P3,4 의 observation 이용해 layer간 interconnection 찾음 : 어느 layer의 read request들과 그것보다 전의 layer들의 write request들의 교집합이 공집합이 아니라면, 두 layer 간에 connection 만듦.

P6 : P5에서는 feature map data의 mem addr trace가 **complete할 필요가 없어서**, memory traffic filtering으로 방어하려 해도 소용 없음 (robustness)

### 5.3 Dimension Size Estimation

P1 : Dimension size estimation은 아래 두 단계로 이루어짐
- Layer feature map size prediction
- Dimension space calculation

P2 : **ReLU의 cache miss rate이 98% 이상임을 이용**해, R_v(read volume)을 알면 그것과 비슷한 ReLU의 input feature map size를 알 수 있고, R_v를 통해 W_v(write volume)을 알 수 있음. ReLU의 input/output size를 통해 DNN model의 dimension parameter를 알아냄

P3 : **Step-1 : Layer feature map size prediction**. 이전 layer의 feature map output size와 이후 layer의 feature map input size가 같음을 이용해, ReLU 전후에 있는 BN/Add/Conv/FC layer의 feature map size를 예측한다.

P4 : **Step-2 : Dimension Space Calculation**. 앞에서 얻은 constructed layer topology 및 각 layer의 input/output size를 이용해 다음과 같은 dimension space를 계산한다.
- input/output channel size (IC_i/OC_i)
- input/output height (IH_i/OH_i)
- input/output width (IW_i/OW_i)
- weight size (K X K)
- convolution padding P
- convolution stride S

P5 : 이 논문에서는
- 비전에 집중하므로 IC_0 = 3
- 일반적인 경우처럼 feature map의 height과 weight 같음 + stride = 1
- kernel size를 1,3,5로 늘려가며 다른 parameter들을 예측함

P6 : DeepSniffer가 정확한 dimension size parameter를 예측하는 것이 아니고, 효율적인 attack이 가능한 dimension size parameter를 예측함 (victim model의 dimension size parameter와 다를 수 있음)

## 6 Experimental Results

두 시나리오에서의 정확도와 robustness 판단

### 6.1 Evaluation Methodology

P1, P2 : 실험 환경

P3 : Training data 준비 - DNN model을 나름의 방식으로 랜덤하게 만듦

P4 : 효율성과 범용성 테스트 위해 여러 벤치마크 사용

### 6.2  Layer Sequence Identification Accuracy

layer sequence identification 정확도 분석 + layer context information의 중요성과 architectural hints 속 noise의 영향 분석

#### 6.2.1 Evaluation Metric

Layer Prediction error Rate (LER)을 이용해 얼마나 ground-truth layer sequence L\*에 가깝게 예측했는지 평가함

#### 6.2.2 Side-Channel Attack Scenario

Bus Snooping scenario 보다는 side channel attack scenario에서, 간단한 model보다는 복잡한 모델에서 LER가 더 높음

#### 6.2.3 Bus Snooping Attack Scenario

Bus Snooping scenario 에서는 memory address trace로부터 얻은 kernel dependency distance를 추가로 알고 있기 때문에, LER이 더 낮아짐.
DeepSniffer는 충분히 작은 LER을 얻었으며, ground-truth layer sequence와 똑같지 않아도 괜찮음을 6.4에서 보일 것

#### 6.2.4 Robustness to Hint Noise

layer sequence identifier는 architectural hint noise에 sensitive하지 않는 결과를 보여줌

#### 6.2.5 Why is Inter-Layer Context Important?

inter-layer context (fig3의 sequence model을 포함하는지 여부)를 고려해야 
- LER이 작음
- model이 복잡해져도 LER 증가폭이 작음

### 6.3 Model Size Estimation

대부분의 layer는 정확히 input/output size를 예측하지만, FC는 정확하지 않음. network의 끝에 주로 있고, neuron 수가 줄어들기 때문.
하지만 dimension size 예측은 나머지 2개에 비해 덜 중요하다고 주장함.

### 6.4 How Effective are the Extracted Models?

#### 6.4.1 Adversarial Attack with Extracted DNN Archs

adversarial attack에서는 input image에 사람이 알아보기 힘든 작은 변화를 넣음.
이 때 모델이 올바르지 않은 결과를 내도록 하는 가장 작은 변화를 찾는 것이 adversary의 목표임.

transfer-based adversarial attack flow
- Build substitute models : 얻어낸 network architectures를 갖는 substitute model을 train한다.
- Generate adversarial examples : 여러 모델에서 adversarial한, 즉 효율적인 adversarial image를 찾음
- 효율적인 adversarial examples를 input data로 사용해 black-box model을 공격함

#### 6.4.2 Adversarial Attack Efficiency

DeepSniffer는 기존의 adversarial attack과 다르게 network architecture를 예측하여 substitute model을 만들었기 때문에 더 효율적임. 따라서 network architecture를 보호하는 것이 중요함!

### 7 Discussion

#### 7.2 Defence Strategies

- Microarchitecture Methodologies.
  - Oblivious RAM : data address를 암호화해 memory access pattern 숨기는 방법. memory bandwidth overhead가 너무 큼
  - Dummy Read/Write Operations : noisy R/W operation을 추가함 : layer sequence prediction을 효과적으로 막지는 못함
- System Methodologies
