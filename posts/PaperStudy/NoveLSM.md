# [ATC'18] Redesigning LSMs for Nonvolatile Memory with NoveLSM

## 1. Critical problems on LSM when we use NVM

* Different in-memory and storage(persistent) form of the data
  * High (de)serialization cost
* Only in-memory data can be changed. storage data is immutable.
  * Limited memory capacity leads to frequent compaction, which increases stall time
  * Becuase memory is volatile, updates must be logged. This also increases latency, and leads to I/O RW amplification
* Adding NVM to the LSM hierarchy can increase read access latency

## 2. Key points

* persistent NVM-based memtable design
  * Immutable persistent memtable
    * DRAM(mutable) memtable is moved(via memcpy()) to NVM immutable state
    * problem: Compaction frequency does not change becasuse DRAM structure is not modified.

  * Mutable persistent memtable
    * Both DRAM and NVM have mutable/immutable memtables.
    * NVM memtable has larger size
    * When DRAM memtable is full, application uses NVM memtable without stalling for DRAM memtable compaction to complete

* optimistic pararell read


<hr>

Considering the time of publication, It seems to be it is worth because this research provides the points to be considered when we utilize NVM on LSM based KV stores
