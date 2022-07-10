# [FAST'19] SLM-DB: Single-Level Key-Value Store with Persistent Memory

## 1. Motivations

* LSM-tree based KV stores are optimized to support write intensive workloads
  * It has high R/W amplification and low read performance

* Recently, typical workloads has changed to have similar proportions of R/W
  * KV stores need to be optimized for both read and write workloads

## 2. Optimization point

* Improve read perfomance
* Reduce R/W amplification

## 3. Key points

* Alternate DRAM by using NVM (persistent memory)
  * No WAL
* B+-tree indexing for read performance
* Single-level LSM-tree for reducing R/W amplification
  * Selective compaction for range query

<hr>

* Question
  * How much performance difference between NVM latency + consistency overhead and DRAM latency + logging overhead?
  * Is there are ways to utilize DRAM on SLM-DB?