---
title: "CASYS :: Eyeriss"
date: 2021-01-18 16:51:00 +0900
categories: casys
tags: 논문
---

## Eyeriss: A Spatial Architecture for Energy-Efficient Dataflow for Convolutional Neural Networks

Context
- 정확하면서 에너지 cost 적은 CNN을 위해 병렬 프로세싱에서 data movement cost를 최소화하는 dataflow가 필요함
  - local data reuse of filter weights and feature map pixels
  - minimizing data movement of partial sum accumulations

Gap
- 기존에 존재하던 dataflow들은 특정 종류의 data movement만을 줄임

Innovation
- **RS(row stationary) dataflow**는 모든 data movenet를 줄임 -> **energy efficient**


## Context
- 계산량 많은 CNN에서 throughput과 energe efficiency 둘 다 성취하기 위해서는?
  - dataflow supporting parallel processing with minimal data movement가 필요함
- Spatial Architecture (SA) : processing engines(PEs) 사이의 direct communication을 통해 high cimpute parallelism 제공하는 accelerator
  - CNN layer의 operation들은 동일하고 병렬적임
  - partial sum을 전달해 distributed accumulation + sharing same input without higher energy data transfers
- **SA accelerator**
  ![Imgur](https://imgur.com/wyZWeo2.png)
  - iFIFO / oFIFO : Global buffer + array of PEs + DRAM끼리 communicate
    - Input : CPU/GPU -> DRAM -> accelerator
    - Output : 역순
  - Global buffer
    - 느린 DRAM 대신 캐시 + 임시로 데이터 저장할 때
  - Processing Engines
    - 서로 Noc(Network on Chip)을 통해 연결됨
    - ALU datapath 포함 : MAC(multiply-and-accumulate)와 addition
    - RF(register file) : local scratchpad
  - four levels of storage hierarchy for data access
    - DRAM (highest energy cost)
    - global buffer
    - array (inter-PE communcation)
    - RF (lowest energy cost)
  - dataflow에 따라 RF 크기, global buffer 크기, NoC 디자인 등이 달라짐
- **CNN background**
  - 여러 computation layers을 통해 input data의 higher-level abstraction인 **feature map (fmap)** 얻음
  - **CONV(convolutional) layer input fmaps (ifmaps)에 filter를 적용해 output fmaps (ofmaps)를 얻음**
  ![Imgur](https://imgur.com/5sqS5Br.png)
  - CONV layer가 CNN의 throughput/energy efficiency에 가장 중요함 (FC layer 보다 더) : two challenges -> **data handling** and **adaptive processing**
  - data handling
    - 읽어야 할 input 많음 -> bandwidth + energy consumption issue -> **data reuse**
      - convolutional reuse (reuse filter + ifmap pixel)
      - filter reuse
      - ifmap reuse
    - intermediate data (i.e., partial sum) 양 많음 -> storage pressure + memory R/W memory
  - adaptive processing
    - partial sum을 최대한 빨리 reduce하는 것은 data reuse와 동시에 할 수 없다
    - 따라서 dataflow가 여러 경우에 효율적으로 바꿔야 한다?

## Gap
Existing CNN dataflows
![Imgur](https://imgur.com/kwMqyla.png)
- Weight Stationary (WS) Dataflow
  - **RF 안에 filter weight를 유지**해서 convolutional + filter reuse 최대화
  - ifmap pixel이 PE들에 broadcast + spatially accumulate across PEs
  - **stationary weights를 재사용하는 것에 초점이 맞춰져 있어, psum이 바로 reduce되지 않고 global buffer에 저장되어 있을 때도 있음 -> global buffer 용량에 따라 한 번에 생길 수 있는 pusm 개수 제한 -> on-chip에 로드되는 filter 개수 제한**
![Imgur](https://imgur.com/wrppfbr.png)
- Output Stationary (OS) Dataflow
  - **PE 안에 accumulation** of each ofmap pixel를 유지
  - input data reuse는 array level (inter-PE communication)
  - psum accumulation은 RF level
  - **단점??**
![Imgur](https://imgur.com/ah3fKc2.png)
- No Local Reuse (NLR) Dataflow
  - **RF level이 아닌**, array level (inter-PE communication)으로 ifmap reuse + psum accumulation
  - global buffer를 psum 저장 + reuse input을 위해 사용
  - **단점??**

## Innovation

![Imgur](https://imgur.com/tzIz0RV.png)
- **RS(row stationary) dataflow**는 모든 data movenet를 줄임 -> **energy efficient**
  - utilize the processing engine (PE) local storage -> (**Reg File**?)에 filter/fmap/psum
  - direct inter-PE communication
  - spatial parallelism
  - 위 그림(1D Convolution Primitives)는 각 primitive(convolution 전체에 몇십만개의 primitive 있음)를 PE array에 배정하는 데 에너지 낭비가 커서, Two-Step Primitive Mapping이용
    - Logical mapping
      - primitive를 logical PE in logical PE array로
      - *logical PE set* : spatial locality를 이용해 reuse 최대화
    - Physical mapping
      - *folds* logical PE array -> physical PE array에 맞춰짐 : 서로 다른 logical PE의 1D convolutional primitives를 같은 physical PE에서 진행
      - *fold*는 logical PE set을 기준으로
        - intra-set convolutional reuse + psum accumulation at array level 보존
        - 서로 다른 set끼리도 data reuse + psum accumulation 가능하기 때문 (서로 다른 set에서 같은 부분을 하나의 physical PE로 합칠 때)
    - physical PE는 여러 개의 logical PE sets (a *processing pass*)를 실행하는데, *processing pass* 단계에서의 folding도 필요함


Analysis
- input data access energy cost (filters + ifmaps) + psum accumulation energy cost
- DRAM, global buffer, array, and RF 4개의 계층으로 나눔
- 아래 두 횟수의 합을 최소화하는 optimization (global buffer, RF, array의 크기를 바꾸며)을 통해 평가
  - Reuse at each level : 해당 level 및 하위 level에서 reuse된 횟수
  - Acuumulation at each level : 하위 level들로 데이터가 읽고 쓰여진 횟수
- RS Dataflow Energy Consumption
  - CONV layer에서는 RF access가 지배적 -> 상위 계층에 접근하지 않아도 되도록 잘 수정되었음
  - FC layer에서는 DRAM이 지배적이지만, CONV의 비중이 더 큼
- Dataflow Comparison in CONV Layers
  - DRAM 접근이 제일 오래걸리는데, RS/NLR이 그 횟수가 적고 WS는 많음
  - RS는 DRAM 횟수뿐 아니라 전체 4계층을 고려할 때 energy consumption 제일 적음
  - Energy-Delay Product(EDP)
    - **energy efficency를 얻기 위해 throughput을 희생하지 않았는지 측정하기 위함**
    - Active PEs 개수의 reciprocal(역수?) -> RS는 1D convolution primitives를 효율적으로 mapping하기 때문에 EDP가 제일 적음 
- FC layer에서도 RS가 제일 효율적
- Hardware Resource Allocation for RS
  - RS에서 PE 개수 늘리면 throughput이 많이 늘어나지만, PE array가 커지며 data reuse가 늘어남에 따라 energy cost는 조금만 늘어난다.
  - tradeoff between throughput and energy is **not** monotonic