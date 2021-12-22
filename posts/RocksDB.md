# RocksDB
Ref: https://github.com/facebook/rocksdb/wiki/RocksDB-Overview

## 1. Overview

* persistent key-value store
* Key, value들은 arbitrarily-sized byte stream
* user-specified comparator function으로 key가 ordered됨
* pure memory, flash memory, hdd, remote storage 등 다양한 production env에 적용이 될 수 있다,.

## 2. Assumptions and Goals

### Performance

* design point : it should be performant for fast storage and for server workloads
* 효과적인 point lookup과 range scan을 해야한다.

## 3. High Level Architecture

### Basic Structure

* memtable :  in-memory data structure
* sstfile : memtable이 꽉 차면 스토리지의 sstfile로 flush된다.
* logfile : sequentially-written file on storage. Crash recovery를 위해 존재하는듯 하다.
* write를 하면 기본적으로 memtable에 삽입되고 선택적으로 logfile(WAL)에 기록이 된다.

## 4. Features

### Column Families

* DB 인스턴스를 나누는 logical partition
* 각 K-V pair는 cf와 1:1로 매칭된다
* 각 cf에서 atomic write가 지원된다
* cf 전체에서 consistent한 view를 가진다
* 다른 cf를 독립적으로 구성할 수 있음
* cf를 추가하고 삭제하는 작업이 즉각적으로 이루어지며 빠르다

### Updates

* Put : single k-v pair를 db에 삽입. 이미 key가 존재한다면 기존 value에 overwrite
* Write : multiple k-v pair를 atomically insert, update or delete.
  * Single write call에서 하나라도 insert가 되지 않으면 모든 insert가 취소된다.
* DeleteRange : range에 해당하는 모든 key를 삭제

### Gets, Iterators and Snapshots

* Get, MultiGet : fetch k-v pair from DB
* Iterator : DB에서 range scan을 위해 제공하는 api. 모든 데이터는 logically arranged in sorted order.
  * Iterator creation시점에 database의 point-in-time veiw가 생성되기 때문에 iteration의 일관성이 보장된다.
* Snapshot : point-in-time view를 생성. Get와 Iterator는 specified snapshot에서 data를 read할 수 있다.
* short-lived, foreground scan은 iterator, long-running,background scan은 snapshot을 쓰는게 가장 좋다
* Iterator는 file의 reference count를 유지하기 때문에 Iterator가 release되기 전까지는 파일이 삭제되지 않는다.
* Snapshot은 파일 삭제를 막을 수 없다. 대신 compaction process가 Snapshot의 존재를 이해하고 Snapshot에서 볼 수 있는 key를 삭제하지 않는다.
* Snapshot은 not persistnet하다; reloading을 하게 되면 기존의 Snapshot들은 모두 날아간다.

### Transactions

* Pessimistic (TransactionDB), Optimistic (OptimisticTransactionDB) 모드가 있다.
* 기본적으로 BEGIN, COMMIT, ROLLBACK api를 가지고 있으며 RocksDB가 conflict checking을 하는 동시에 data modification이 가능하다.
* conflict가 없을 때만 write가 이루어지며,  commit이 될 때까지는 다른 스레드가 transaction의 change를 확인할 수 없다.

### Prefix Iterators

* 대부분의 application은 pure-random scan을 하기보다는 key-prefix 내에서 scan을 한다
* RocksDB는 이러한 점을 이용하여 application이 key-prefix based filtering을 활성화 하여 Iterator가 Bloom filter를 통해 효율적인 scan을 할 수 있게 해준다.

### Persistence

* write operation(Put, Delete, Merge)을 할 때 선택적으로 WAL(write ahead log)에 insert되는데, restart시 WAL에 기록된 모든 transaction을 다시 처리한다.

### Data Checksuming

### Multi-Threaded Compactions

### Compaction Styles

* Level style compaction (default)
  * optimizes disk footprint (space amplification)

* Universal style compaction
  * optimizes total bytes written to disk (write amplification)

* FIFO style compaction
  * level 0에서 가장 오래된 file을 드랍한다.

* 개발자가 custom policy를 직접 개발할 수 있다

### Metadata Storage

* 모든 DB state change를 manifest log file에 기록한다.

### Avoiding Stalls 

* Stall 발생 조건
  * Background compaction thread는 memtable(memory) 데이터를 storage file로 flush한다.
  * 모든 background compaction thread가 compaction을 하고 있는 중에 burst of write가 발생하여 memtable을 빠르게 채운다면 이후의 write가 stall된다.
* 이 상황을 피하기 위해서 일정 스레드 셋을 memtable flushing을 위해 명시적으로 남겨두도록 RocksDB 설정을 한다.

### Compaction Filter

* Compaction time에 key를 processing해야하는 application이 있을 수 있다.
  * ex) TTL을 기본적으로 지원하는 DB는 expired key를 제거함
* Application-defined Compaction-Filter를 설정하면 된다

### ReadOnly Mode

* Readonly mode에서는 lock이 걸리지 않기 때문에 read performance가 많이 향상된다.

### Database Debug Logs

### Data Compression

* 다양한 압축 알고리즘을 지원한다.
  * lz4, zstd, zlib 등

### Full Backups and Replication

### Support for Multiple Embedded DBs in the same process

* Single server process가 여러개의 RocksDB instance를 동시에 실행할 수 있다

### Block Cache

* RocksDB는 Read 성능을 위해 LRU block cache를 사용한다
* Block cache는 압축/비압축 block cache로 나누어져 있다.

### Table Cache

* Open file descripter로 구성된 캐시
* 이 Fd는 sstfile을 위한 것이다.

### I/O Control

* 다양한 I/O 옵션이 있다...

### Stackable DB

* RocksDB는 code DB kernel 위에 레이어로 기능을 추가할 수 있는 built-in wrapper mechanism을 가지고 있다.
  * ex) TTL 기능을 "StackableDB" API로 구현할 수 있다.
* RocksDB 코어를 건드리지 않기 때문에 modularized and clean

### **Memtables**

* Pluggable Memtables
  * Memtable의 default implementation은 'skiplist'
  * skiplist: sorted set으로, 워크로드가 range-scan으로 write를 interleave할 때 필요함.
  * 그러나 어떤 app들은 interleaved write 또는 range-scan을 하지 않는다.
    * 이런 app에서는 sorted set의 performance가 optimal하지가 않다.
  * Memtable은 세 가지 종류가 있다
    * skiplist/vector/prefix-hash memtable
    * vector memtable: bulk-loading에 적합
      * 모든 write가 vector의 끝에 insert된다.
      * Flush 될 때 정렬되어서 기록된다
    * prefix-hash memtable
      * get,put,scan-within-a-key-prefix를 효율적으로 프로세싱한다.

* Memtable Pipelining
  * RocksDB는 db의 memtable 수를 임의로 설정할 수 있다.
  * Memtable이 꽉차게 되면 immutable하게 바뀌며 background thread가 flushing을 시작한다
  * 그러면서 new write는 새로 할당된 memtable에서 계속 진행된다.
  * 이러한 flush, allocate pipelining으로 RocksDB의 write throughput이 향상되었다(특히 slow storage device에서)

* Garbage Collection during Memtable Flush
  * Memtable이 storage로 flush되면 inline-compaction process가 실행된다
  * Garbage들은 compaction과 같은 방식으로 제거가 된다