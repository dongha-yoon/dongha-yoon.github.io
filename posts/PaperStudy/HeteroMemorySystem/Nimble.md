# [ASPLOS'19] Nimble Page Management for Tiered Memory Systems

#### Ref: <a><https://dl.acm.org/doi/pdf/10.1145/3297858.3304024></a>

## 1. Introduction
* 현대의 processing, memory system에서 hetrogeneity가 많아지는 추세
* 효율적인 자원의 관리가 필요
  * hot data는 fast memory, cold data는 slow memory에 배치
  * 이 hot, cold data migration의 오버헤드가 너무 커서 hardware의 banwidth에 한참 미치지 못한다
  * 이 논문에서 제시하는 4가지 optimziation
    * Huge page migration
    * Parallelized data copy
    * Concurrent multi-page migration
    * Symmetric exchange of pages

## 2. Background 
### 2.1 Page Management Policies and Mechanisms
* Two parts of memory manegement
  * Policy decisions
  * Mechanism
* 기존의 연구들은 migration mechanism은 충분하다고 생각하고 policy에만 집중을 했다

## 3. Native OS Support for Multi-Level Memories
### 3.1 Optimizing Page Migration Mechanisms
* Larger data sizes
  * Migration data가 클수록 software overhead가 hiding이 된다
* Multiple threads
  * multi-threaded copy로 데이터 복사에 소모되는 시간 단축
* Concurrent migration
  * 한 번에 여러개의 page migration을 하여 bottleneck을 피함
* Two-sided migration
  * 일반적으로 fast memory는 low capacity.
  * Caching과 비슷하게 hot page를 slow memory에서 fast memory로 옮기려면 victim(cold page)을 slow memory로 옮겨줘야함
  * 결국엔 두개의 page를 한번에 옮겨야 하는데 기존의 migration은 따로 옮겼다.
  * 1xtwo-sided migration의 성능이 2xone-sided migration보다 좋게 나온다

#### 3.1.1 Native THP Migration
* THP(Transparent Huge Page) migration이 훨씬 성능이 좋다.
* 기존 리눅스 커널이 THP를 migration을 하게 되면 페이지 하나가 512개의 base page(4kb)로 split된 후에 옮겨지는데 splitting과 이로 인한 TLP invalidation, shootdown으로 오버헤드가 매우 커지게 된다

#### 3.1.2 Parallelized THP Migration
* Nimble에서는 Page migration operation에서 variable-thread count based copy subroutine을 이용하여 빠른 data copy를 할 수 있게 한다

#### 3.1.3 Concurrent Multi-page Migration
* Spatial locality, prefetching을 고려했을 때 Multi-page migration이 효과를 올려줄 것으로 기대됨
* Single page migration 절차
  1. Allocate new page / Unmap old page
  2. Copy data
  3. Map new page / Free old page
* 기존에는 migration이 serialize되어 한번에 한 page만 migration (1->2->3, 1->2->3)
* 논문에서 제시하는 메커니즘은 (1,1,1) -> (2,2,2) -> (3,3,3)의 방식으로 page allocation/mapping을 동시에 한다

#### 3.1.4 Symmetric Exchange of Pages
* Hot page를 fast memory(small capacity)로 옮기면 victim page를 먼저 빼줘야 한다 -> 2번의 migration필요
* 그런데 2번의 migration동안의 alloc/map/unmap/free를 합쳐 overhead를 최소화 할 수 있다
* fast - slow memory간의 page exchange이기 때문에 미리 할당된 페이지에다가 mapping만 바꿔주면 된다
* 이 과정에서 exchange를 위한 temporal space가 필요한데, CPU register를 이용하여 추가적인 temporal page allocation없이 exchange를 할 수 있다

### 3.2 Optimizing Page Tracking and Policy Decisions
* Policy는 충분히 general하고 real-worl scenario에 잘 대처해야 한다
* Page replacement algorithm에서 가장 중요한 것은 identifying hotness
* Linux는 active/inactive list로 hotness를 구별한다
* 이 논문에서도 이와 비슷한 방식으로 hardware access bit와 software access bit를 이용하여 hotness를 판단한다.
  * 이 방식은 Page table 전체를 스캔하는 방식인데, page size가 작고 메모리 space가 커질수록 scan해야할 pte가 많아지기 때문에 상당한 오버헤드를 가진다
  * 그래서 뒤에 evaluation에서도 base page(4kb)에서는 optimization으로도 커버가 힘든 상당한 벽이 있다고 말함