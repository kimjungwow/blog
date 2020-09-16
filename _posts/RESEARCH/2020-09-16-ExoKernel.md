---
title: "Research :: ExoKernel"
date: 2020-09-16 18:58:00 +0900
categories: Study
tags: Study OS 논문
---

# ExoKernel : Exterminating abstractions from kernel (수업)

Idea paper: How to create a right abstractions to expose HW

Big idea: OS의 목적 = allow for secure multiplexing for HW; library OS implement system interfaces and policies

Exokernel의 장점
- Simpler
- **Standardization**

Interface는 abstraction이 아니다. 예를 들면, pipe와 file은 same interface 가지지만, 서로 다른 abstraction이다.

Problems to solve : abstraction으로 machine 관련 정보를 숨기면 app의 성능/자유도 제한됨. Fixed high-level abstraction이 app과 HW resource 사이의 유일한 인터페이스이기 때문이다.

Exokernel은 OS 디자인을 완전히 바꾼다. rudimental. 반면 SPIN은 extension을 kernel과 연결 짓는다. (linux kernel module 처럼)

Exokernel : end-to-end argument. 성능 향상이 주 목적이면 OS가 아닌 app이 권한을 가지는 것이 맞음. OS는 generality를 위한 것이지만, app은 자신의 목적을 더 잘 알기 때문임.
즉 Exokernel에서 kernel은 최소한의 기능(protection과 관련된 resource management)만 담고, abstraction은 담지 않음. **Separate protection(kernel) from management(app)**
이를 위해서는 HW를 app에 직접 노출시켜야 한다.

Library operating system은 필요한 app에 한해서 abstraction 제공 (not generally)

**하지만 app을 개발할 때 abstraction까지 고려하는 것은 너무 복잡해서 monolithic kernel이 더 많이 사용됨**

Exokernel의 메커니즘 : protect and manage resource -> Secure binding with capabilities

Secure binding 은 authorization과 다르다. resource에 접근하는 두 경우는 bind time과 access time
- Bind time에는 verify with authorization (Bind time에만 검사하고, 사용하게 허락해줌.) -> 예를 들면, `open()`에서만 권한 확인하고, `write()`에서는 사용하게 둔다.
- Access time : let it use
- 예 : page table mapping
    - Bind time에는 vaddr과 paddr 연결을 허락해줌
    - Access time에는 검사하지 않고 사용하게 둠

Secure binding: Memory
app은 read-only, LibOS가 관리?

하지만 user-level에서 네트워크 패킷을 demultiplex하려하면 불필요한 과정이 들어가서 성능이 안 좋아진다.
`NW -> kernel -> Destination process` vs `NW -> kernel -> Demux process -> kernel -> Destination process`
그래서 Application-specific safe handler (ASH)는 커널로 packet filter를 다운로드한 뒤, 커널 레벨에서 원하는 방식으로 패킷을 처리한다.
- Bind time : exokernel이 libOS가 packet filter를 설치하도록 함
- Access time : libOS는 kernel 방해 없이 packet filter 사용

kernel이 HW를 직접 노출하는 경우, 어떻게 리소스를 공유할지 고민해야 한다. -> **performance isolation**
예를 들어 한 app이 모든 storage bandwidth를 사용한다면?

LibOS는 Exokernel에 접근하기 위해 syscall 사용 + each process마다 LibOS 가짐
