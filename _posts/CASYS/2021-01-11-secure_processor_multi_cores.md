---
title: "CASYS :: Secure Processor"
date: 2021-01-11 16:51:00 +0900
categories: casys
tags: 논문
---

## Reducing the Memory Bandwidth Overheads of Hardware Security Support for Multi-Core Processors

Secure processor provides **confidentiality** and **integrity**.  

**confidentiality** : 허가된 자만 접근할 수 있게
- Requires data encryption for all off-chip data
- Requires data decryption when fetching off-chip data into on-chip cache
- Counter-mode encryption : techinque to eliminate complex computation for decryption off the critical path
  - Overlap OTP (one-time pad) construction with off-chip memory access
  - Use a separate counter cache to hold frequently used counters

**integrity** : 특정 데이터를 보호해 데이터를 정상인 상태로 유지하는 성질
- MAC (Message Authentication Code) : a keyed hash value of the corresponding cacheline
  - When cache miss occurs, generate another MAC using the normal data and MAC
  - Compare two MAC values for any changes while the cacheline was residing in the memory
- Use Merkle tree to avoid splicing and replay attacks
  - Verify a cacheline through MAC comparison
  - Verify MAC value itself by comparing the hash value of MAC to the parent node in the tree
  - Root must reside in on-chip storage
  - Cache the Merkle tree to reduce on-chip memory bandwidth

Baseline Secure Processor
- Use an unique address and counter for the cacheline to generate OTP
- BMT (Bonsai Merkle Tree) only hashes counter entries instead of MAC
  - A counter entry covers a 4KB page
  - the height of BMT is significantly shorter than the Merkle tree for MAC

Data Miss Handling
- If a counter is not in the counter cache,
  - fetch it from the external memory
  - and verify the integrity of counter using the BMT

Multi-Core Scalability of Secure Processors
- Goal : To improve the performance of secure processors as close as to that of non-secure processors
  - Multi-cores have lower off-chip memory bandwidth per core
- Caching strategy
  - **LLC (Last Level Cache) should be shared by all types of data, counter, hash and MAC blocks**
    - MAC blocks aren't cached in single-core
- Memory scheduling
  - **Prioritize memory requests based on block types**
    - Hash and MAC blocks are used for integrity checking
    - integrity checking can be postponed : core can continue to execute instructions during the integrity checking process
    - Use *DC priority* scheme : high priority to data and counter blocks
    - Use threshold counter to schedule non-priority requests sometimes
  - Improve row buffer hit rates in the DRAM by **locating MAC (M) in the same row as the corresponding data (D)** : *DM coupling*
- Type-aware **dynamic cache insertion**
  - Use MRU, BIP, or half-MRU insertion
