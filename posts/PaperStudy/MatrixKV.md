# [ATC'20] MatrixKV: Reducing Write Stalls and Write Amplification in LSM-tree Based KV Stores with Matrix Container in NVM

## 1. Challenges on LSM-tree based KV stores

* Write stalls
  * LSM구조에서 L0레벨은 DRAM에서 flush된 Memtable이 그대로 저장되기 때문에 정렬이 되어 있지 않다.
  * Compaction은 merge sort 방식으로 하는데, L0이 정렬된 상태가 아니기 때문에 여기서 오는 overheadr가 크다.
* Write amplification
  * WA = n*AF (n: # of levels, AF: Amplification factor)
  
## 2. Previous work (NoveLSM, ATC'18)

* NoveLSM은 NVM을 DRAM을 대체하여 Memtable의 사이즈를 늘려 NVM에 저장했다.
* Memtable크기가 커진 만큼 flush가 더 적게 일어나 compaction횟수가 줄어들게 된다.
* 하지만 크기가 커진 memtable이 flush되면 compaction할 데이터의 양이 그만큼 늘어나게 되고, 앞에 설명한 write stall의 근본적인 원인인 L0-L1 compaction에서 오는 overhead가 더욱 커지게 된다

## 3. Proposed Solution

* Multi-tiered structure(DRAM-NVM-SSD)
  * NoveLSM은 DRAM을 NVM으로 대체하여 memtable 사이즈를 늘림
  * MatrixKV는 L0를 NVM에 저장하여 더 효율적으로 NVM을 utilize함
    * Memtable (DRAM)
    * L0 (NVM)
    * L1 ~ Ln (SSD)

* Solutions for:
  * Write stall problem
    * Matrix container (해당 논문 Figure 6 참고)
      * L0를 NVM에 배치함으로 조금 더 fine-grained한 processing을 할 수 있게 됨
      * reciever와 compactor로 구성되어 있다
      * reciever
        * flush된 memtable을 rowtable로 저장한다 (unsorted)
      * compactor
        * reciever에서 rowtable이 적당히 쌓이면 compactor로 변환시키고 새로운 reciever를 할당한다
        * compactor에서는 fine-grainded하게 row단위로 sort한다
        * 이렇게 하면 comlum단위로는 key가 overlap되지만 row단위로는 잘 정렬이 됨
    * Column compaction
      * Matrix container내 compactor에서 fine-grained하게 정렬을 해두었기 때문에 column단위별로 빠른 compaction이 가능해짐

  * Write amplification problem
    * Write amplification문제는 LSM-tree의 깊이, AF와 관련이 있는데, L0의 크기를 키워주면 AF를 바꾸지 않고도 tree의 depth를 줄일 수 있다.
    * L0를 NVM에 두었기 때문에 성능 저하를 최소화할 수 있다
  * Improving read performance
    * Matrix container에서 row단위로는 key가 overlap되지는 않지만, column단위로는 될 수 있기 때문에 각각의 row를 binary search해야 했다.
    * Cross-row Hint search
      * 말 그대로 row간에 힌트를 주어 next row에서 현재 키 값보다 크지 않은 키를 pointing하여 빠른 searching을 할 수 있도록 함