---
title: "Research :: The UNIX Timesharing System"
date: 2020-09-06 21:05:00 +0900
categories: Study
tags: Study OS
---

The UNIX Timesharing System. Dennis M. Ritchie and Ken Thompson. Communications of the ACM 17(7), July 1974.

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