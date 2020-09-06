---
title: "Research :: The UNIX Timesharing System"
date: 2020-09-06 21:05:00 +0900
categories: Study
tags: Study OS 논문
---

The UNIX Timesharing System. Dennis M. Ritchie and Ken Thompson. Communications of the ACM 17(7), July 1974.
를 읽고 기억할 내용을 적었다.

0. Abstraction

- UNIX가 제공하는 것
    1. a hierarchical file system incorporating demountable volumes
    1. compatible file, device, and inter-process I/O
    1. the ability to initiate asynchronous processes
    1. system command language selectable on a per-user basis
    1. over 100 subsystems including a dozen languages

1. Introduction

- UNIX는 사용하기 편리하고 단순한 것이 장점이다.

2. Hardware and Software Environment

- UNIX SW는 어셈블리가 아닌 C로 쓰여서 이해/수정이 쉽고 기능이 많다.

3. The File System

- Directory
    - 디렉토리는 파일 이름과 파일을 이어주는 것이다. 
    - `link`라 불리는 directory entry란 단순히 이름 + 파일로 향하는 포인터로 이루어졌다. 즉, 파일은 특정 디렉토리 안에 존재하는 것이 아니다. (디렉토리와 독립적임)
    - `link`는 nondirectory file에 대해서만 생성 가능

- Special Files
    - I/O device마다 하나의 special file과 관련됨.
    - `dev` 디렉토리에 있음
    - paper tape에 입력하려면, `dev/ppt`라는 파일에 쓰는 방식이다.
    - 이처럼 I/O device를 파일로 취급하면 (1) 파일과 device를 비슷하게 생각해 파일 이름을 파라미터로 받는 프로그램은 device name도 받을 수 있으며, (2) special files은 regular files와 같은 방식으로 보호할 수 있다.

- Removable File Systems
    - `mount` system request는 hierarchy tree의 leaf (ordinary file)을 whole new subtree (removable volume 속 hierarchy)로 대체하는 것이다. root를 제외한 파일 시스템의 일부는 다른 device에 있을 수 있어서 mount가 필요하다.
    - 서로 다른 device끼리 `link`는 불가능 -> unmount시 link도 일일이 지울 필요 없애기 위함

- Protection
    - 7개의 비트 중 6개가 owner/다른 사람의 rwx
    - 1개는 현재 유저가 해당 파일을 실행했을 때 파일의 owner와 같은 권한을 가지는지를 나타냄 : set-user-ID bit

- I/O Calls
    - `create` : 파일이 존재하면 길이 0으로 truncate

4. Implementation of the File System

- `open, create` system call은 file name을 사용해 file descriptor를 얻는다. file descriptor를 인덱스로 이용해 system table에 접근하면 device, i-number 등 파일에 대한 정보를 얻을 수 있다.
- UNIX의 i-list : hierarchy에 상관 없이 linear list만 검증하면 되므로 좋음.
- 한 파일이 여러 유저에 의해 linked 가능 -> 파일의 용량은 어떤 유저에게 charged 되는가? 공평하게 하기 위해 UNIX는 아무에게도 charge하지 않음.

5. Processes and Images

- `image` : a computer execution environment. core image, general register values, status of open files, current directory 등 포함
- `process` : the execution of an image

- Pipes
    - `pipe()`는 file descriptor를 리턴하며 interprocess channel인 pipe를 만듦. 이 pipe에 `read, write`하여 프로세스간 소통. 이 때 pipe를 file처럼 취급

- Execution of Programs
    - `execute(file, arg1, arg2, ..., argn)`는 coda/data section을 file로 대체함

6. The shell

- `shell` : a command line interpreter. `command arg1 arg2 ... argn`으로 해석하며, `command`에 해당하는 파일을 찾는다. 없으면 `/bin` 디렉토리에서 찾는다.

- Standard I/O
    - `>, <`를 이용해 typewriter를 standard input/output로 사용하지 않을 때, `>, <`는 shell에서 argument로 따지지 않고 shell이 interpret함

- Filters
    - `|`로 구분되는 command들은 동시에(simultaneously) 실행되는데, `|` 왼쪽 command의 output이 오른쪽 command에 input으로 전달됨
    - `pr`처럼 standard input을 standard output에 복사하는 프로그램을 `filter`라 함.