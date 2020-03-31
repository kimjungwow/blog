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