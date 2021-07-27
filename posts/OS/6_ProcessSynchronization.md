<link rel="stylesheet" type="text/css" href="../../css/theme.css">

# [Chapter 6] Process Synchronization

## 6.1 Background

* *Race condition*
  * count++와 같은 간단한 코드도 여러개의 instruction 으로 구성되어있다.

```c
count++ :
  reg1 = count;    // 레지스터에 count값을 load
  reg1 = reg1 + 1; // 레지스터 값 증가
  count = reg1;    // count값 update
```

* process1(P1) 과 process2(P2)가 동시에 count에 접근할 때 문제가 발생 할 수 있다.
* count의 초기값이 0이고 P1과 P2가 모두 count++할 때, count의 결과(기대)는 2가 되어야 한다.
* 그러나, P1이 값을 증가 시킨 후 count를 update하기 전에 P2가 count++를 하게 된다면 결과적으로 count값은 1이 되어버린다.
* 이와 같이 여러 프로세스가 같은 데이터에 동시에 접근할 수 있을 때 execution의 결과가 data access의 순서에 따라 달라지는 것을 race condition라고 한다.
* 이러한 예측 불가능한 결과를 막기 위해 shared data와 관련된 operation을 한번에 하나의 프로세스만 할 수 있도록 하는것을 *synchronization*이라고 한다.

* * *

## 6.2 Critical Section Problem

* **Critical section**
  * 둘 이상의 프로세스가 공유하는 데이터를 access, update하는 code segment.

* Critical section 해결을 위한 3가지 조건
  * **Mutual exclusion**  
  : 상호배제. 한 프로세스가 critical section을 실행중이라면 다른 프로세스는 이 critical section을 실행해서는 안된다.
  * **Progress**
  : 어떠한 프로세스도 critical section을 실행하고 있지 않고 일부 프로세스들이 들어가기를 원하는 경우 그 중 하나가 즉시 들어가야 하며 이 과정이 무기한 연기되어서는 안된다.
  * **Bounded waiting**  
  : 프로세스가 진입 요청을 한 후 승인이 되기 전에 다른 프로세스가 critical section에 진입할 수 있는 횟수에 제한이 있어야 한다. starvation이 있어서는 안된다는 의미

* Single-core 환경에서는 concurrent만 막으면 되기 때문에 단순히 critical section에서 interrupt를 차단하여 preemption을 막기만 하면 critical section problem을 해결할 수 있다.

* 그러나 multi-core 환경에서는 parallel한 execution도 가능하기 때문에 single-core와 같은 방법만으로는 해결이 되지 않는다.

* * *

### Reference

* Operating System Concepts 10th edition
