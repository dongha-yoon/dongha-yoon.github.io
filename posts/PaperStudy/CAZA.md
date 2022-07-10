# [HotStorage'22] Compaction-Aware Zone Allocation for LSM based Key-Value Store on ZNS SSDs

## 1. Motivations

* ZNS SSD
  * Space management is moved from device to host
  * Thus, application must perform zone cleaning
  * Write amplification: When cleaning a zone, if valid data remains, this data should be copied to other zone
  * To reduce WA, data with the same lifetime should be written in the same zone

* LIZA(LIfetime-based Zone Allocation) algorithm
  * Implemented in current ZenFS
  * How this algorithm predicts lifetime of data?
    * Same LSM level - same life time
     * However, LIZA fails by not accurately identifying lifetime
  * The two situations in which LIZA cannot avoid
    * SSTables have overlapping key ranges at adjacent levels of the LSM-tree can be placed in different zone
    * Compaction can invalidate SSTables across zones with different lifetime hint values.

## 2. CAZA(Compaction-Aware Zone Allocation) algorithm

* key point
  * A group of SSTables' lifetime is determined by the LSM-tree's compaction algoritm.

* Allocates a zone to SSTables that are newly created after merging SSTables with overlapping key ranges at ajacent level by considering how compaction input is selected in the LSM-tree

<hr>
