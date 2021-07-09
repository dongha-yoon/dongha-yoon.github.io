<link rel="stylesheet" type="text/css" href="../../css/theme.css">

# [Chapter 4] Threads & Concurrency

## 4.1 Overview

* ***Thread*** : Basic unit of CPU utilization.

### - Motivation

* 프로세스는 한번에 여러가지 task를 수행하면 더 효율적일 수 있다
  * 웹 서버에서 여러 client의 요청을 처리: 한 번에 하나의 요청만 처리 한다면 대기 시간이 매우 길 것이다
  * 웹 브라우저에서 이미지를 띄우는 동시에 데이터를 불러오기
  * 워드 프로그램에서 display를 담당하는 thread와 텍스트 입력을 담당하는 thread로 나누어 작업

* 왜 thread를 쓸까? 똑같은 process를 새로 실행하면 안되는 것일까?
  * **Process creation time > Thread creation time**
  * Process는 메모리에 로드될 때마다 code, data, openfile 등을 다 로드 해야한다
  * Thread는 같은 Process 내에서 code, data, openfile 등을 공유하기 때문에 context(register, program counter, stack)만 새로 만들어주면 된다.

### - Benefits

1. **Responsiveness** : User interface 디자인에 효과적이다
2. **Resource sharing** : Process의 resource를 공유
3. **Economy** : 앞에서 언급 했듯이 thread의 creation이 process보다 더 빠르다. 비슷한 의미로 context switch에서도 process-process switch보다 thread-thread switch가 더 빠르다.
4. **Scalability** : CPU 코어 수가 아무리 많아도 single-threaded 프로세스는 코어 하나에서만 돌아갈 수 있다. multi-threaded 프로세스는 최대로 thread의 수만큼의 코어를 쓸 수도 있다.

<hr>

## 4.2 Multicore Programming

* Concurrency & Parallelism
  * ***Concurrency*** : 프로세스가 끝나기 전에 다른 task를 처리하는 것
    * P1 - P2 - P1 - P3 - P1 - P2
  * ***Parallelism*** : 동시에 여러 프로세스의 task를 처리하는 것
    * core1: [xxxxP1____x]
    * core2: [xxP2____xxx]
    * core3: [xxxxxP3____]

### - Programming Challenges

* 물론! multicore programming이 좋은 성능을 보이지만 그만큼 복잡한 문제들이 발생하게 된다.

1. **Identifying tasks** : 각 task들을 parallel하게 실행하려면 서로 독립적이여야 한다.
2. **Balance** : 각 task들의 size, runtime 등이 비슷해야 한다.
3. **Data splitting** : Application이 여러 task로 나뉘는 것처럼 data역시 task에 따라 분할되어야 한다.
4. **Data dependency** : 3.에서 서로 다른 task가 같은 data를 read/write한다면 dependency가 생기게 되는데 이 때 synchronizaion을 해주어야 한다.
5. **Testing and debugging** : Single-threaded에서는 execution path가 매우 단순하지만 multi-threaded에서는 그렇지 않다. 작은 변화에도 execution path가 달라지기 때문에 test/debugging이 매우 어렵다.

### - Types of Parallelism

* ***Data parallelism***
  * 데이터 전체를 subset으로 나누어 각 코어에서 똑같은 operation(task)를 실행한다
  * ex) Array[100000]의 총 합을 구하기 위해서 4개의 코어에 데이터를 0~24999, 25000~49999, 50000~74999, 75000~99999로 나누어 따로 더한후 최종 값을 구한다.
* ***Task parallelism***
  * Unique operation을 수행하는 여러 task로 나누어 실행한다.

<hr>

## 4.3 Multithreading Models

* ***User thread***
  * User space에서 만들어진 thread. Core에서 돌아가기 위해선 kernel thread에 올라야 한다.
* ***Kernel thread***
  * OS(kenel)에서 관리되는 thread

### - Mant-to-One Model

* 여러 user thread가 하나의 kernel thread에 mapping된다. 하나가 돌아가고 있으면 나머지 user thread는 block된다(parallel하게 실행될 수 없다).

### - One-to-One Model

* User thread와 kernel thread가 1:1로 mapping된다.
* User thread가 생성될 때마다 kernel thread가 같이 생성되므로 많아지면 system performance에 문제가 생긴다.

### - Many-to-many Model

* N:M mapping
* Flexible하다. 문제는 implementation이 매우 어렵다.

<hr>

## 4.5 Implicit Threading

* Application programmer입장에서 앞에서 언급된 challenge나 system특성을 모두 이해하면 threading을 구현하기에는 너무 복잡할 수 있다.
* Thrad creation, management를 application programmer가 아닌 compiler 또는 run-time library들을 통해 처리를 하는 것이 implicit threading이다.

