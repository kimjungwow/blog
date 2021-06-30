---
title: "CASYS :: InDatacenter"
date: 2021-06-30 16:51:00 +0900
categories: Study
tags: 논문
---

# In-Datacenter Performance Analysis of a Tensor Processing Unit (ISCA, 2017)

TPU가 필요한 이유, TPU가 좋은 이유

## 1 INTRODUCTION TO NEURAL NETWORKS

P1 : 큰 data set를 다룰 수 있는 cloud와 좋아진 computing power로 인해 ML 분야는 계속 발전 중임

P2 : Neural Network (NN)에서는 한 layer의 output이 다음 layer의 input이 되며, DNN은 더욱 많은 layer를 통해 더 정확한 모델을 만들기 때문에 "deep"하다고 함

P3 : NN은 training과 inference 두 phase로 구성되며, layer 수나 NN type을 적절히 정한 뒤 training을 통해 weight을 결정한다. NN에서는 floating point가 주로 사용되기 때문에 GPU의 역할이 크며, floating point를 8bit integer 등으로 바꿔 energy/area overhead를 줄이는 quantization도 중요하다.

P4 : NN의 세 종류 
- Multi-Layer Perceptrons (MLP) : 각 layer는 모든 이전 layer들의 output의 (fully connected) weighted sum의 nonlinear functions로 이루어짐.
- Convolutional Neural Networks : 각 layer는 이전 layer 중 spatially nearby subsets들의 weighted sum의 nonlinear functions로 이루어짐 -> weight 재사용 가능
- Recurrent Neural Networks (RNN): 각 layer는 이전 layer의 output과 previous state를 이용함. 예를 들어 Long Short-Term Memory인 LSTM은 무엇을 forget/pass 할지 적절히 정함

P5 : Table 1 + TensorFlow로 짜면 NN들 코드 짧음
[table1](https://imgur.com/Xbe5xCB.png)

P6 : Table 1에서 보면 weight는 5~100MB 필요한데, time/energy to access weight를 줄이기 위해 weight를 batch 단위로 재사용해 성능 향상 시킴

P7 : preview of hightlights
- Inference는 주로 유저가 직접 보는 부분이어서, response-time이 throughput보다 중요함
- latency limit이 있기 때문에, K80 GPU가 훨씬 좋은 peak performance와 좋은 memory bandwidth를 가짐에도 inference에 있어서는 K80 GPU는 Haswell CPU 아주 조금만 빠르다.
- 대부분의 아키텍쳐가 CNN 가속에 집중하지만, 이 논문의 datacenter에서 CNN은 겨우 5%다.
- TPU는 inference에 있어서 K80 GPU와 Haswell CPU보다 15~30배 빠르다.
- Table1의 6개 NN 중 4개는 메모리 제한이 있는데, TPU가 K80 GPU만큼의 메모리를 가지면 CPU, GPU보다 30~50배 빨라질 것이다.
- TPU의 power chip은 작지만, K80 GPU보다 더 많은 (25배) MAC와 (3.5배) chip memory 가진다.
- 전력소비량 당 성능은 TPU가 CPU/GPU보다 30~80배 좋으며, TPU의 메모리가 더 커지면 CPU/GPU보다 70~200배 좋아질 것이다.

## 2 TPU ORIGIN, ARCHITECTURE, IMPLEMENTATION, AND SOFTWARE

P1 : speech recognition DNN의 많은 inference 계산양을 GPU보다 빠르게 처리하기 위해 TPU가 디자인되었다.

P2 : TPU를 CPU와 통합하는 대신, GPU처럼 PCIe I/O bus로 연결된 coprocessor로 디자인했다. 또한 host server가 TPU에 instruction을 보내기 때문에, TPU 스스로 instruction을 fetch할 필요가 없다.

P3 : inference 전체를 TPU에서 실행함으로써 CPU와의 통신을 줄이고, TPU 개발 시점보다 더 미래의 NN 까지 다룰 수 있는 TPU를 디자인하는 것이 목표이다.

[fig1](https://imgur.com/l0HO8Ob.png)

P4 : Figure 1 설명
- host가 보내는 TPU instruction은 왼쪽의 PCIe interface를 통해 들어옴
- 각 internal block은 256-byte-wide path로 연결됨
- 우측 상단의 Matrix Multiply Unit은 256 X 256 MAC를 가지며, 각 MAC는 8bit multiply-and-add 수행함.
- Matrix unit 밑의 accumulator는 16-bit product를 모으는데, 4MB accumulator는 총 4096개의 256-element, 32-bit accumulator로 구성됨.
- Matrix unit은 cycle 당 1개의 256-element partial sum을 만듦
- 4096개인 이유 : peek performance를 위해 byte당 1350번의 operation이 수행되어야 하는데, 그걸 반올림해서 2048이 되었고, double buffering을 위해 2배 해서 4096

P5 : Figure 1 설명 계속
- Matrix unit은 weight와 activation을 8-bit/16-bit 모두 사용 가능하나, 16-bit을 사용하면 속도 절반으로 감소
- Weight FIFO는 Weight Memory라 불리는 8GB DRAM로부터 weight를 읽어오는데, inference에서는 weight은 read-only
- unified buffer는 intermediate results를 저장하는데, CPU Host memory와 데이터를 주고 받음

[fig2](https://imgur.com/l0HO8Ob.png)

[fig3](https://imgur.com/3jiPlbf.png)

P6 : Figure 2 설명
- datapath가 TPU die의 2/3 차지함
  - unified buffer는 1/3
  - matrix multiply unit은 1/4
- control을 위한 공간은 겨우 2%만 차지함

P7 : TPU instruction은 PCIe bus를 통해 전달되기 때문에, TPU instruction은 CISC instruction을 따름. CISC instruction은 cycles per instruction (CPI)가 10~20으로 느리며, 중요한 instruction은 다음과 같음
1. Read_Host_Memory : CPU host memory의 data를 unified memory에 적음
2. Read_Weights : Weight memory의 weights를 Weight FIFO에 적어 Matrix Unit의 input으로 사용함
3. MatrixMultiply/Convolve : B X 256 input과 256 X 256 matrix를 곱해 B X 256 output matrix 구함
4. Activate : ReLU, Sigmoid, Pool 등
5. Write_Host_Memory : unified memory의 data를 CPU host memory에 적음