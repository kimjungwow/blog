---
title: "Research :: InstructionSetPRinciples"
date: 2020-03-31 17:17:00 +0900
categories: Study
tags: Study Architecture
---
H&P textbook Appendix A를 읽고 정리했다.  

- 레지스터는 메모리보다 빠르며, stack architecture와 다르게 정해진 순서로 instruction을 실행하지 않아도 된다. 또한 레지스터에 변수를 저장할 수 있어 코드가 짧아진다.
- Big-endian : "big end" (most significant value in the sequence) is stored first (at the lowest storage address).
반면 "small end" = least significant value
- align : 정해진 byte나 word대로 데이터가 있지 않으면 misalign. 예 : 3번째비트부터 8비트의 데이터가 저장된 경우
  - misalign을 허용하더라도 여러 개의 aligned memory를 받게 되어서 결국 aligned만 허용하는 것이 빠르다.

- addressing modes : how addresses are specified by instructions
  - constraint와 register로 계산한 address = *effective addres*
  - Displacement : `Add R4,100(R1)`
  - Immediate : `ADd R4,#3`
  - Register indirect : `Add R4, (R1)`

- *packing decimal* or *binary-coded decimal* : 각 4비트가 0~9중 하나를 나타내며, 2개의 수가 모여 한 바이트를 이룸
  - 십진수 `0.10`은 십진수로 나타내면 쉽지만, 이진수로 나타내면 무한소수이다. 이 경우 *packing decimal*로 나타내는 것이 편하다.
- *unpacked decimal* : numeric character strings

|Operator Type|Examples|
|------|---|
|Arithmetic and logical|Integer artihetic and logical operations : add, subtract, and, or, multiply, divide|
|Data transfer|Loads-stores (move instructions on computers with memory addressing)|
|Control|Branch, jump, procedure call and return, traps|

- control flow instructions
  - *PC-relative* : 주로 현재 위치와 가깝고, 표시하는데 필요한 비트 수가 적어서 좋음. 또한 load된 곳과 독립적으로 코드를 실행하게 해줌 = *position independence*
  - return, indirect jumps처럼 목적지가 run-time에 정해지는 경우 PC-relative address가 아닌 dynamic address 사용
    - run-time에 주소가 알려지는 예
      - `case`, `switch` : 너무 클 수도 있음
      - `virtual functions` or `virtual method` : argument의 타입에 따라 다르게 불릴 수 있음
      - `High-order functions` or `function pointers` : function이 arguments로써 전달될 수 있음
      - `Dynamically shared libraries` : library가 프로그램에 의해 invoked되었을 때만 runtime에 loaded and linked됨
  - MIPS는 condition register를 이용해 specify branch condition
  - 대부분이 simple test 혹은 0과 비교
  - Procedure Invocation Options
    - procedure은 return address나 value 등을 저장해야 함
    - *Caller saving* : calling procedure가 레지스터를 저장해야 함
    - *Callee saving* : callee가 저장
    - procedure와 그것이 부른 다른 procedure가 같은 변수를 필요로 할지 파악하기 어려우므로, compiler는 단순히 caller가 다 저장하는 caller saving을 선택

- Encoding an Instruction Set
  - Variable : 모든 operation에 많은 addressing mode 가능해서 코드가 짧아지지만, 다양한 길이의 instruction 가능
  - fixed : 통일된 길이의 instruction 가능하지만, 코드가 길어짐. performance 좋아짐
  - hybrid : 둘의 장점을 합침

- compiler
  - global common subexpression elimination
  - 같은 값 계산하는 두 instance 찾아서 두 번째 계산부터는 첫 번쨰 계산의 결과를 이용. 이 때 첫 결과는 메모리가 아니고 레지스터에 저장되어야 빨라서 최적화의 의미가 있음. 하지만 register allocation은 최적화의 나중 단계에 진행되서, phase ordering이 어렵다.
  - register allocation은 graph coloring와 비슷해 heuristic algorithm으로 풀 수 있지만, integer type을 위한 16개 이상의 레지스터 + floating point type을 위한 별도의 레지스터가 있어야 잘 동작한다.
  - how high-level languages allocate data
    - stack : local variable. 주소는 stack pointer에 상대적인 주소로 표현되며, array형 데이터보다는 single variable(scalar).
    - global data area : statically declared object (global var, constant - 주로 array)
    - heap : dynamic objects. 보통 scalar는 아님.
    - register allocation은 stack이 제일 적절. 또한 aliased variable은 register에 등록 못 함. aliasing이란, 메모리의 데이터에 다른 symbolic name을 이용하여 접근할 수 있는 경우. 예 : `int arr[2]={1,2}; int i=10; arr[2]=20;`시 실제 i의 주소에는 20이 들어감. 하지만 컴파일러는 i의 값을 레지스터에 저장해 10임을 앎?
  - Some instruction set properties
    - Provide regularity : operations, data types, addressing modes 가 각각 orthogonal. orthogonal이란 independent하다는 뜻. 하나의 addressing mode는 어떤 operation과 사용해도 같은 값 나오는 것이 orthogonal
  - Provide primitives, not solutions
    - 특정 언어에만 적절한 기능은 다른 언어에는 부적절
  - Simplify trade-offs among alternatives
    - 예 : 메모리가 아닌 레지스터에 저장하기를 몇 번 이상 참조될 때 해야 이득인가
  - Provide instructions that bind the quantities known at compile time as constatns 
    - compile time에 정해지는 것을 run-time에 interpret하면 compiler는 안 좋아함
  - Multimedia Instructions (SIMD?)
    - Vector computers include *strided addressing* and *gather/scatter addressing*.
      - *strided addressing* : 일정한 길이를 skip. sequential addressing = *unit stride addressing*
      - *gather/scatter addressing* : find their address in another vector register

