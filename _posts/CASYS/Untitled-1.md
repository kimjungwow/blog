---
title: "CASYS :: Perforated Pages"
date: 2021-04-25 16:51:00 +0900
categories: Study
tags: 논문
---

큰 페이지 필요하다
하지만 만들기 어렵다

최근 데이터 중심 어플리케이션에서 사용되는 메모리의 크기가 증가함에 따라, address translation을 효율적으로 하는 것이 중요해지고 있다. 이를 위해 Translation look-aside buffer (TLB)의 효율성을 높이기 위해 페이지의 크기를 2MB나 1GB로 키우는 방식이 널리 사용되고 있다. 하지만 기존의 방식으로는 페이지의 일부가 할당되지 않거나 (bloating), 물리 메모리 상에서 연속적으로 할당되지 않거나 (fragmentation), 고정된 4KB 페이지가 존재하거나 (immovable page), 한 페이지 내에 여러 권한을 고려해야 하는 경우에는 해당 region에 큰 페이지를 사용할 수 없었다. 즉, 큰 페이지를 사용할 때는 translation efficiency와 memory management flexibility의 균형을 적절히 고려해주어야 한다.
본 연구는 2MB의 큰 페이지 내에 4KB의 hole pages를 표시할 수 있는 perforated pages를 제안한다. perforated page는 물리 메모리의 2MB region에 해당하는 가상 메모리의 2MB region을 나타낸다. 하지만 물리 메모리에 immovable page가 있거나 하나의 페이지 내에 여러 권한이 필요한 경우 등에는 perforated page 내에 4KB의 hole pages를 만들어 물리 메모리의 다른 4KB region과 mapping할 수 있다. 본 연구에서는 hole pages를 가리키는 page table entry (PTE) 페이지의 주소를 저장하기 위한 shadow PTE와 hole pages를 효율적으로 식별하기 위한 two-level hole bitmaps를 구현했다.