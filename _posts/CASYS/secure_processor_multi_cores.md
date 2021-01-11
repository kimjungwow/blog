
---
title: "CASYS :: Secure Processor"
date: 2021-01-11 16:51:00 +0900
categories:  
tags: 
---

Secure processor provides **confidentiality** and **integrity**.  

**confidentiality**
- Requires data encryption for all off-chip data
- Requires data decryption when fetching off-chip data into on-chip cache
- Counter-mode encryption : techinque to eliminate complex computation for decryption off the critical path
  - Overlap OTP (one-time pad) construction with off-chip memory access
  - Use a separate counter cache to hold frequently used counters

**integrity**
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

2.3까지