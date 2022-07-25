# [HotStorage'22] Lifetime-leveling LSM-tree compaction for ZNS SSD

## 1. Motivations

* Compaction problems in LSM-based KV stores with ZNS SSD
  * Space amplification (Long-lived SSTs)
    * 
  * Write amplification (Short-lived SSTs)


## 2. LL (Lifetime leveling) compaction algorithm

* Key principles
  1. Allocates dedicated zones for each level
  2. (For avoiding long-lived SSTs) Each compaction must involve all the lower-level between the current CP(compaction pointer) and next CP
  3. Store short-lived SSTs at separated zones (or DRAM)
  