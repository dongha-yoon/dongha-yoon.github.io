# [SOSP'21] HeMem: Scalable Tiered Memory Management for Big Data Applications and Real NVM

#### Ref: <a><https://dl.acm.org/doi/10.1145/3477132.3483550></a>

## 2.Background

### 2.1 Application Memory Demands
  * 현대의 application들은 data-intensive
    * ex) web, machine learning, in-memory DB...
    * Requirements
      * large memory capacity, bandwidth
      * low memory access tail-latency

  * 위에서 언급한 application들의 memory lifetime 특성은 allocation size와 관련이 있다
    * Large memory range: Disk로부터 pre-load되는 시점부터 application의 lifetime 동안 함께하며, 크기가 지속적으로 커진다
    * Separate memory range: Query state와 같은 일시적인 object로, 빠른 시간 안에 de-alloc된다

  * Server 내에서는 작은 수의 long-lived big data app들이 돌아가기 때문에 per-application memory management가 효과적으로 작동할 수 있다.

### 2.2 Intel Optane DC NVM
  * DRAM과 같은 memory interconnect에 장착되며, DRAM과는 다르게 비휘발성, 큰 용량을 가질 수 있다.
  * 서버 소켓 하나당 3TB용량까지도 가질 수 있다
  * 단점
    * DRAM과 비교했을 때 write가 느리다
    * Media access 단위가 cache line보다 크다
    * 내구성이 DRAM보다 떨어진다
    * Memory capacity가 커짐에 따라 memory translation 성능이 미치는 영향도 이전보다 커진다
  * 따라서,
    * NVM에 write를 많이 하게 되면 application 성능이 매우 낮아질 수 있다 -> migration의 필요성
    * 4KB 이하의 small object에 random access가 느리다 -> small object는 DRAM에 계속 두는게 낫다

### 2.3 NVM Impact on Memory Management
  * 기존 방식은 page table scan을 하면서 access, dirty bit 정보를 가지고 page의 hotness를 판단했다
    * Memory capacity가 커질수록 PTE가 많아지므로 scan에 대한 overhead가 커지게 된다
    * access, dirty bit를 clearing하면 TLB shootdown을 해야한다 -> 메모리 시스템 성능 저하
  
### 2.4 NVM as a Memory Teir
  * Optane NVM이 OS에 잡히는 방식은 memory mode(MM)과 app-direct mode(AP)이다.
  * Memory mode
    * DRAM이 NVM의 캐시(direct-mapped)가 되어 L4캐시처럼 인식된다.
      * Direct mapped cache이기 때문에 캐시가 채워질 수록 confilct miss rate가 올라간다
      * miss에 의한 eviction으로 NVM에 write-back -> NVM random write -> tiered memory system 성능 저하
    * cache line size는 프로세서와 같다
    * Hardware base이기 때문에 OS에서 추가적인 컨트롤을 필요로 하지 않는다(할 수가 없다)
      * -> hotness detection과 같은 복잡한 tiering policy 적용이 불가능
  * App-direct mode
    * OS는 NVM을 하나의 물리 메모리 영역으로 인식한다
    * Software로 explicit managing이 가능함
    * NUMA memory management system을 이용할 수 있다
      * policy thread가 주기적인 asynchronous managing을 하는 방식
    * NVM을 file system으로 mount하여 사용하는 방법도 있다
      * Application이 직접 NVM control을 할 수가 있음
      * HeMem도 이 방식으로 접근을 한다

## 3. HeMem Design
  * HeMem은 user-space에서 tiered memory를 관리하는 라이브러리다
  * mmap, madvise와 같은 system call을 intercept하여 직접 처리한다
  * Migration은 DMA를 이용한다
  * hot, cold, free page를 추적하기 위해 메모리 타입(DRAM, NVM)마다 3개의 FIFO큐를 쓴다

### 3.1 Memory Access Measurement
  * 핵심은 Migration target을 효율적으로 찾는 방법이다
    * 비효율적인 page table scan없이 page의 hotness를 판단하는 메커니즘이 필요하다
    * **Asynchronous memory access sampling**
      * HeMem은 processor event based sampling(PEBS)을 이용하였다
      * Performance counter가 overflow되면 processor가 memory buffer에 record를 쓴다
        * Counter:
          * All load from NVM (MEM_LOAD_RETIRED.LOCAL_PMM)
          * All load from DRAM (MEM_LOAD_L3_MISS_RETIRED.LOCAL_DRAM)
          * All stores (MEM_INST_RETIRED.ALL_STORES)
      * Sampling frequency는 fidelity와 overhead간의 trade-off
      * PEBS thread가 PEBS buffer를 읽고 page statistic을 업데이트
        * Nimble에서는 page scanning과 migration이 같은 스레드 내에서 같이 이루어지지만 HeMem은 이것을 분리하여 page fault handling과 migration같은 오래 걸리는 task에 의한 간섭울 줄였다

### 3.2 HeMem Library Mechanisms
  * Allocation
    * HeMem이 user-level library에서 구현이 되어 있기 때문에 application으로부터 runtime-level informaion을 적은 오버헤드로 수집할 수 있다
    * system call이나 C standard library 함수를 intercept했기 때문에 application memory range내에서 메모리 관리에만 집중 할 수 있다
  * Migration
    * background에서 policy thread가 주기적으로 migration을 한다
    * page migration을 시작하면 page를 write-protected marking을 하여 migration 중에도 read를 할 수 있게 해주며 write는 migration이 끝날 때까지 기다리게 한다
    * migration이 끝나면 write protection을 푼다
  * Page fault handling
    * Migration을 위한 write protection에 의한 fault면 migration이 끝날 때까지 기다리게 한다

### 3.3 HeMem Policies
  * Memory allocation and placement
    * Ephemeral data: 짧은 시간동안 hot, 빠르게 de-alloc -> 처음부터 DRAM에 할당한다
    * DRAM이 꽉 차면 NVM에 할당한다.
    * 대부분의 프로그램이 DRAM에서 실행되게 하기 위해서 HeMem은 DRAM의 일정 부분을 free상태로 남겨둔다
    * small allocation은 커널이 관리하게 두고 large allocation만 HeMem이 직접 관리한다
      * NVM이 large access granuarity에서 더 효과적이기 때문이기도 함
  * Memory migration
    * 기본적으로 DRAM의 cold list, NVM의 hot list를 스캔하여 이들의 migration을 한다
    * NVM이 read보다 write성능이 더 안좋기 때문에 write-heavy page에 가장 높은 우선순위를 준다

### 3.4 Discussion
  * HeMem은 hot memory set이 존재하는 application을 target으로 하였기 때문에 working set의 size가 DRAM capacity를 넘어가게 되면 다른 시스템보다 큰 이점이 없다

## 5. Evaluation
  * 비교군: Memory mode(Hardware base), Nimble, X-Mem(emulation)
  * Questions
    * 각각의 HeMem design principle들이 HeMem 성능에 얼마나 영향을 주었고 다른 비교군들과는 얼마나 나아졌는가?
    * 여러 Big data application이 HeMem을 사용했을 때 runtime, throughput, latency 측면에서 얼마나 더 이점을 얻을 수 있는가?
    * NVM device 내구성에 미치는 영향은 어느정도인가



<hr>

### * 무엇보다 기존 programming interface를 intercept하는 방식으로 application programm의 수정 없이 적용할 수 있다는 점이 큰 장점인것 같다