### - Thread Pools

* 무제한적인 thread는 system resource를 낭비한다.
* ***Thread pool***: System start-up에서 여러개의 thread로 구성된 pool을 생성하여 관리한다.
* Benefits:
  * 이미 thread가 pool에 생성되어 있기 때문에 thread를 새로 만들어서 실행하는 것보다 빠르다.
  * pool에서 thread의 수을 제한하고 있기 때문에 많은 수의 thread를 지원하지 못하는 system에서 유용하다
  * Task perform과 task creation이 분리되어 있기 때문에 flexible execution이 가능하다.
    * ex) execute after a time delay, execute periodically
* Pool에서 thread의 수는 cpu, memory, expected number of concurrent request 등에 의해 heuristic하게 결정된다.

### - Fork Join

### - OpenMP

* Shared-memory 환경에서 parallel programming을 지원해준다.
* Parallel region을 설정해주면 알아서 thread를 만들어 처리한다.

### - GCD (Grand Central Dispatch)

### - Intel Thread Building Blocks

<hr>

## 4.6 Threading Issues

### - fork() and exec() System call

* 한 thread가 fork() call을 했다. 그렇다면?

```
1. 새롭게 생성되는 process는 다른 thread들을 포함한 entire process를 복사한다.
2. fork()를 call한 thread만 복사한다.
```

* 어떤 UNIX system은 위의 두가지 버전의 fork()를 지원한다.

* exec() system call은 다른 모든 스레드를 포함한 entire process를 새로운 프로세스로 교체한다.
* 대부분의 exec()은 fork()를 한 직후에 실행된다. 이런 경우에는 entire process를 복사할 필요가 없다.

### - Signal Handling

* ***Signal** : 프로세스에 사전에 정의된 특정 이벤트가 발생하면 kernel에서 알려주는 것을 말한다. Signal이 도달하면 process는 무조건 handling 해야한다.

* Synchronous signal : Signal을 일으킨 프로세스와 signal의 destination process가 같다.
* Asynchronous signal : 다른 프로세스로부터 도달한 signal

* Multithreaded program에서는 여러 경우의 수가 있어 signal의 처리가 복잡하다.

```
1. Signal을 발생시킨 thread에 전달한다.(Synchronous signal)
2. Signal을 받은 process의 모든 thread에 전달한다.
3. Signal을 특정한 thread에 전달한다.
4. process내에서 signal을 처리할 thread를 따로 할당한다.
```

### - Thread Cancellation

1. **Asynchronous cancellation** : target thread를 즉시 종료
2. **Deferred cancellation** : 주기적으로 termination 여부를 체크하여 종료

* Asynchronous는 즉시 종료를 하기 때문에 종료될 thread에 할당된 resource와 다른 thread와 공유중인 data와 관련해서 문제가 생길 수 있다.
* Deferred에서는 안전하게 resource관련 문제를 다 처리하고 끝낼 수 있다.

### - TLS (Thread-Local Stroage)

* 기본적으로 thread들은 process내의 data segment를 공유하지만 특정 thread만의 독자적인 복사본이 필요한 경우가 있다. 이것을 TLS라고 한다
* Local variable(stack)은 single fucntion에서만 쓸 수 있지만 TLS는 해당 thread내에서는 global하다.
* Local variable 보다는 static data와 유사하다고 볼 수 있다.

### - Scheduler Activation

* ***LWP*** (LightWeight Process) : Many-to-many model에서 kernel과 user thread간의 중간 다리 역할을 해주는 데이터구조.
  * User-thread 관점에서는 LWP가 user thread가 실행될 수 있도록 schedule할 수 있는 virtual processor로 간주된다.
  * 각 kernel thread에 attatch되어 있으며, OS가 kernel thread를 physical processor에 schedule하면 LWP가 user thread를 실행한다.

* ***Scheduler activation*** : User-thread library와 kernel간의 communication을 위한 방식중 하나.
  * Kernel이 application에 virtual processors(LWPs)를 할당.
  * Application이 user thread를 LWP에 schedule.
  * Upcall : Kernel은 특정 event가 발생하면 application에 알려야 한다. signal과 유사하게 thread library 내의 upcall handler를 통해 LWP내에서 처리를 함.

## 4.7 OS Examples

### - Windows Threads

* One-to-one mapping
* The general components of a thread:
  * Thread identifier (tid)
  * Register set
  * PC (program counter)
  * Stack
  * DLLs (Dynamic linked libraries)

### - Linux Threads

* Window와는 다르게 linux에서는 process와 thread를 구별하지 않고 모두 *task*의 단위로 관리한다.

<br>
<hr>

### Reference

* Operating System Concepts 10th edition