- MIPS64 : load-store architecture
  - Registers
    - 32개의 64-bit General-Purpose Registers (R0...R31)
    - 32개의 Floating-Point Registers
    - R0의 value는 항상 0
  - Instructions
    - 32 bits. 앞의 6bit는 opcode
    - Immediate는 16bits
    - Control Flow Instructions
      - How to specify destination address 
        - PC-relative (replace lower 28 bits of PC)
        - address in a register
    - Compare Instruction
      - Set True(1)/False(0) to result register
    - Floating-point operation
      - *paired single* operation : 하나의 64-bit floating-point register에 앞 뒤로 2 개의 32-bit floating-point가 있다고 생각하고 진행
    - load 바로 뒤에 ALU에서 메모리부터 읽어온 값을 필요로 하는 경우, bypassing으로 해결 불가
    - pipeline interlock이 hazard감지하여 stall 삽입

## Branch prediction
- PC가 target address로 바뀌는 지에 따라 taken/untaken
- addr이 계산될때까지 정확한 PC 모름. compare는 ID에서 확인됨
- How to handle branches
  - freeze or flush : PC+4 fetch했다가, taken이면 그 I를 무시함
  - branch 다음 I를 IF 두 번: 매 branch마다 stall 1 발생
  - predict taken scheme : target이 EX에서 정해진 뒤 예측해도 빨라지지 않으니 예측의 의미가 없음 (HW가 지원되면 미리 target address 알 수 있음)
  - delayed branch :일단 다음 I fetch후 taken이면 target으로감, 하지만 branch의 바로 뒤PC가 역시 branch면 애매해짐. canceled branch : branch slot의 예측이 부정확하면 no op으로 바꿈
- Dynamic branch prediction :Branch history table / Branch prediction buffer
  - 1-bit prediction : 거의 taken인 경우 not taken 마다 miss두번

# C.3 How is Pipelining Implemented?

- Stage별 특이점
  - ID : decoding과 reading registers 동시에 가능.
    - 이것들이 MIPS instruction format에서 정해진 위치에 있기 떄문
  - EX : effective address 계산, execution cycle을 하나의 stage에서 가능 -> 둘 다를 요구하는 I는 없기 때문
  - MEM : nonpipelined에서는 MEM stage에서 PC에 적지만, pipelined에서는 MEM과 IF가 동시에 PC 변경 가능하다. 따라서 target address를 right to left로 옮김
  - WB : register file에 적음
- pipeline latches = pipeline registers
- *instruction issue* : ID -> EX로 instruction이 이동. data hazard는 ID에 파악되며, 마찬가지로 forwarding이 필요한지도 ID에 파악됨
- hazard, forwarding 파악하기 위한 다른 방법 : EX와 MEM 시작 전에 operand를 사용ㅎ는지 파악
- 각 I의 destination register와 뒤, 뒤뒤 I의 source register만 비교하면 됨. R0은 항상 0이므로, R0에 대한 hazard는 무시해도 됨.
- stall하려면 ID/EX pipeline register를 모두 0으로 바꾸고 (no-op), IF/ID pipeline register는 그대로 유지하도록 한다.
- 하나의 ALU를 추가하여 branch comparison과 target address 계산을 ID stage에서 진행하면 branch stall을 3 cycles에서 1 cycle로 줄이게 된다. (기존에는 MEM stage에 target address를 PC로 전달)
  - branch target과 condition을 빨리 판단해서 1 cycle 줄어듦
  - target address를 판단한 cycle과 같은 cycle에 보내서 1 cycle 더 줄어듦
- jump에서는 주어진 26비트를 shift 2한뒤 PC에 더함(word 기준이어서 4 곱함?)

# C.4 What Makes Pipelining Hard to Implement?

- Exception : pipeline에서 진행되던 I들을 abort. 하지만 CPU의 state를 바꾸는 것이 안전한지 파악하기 어려움
- Exception 발생시 HW에서 어떻게 해야하는가?
  - *Asynchronous versus Synchronous*
    - *Synchronous* : 프로그램이 같은 data, memory allocation일 때 event 발생.
    - *Asynchronous* : CPU/memory 말고 외부의 장치 때문에 생긴 event, 현재 진행중인 I를 끝낸 뒤 handle해도 되서 대체로 편함
  - *User requested versus coerced*
    - 유저가 의도한 exception : *User requested*. 예측 가능
    - *coerced* : 유저가 의도하지 않음. 예측하기 어려움
  - *User maskable versus user nonmaskable*
    - user task에 의해 masked/disabled 될 수 있으면 user maskable. 이 때 mask는 단순히 HW가 exception에 응답하는지 아닌지를 통제함.
  - *Within versus between instructions*
    - event가 I 도중에 일어나는지, I와 I 사이에 일어나는지로 구분. I 도중에 일어나면 I들이 멈춘 뒤 재시작해야하서 더 다루기 어려움
  - *Resume versus terminate*
    - 프로그램이 항상 interrupt 이후에 멈추면 *terminating* event
    - interrupt 이후에도 프로그램이 계속 실행되면 *resuming* event
    - *termination* event가 더 다루기 쉬움. handling 후에 CPU가 그 프로그램을 restart할 필요 없기 떄문


