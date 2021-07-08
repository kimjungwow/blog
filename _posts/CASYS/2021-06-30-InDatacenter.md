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
![table1](https://imgur.com/Xbe5xCB.png)

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

![fig1](https://imgur.com/l0HO8Ob.png)

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

![fig2](https://imgur.com/l0HO8Ob.png)

![fig3](https://imgur.com/3jiPlbf.png)

P6 : Figure 2 설명
- datapath가 TPU die의 2/3 차지함
  - unified buffer는 1/3
  - matrix multiply unit은 1/4
- control을 위한 공간은 겨우 2%만 차지함

P7 : TPU instruction은 PCIe bus를 통해 전달되기 때문에, TPU instruction은 (단순한 instruction인 RISC가 아닌 다소 복잡한) CISC instruction을 따름. CISC instruction은 cycles per instruction (CPI)가 10~20으로 느리며, 중요한 instruction은 다음과 같음
1. Read_Host_Memory : CPU host memory의 data를 unified memory에 적음
2. Read_Weights : Weight memory의 weights를 Weight FIFO에 적어 Matrix Unit의 input으로 사용함
3. MatrixMultiply/Convolve : B X 256 input과 256 X 256 matrix를 곱해 B X 256 output matrix 구함
4. Activate : ReLU, Sigmoid, Pool 등
5. Write_Host_Memory : unified memory의 data를 CPU host memory에 적음

P8 : 다른 instruction 설명

P9 : TPU는 instruction execution latency를 숨기기 위해 4-stage pipeline 사용.
- Read_weight는 addr만 보내면 종료 가능 (weight이 weight memory로부터 도착하기 전에)
- MatrixMultiply는 weight와 activation이 준비되지 않았으면 stall함

P10 : 한 stage가 최대 수천 cycle 차지 가능 -> 예를 들면, matrix multiplication이 시작하기 전에 이전 layer가 끝나야, unified buffer로부터 안전하게 읽을 수 있는 RAW pipeline stall

![fig4](https://imgur.com/yQrgJWA.png)

P11 : **unifed buffer에 접근해 read/write 하는 것보다 arithmetic operation이 빠르기 때문에, systolic array 형태로 memory access를 최소화하고 data reusage를 늘린다.**

P12 : TPU Software stack은 User Space Driver와 Kernel Driver로 구성됨. Kernel Driver는 lightweight하며, memory management나 interrupt만 담당함

P13 : User Space Driver는 나머지 담당 : user API를 TPU가 이해할 수 있는 instruction으로 바꾼다.

## 3 CPU, GPU, AND TPU PLATFORMS

P1 : benchmark 6개 재언급

P2 : 논문에서 사용한 TPU 환경

P3 : 논문에서 사용한 CPU, GPU, TPU

P4 : Haswell CPU에서 Turbo mode는 포함하지 않음 -> Turbo mode는 모든 코어가 사용되는 것이 아닐 때를 위한 것인데, 이 data center에서는 보통 모든 코어가 사용됨 (심지어 다른 datacenter job을 실행하여 idel core가 없게 하기도 함)

P5 : K80 GPU는 internal memory와 DRAM에 SECDED protection을 필요로 함

P6 : 사용되는 die의 개수가 일정하지 않기 때문에, 이 논문에서는 die의 개수에 정규화된 결과를 보여줄 것이다.

## 4 PERFORMANCE: ROOFLINES, RESPONSE TIME, AND THROUGHPUT

P1 : 세 가지 프로세서의 성능 평가할 때 high-performance computing (HPC)의 Roofline Performance model을 사용했는데, on-chip cache의 크기가 충분하지 않게 해 computation limited 혹은 memory bandwidth limited가 되도록 한다.
X축은 operational intensity (접근된 DRAM byte당 floating-point operation 개수, FLOPS/Byte), Y축은 FLOPS/sec이다. operational intensity가 충분히 크지 않으면, roofline model에서 기울어진 부분은 memory bandwidth-bound임.

P2 : 실제 operations per second와 X-Y 직선 간격이 operational intensity를 건들지 않고도 향상시킬 수 있는 성능의 양이다. 물론 operational intensity를 늘리면 성능이 더 증가하긴 한다.

P3 : Quantized NN에서는 floating point 대신 integer point operation 횟수를 셈

![fig5](https://imgur.com/lp0aPaB.png)

P4 : fig5 처럼 slanted part가 길다면 peak compute가 아닌 memory bandwidth 때문에 성능이 제한되는 것이다. fig5에서 LSTM와 MLP는 slanted part 아래에 있기 때문에 memory bandwidth가 bottlenectk이다. CNN은 slanted part가 아니므로 peak computation rate가 bottleneck이다.

P5 : fig5의 CNN1 결과를 설명함. feature depth가 얕아서, 적은 수의 MAC만 useful weight를 가지고 있고, 많은 사이클 동안 weight가 메모리부터 읽어지기를 기다려야 하는 것이 문제임

P6 : fig 6,7에 나타나듯, CPU와 GPU는 response time이 길기 때문에 TPU 보다 ceiling(X-Y선) 보다 한참 아래에 있음. inference는 유저가 직접 보는 경우가 많기 때문에, response time이 길어지면 서비스를 사용하지 않을 확률이 커지는게 문제임.

![fig6,7](https://imgur.com/Ni0zTRh.png)

P7 : MLP0에서 response time을 7ms로 제한하는 경우, CPU와 GPU는 그렇지 않은 경우보다 per die throughput (IPS)가 크게 (42%, 37%) 떨어지지만, TPU는 상대적으로 IPS가 덜 떨어진다 (80%). 
이는 TPU가 CPU, GPU처럼 일반 적인 경우에 transistor/energy 사용량을 줄이기 위한 microarchitectural features들 (예: out-of-order execution, multiprocessing 등)가 single-threaded TPU에 없기 때문이다.

P8 : TPU performance를 측정한 table 3에서 host server time은 제외되었다.
host server time에는 CPU가 application 계산을 같이 하는 시간과, CPU와 TPU가 통신하는 시간이 있다.
또한 TPU가 idle인 것이 CPU가 app 계산 중이어서인지, 아니면 input queue가 비어서 CPU 역시 idle이어서인지 알기 어렵다.

P9 : host server overhead를 포함해, die 당 성능을 비교하면 TPU가 CPU보다는 14.5배, GPU보다는 13.2배 좋다. (geometric mean으로 비교함)

P10 : P9와 달리 weighted mean으로 비교하면, TPU가 CPU보다는 29.2배, GPU보다는 15.3배 우수하다.
actual mix of program을 알고 있으면 weighted mean을 계산할 수 있다.

## 5 COST-PERFORMANCE, TCO, AND PERFORMANCE/WATT

P1 : 컴퓨터를 수천 대씩 사면 성능보다 cost-performance가 중요한데, Total Cost of Ownership (TCO) 가 중요한 지표이다. 따라서 가격 관련 지표는 공개하지 못하기 때문에, TCO당 성능 대신 전력사용량(Watt)당 성능을 이 논문에서는 지표로 사용한다.

![fig9](https://imgur.com/Gm4Jujm.png)

P2 : fig 9에서 Total은 TPU/GPU performance/Watt 계산 시 host CPU server가 사용한 전력도 포함하고, incremental은 포함하지 않는다.

P3 : fig9 수치 설명

## 6 ENERGY PROPORTIONALITY

P1 : Thermal Design Power (TDP) : 전력 공급 및 쿨링 비용을 포함함.
전력을 100% 사용하는 시간은 10%밖에 되지 않으며, energy proportionality에 따라 전력 사용량은 수행하는 업무량에 비례해야 함

![fig10,11](https://imgur.com/d1qkTJp.png)

P2 : workload utilization의 변화에 따라 C/G/TPU 성능 및 전력사용량 비교함.
fig 10 참고

P3 : TPU는 die 당 전력소모량이 적긴 하지만, energy proportionality가 낮다. CPU와 GPU는 반대로 energy proportionality가 TPU보다 좋다.

P4 : CPU server가 100% power 사용 중인 G/TPU와 함께 사용되면, CPU는 52%/69% 전력을 사용한다. 즉, 그냥 CPU server만 쓰는 것보다 CPU + 4 TPUs 하면 전력 사용량은 1.2배만 증가하지만, 80배 빠르게 CNN0 실행 가능하다.

## 7 EVALUATION OF ALTERNATIVE TPU DESIGNS

P1 : TPU 성능을 측정하기 위한 model을 만들었는데, model results와 hardware performance counters의 차이는 10% 미만으로 미미하다.

P2 : fig 11은 여러 지표를 0.25배~4배로 바꿀 때 TPU 성능을 보여준다.

P3 : fig 11 결과 설명
- memory bandwidth를 늘리는 것이 성능에 가장 영향이 크다.
- accumulator 증가와 상관 없이, clock rate를 바꾸는 것은 영향이 거의 없다.
  - 이는 MLP와 LSTM은 memory bound고, CNN만 compute bound기 때문이다. 
  - memory bound면 clock rate 올려도 차이가 없고, compute bounㅇ면 clock rate 올리면 성능 향상됨
- accumulator 증가와 상관 없이, matrix dimension이 증가하면 성능이 살짝 감소된다. (가로 세로 각각 2배 해서 한 step당 걸리는 시간이 4배 증가했지만, 사용 안 되는 matrix unit이 많은 듯)

P4 : TPU storage allocator도 개발해서, unified buffer 24MB중 14MB만 사용해도 충분하다. 따라서 더 큰 모델도 돌릴 수 있다.

P5 : 개발 시간이 더 있었으면 만들 수 있었을 hypothetical TPU die (TPU') 도 평가해봤다. TPU' 에서는 GDDR5 memory를 써서 wiehgt memory bandwidth를 5배 증가시키기 때문에, 성능도 더 좋아질 것

P6 : fig11은 host server time을 포함하지 않은 것이고, 포함한다면 TPU'의 성능향상 폭이 줄어들긴 한다.

P7 : DDR3 weight memory를 GDDR5로 바꾸려면 memory channel 수를 두 배 늘려야해 area overhead가 1.1배 있지만, GDDR5는 memory bandwidth가 더 커서 unifed buffer의 크기를 14MB까지 줄여도 괜찮다. (이것이 area overhead 해결)

P8 : TPU의 C/GPU 대비 성능 향상 재언급

## 8 Discussion

P1 : NN inference에서는 response time만큼 throughput도 중요하다는 오류 -> datacenter NN inference app은 response time이 아주 중요하다.

P2 : K80 GPU는 NN inference에 적절하다는 오류 -> GPU는 high-bandwidth DRAM과 수천 개의 스레드를 통해 high-throughput 가지지만, CPU보다 조금만 빠르고 TPU보다는 매우 느리다.

P3 : 아키텍쳐 분야에서 중요한 NN task를 무시한다는 위험 -> 요즘에는 NN 관련 논문들도 많이 나온다. 특히 CNN 관련 논문이 많은데, datacenter에서 CNN 비율은 아주 적고, 오히려 MLP와 LSTM이 많다.

P4 : NN architecutre에서 Interfences Per Second (IPS)는 부정확한 지표라는 생각 -> IPS로만 판단하는 것은 위험하고, 적절한 벤치마크가 필요하다.

P5 : Boost mode를 활성화하면 K80 GPU는 더 좋은 결과를 낼 것이라는 오류 -> LSTM1에서 실험해보니 clock rate 높여 성능은 좋아지지만 에너지 사용량도 많아진다.

P6 : CPU나 GPU를 더 효율적으로 쓰면 TPU와 비슷한 성능을 낼거라는 오류 -> 왜 이 논문에서 특정 CPU와 GPU를 선택했는지 설명

P7 : Performance counter는 NN HW에 나중에 덧붙인 것이다는 생각 -> NN HW는 성능이 중요한데, 성능을 정확히 평가하는 방법을 처음부터 알기 어려우니 106개의 performance counter를 TPU에 만들었다.

P8 : 2년간 개발해보니 TPU 성능 올리기 위한 방법은 HW upgrade 뿐이라는 생각 -> HW upgrade 없이 CNN에 더 TPU를 특화시키기 위한 노력하면 개선 가능하다.

P9 : domain-specific 아키텍쳐 디자인 할 때 아키텍쳐 역사를 무시했다는 생각 -> TPU의 중요한 요소인 systolic array, decoupled-access/execute, CISC instruction은 1980년대 초 아키텍쳐 분야에서 연구된 것이다.
- systolic array : matrix multiplication unit의 area/power overhead 줄임
- decoupled-access/execute : matrix multiplication unit이 동작함과 동시에 weight를 fetch -> double buffering인듯
- CISC instruction : instruction을 (TPU에) 전달할 때 제한된 bandwidth를 최대한 활용함
  
### ..
- Moore's law가 끝나가서, general purpose 대신 domain specific architecture를 개발하는 쪽으로 방향이 바뀜
- 짧은 기간 (15개월)에 개발한 것임을 강조함..
- PCIe로 host와 연결 가능해서, 기존의 서버에 바로 꽂아서 사용 가능함
- MLP, CNN, RNN 타겟
- instruction을 직접 fetch 하는 것이 아닌, host server가 보내면 받아오는 구조여서 심플함
- 8비트를 곱한 값은 16비트로 저장, 16비트들을 더한 값은 17,18비트 등 점점 길어질 수 있으니 accumulate해서 32비트로 저장
- Matrix Multiply unit이 systolic array이다. 행렬 계산에서 여러 번 사용되는 특정 값을 재사용 해서, latency 큰 SRAM 접근 안 해도 된다. weight 값을 미리 깔아두고, input을 세로로 뿌리고, output(psum)은 가로로 이동하며 모임.
- accumulator가 4096개인 이유 : peek performance를 위해 byte당 1350번의 operation이 수행되어야 하므로, 최소 1350개를 저장할 수 있어야해서 반올림해서 2048이 되었고, double buffering (저장된 값이 buffer에 써지는 동안, 다음 데이터가 accumulator에 써지도록)을 위해 2배 해서 4096
- Roofline Performance Model : memory bandwidth와 computation power 중 어떤 것이 bottleneck인지 따지기 좋음
- MLP, RNN은 memory bound, CNN은 copmutation bound