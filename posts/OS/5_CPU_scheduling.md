<link rel="stylesheet" type="text/css" href="../../css/theme.css">

# [Chapter 5] CPU Scheduling

## 5.1 Basic Concepts

* **CPU scheduling의 목적 : Maximize CPU utilization!**

### CPU-I/O Burst Cycle

* Process execution은 *CPU execution*과 *I/O wait*으로 나누어진다.
* 프로그램은 실행 특성에 따라 구분할 수 있다
  * **CPU-bound program** : 적은 수의 long CPU burst
    * I/O보다는 직접적인 computation을 많이 하기 때문에 CPU 사용에 시간을 오랫동안 씀
  * **I/O-bound program** : 많은 수의 short CPU burst
    * I/O가 많기 때문에 짧은 시간동안 CPU를 이용하고 I/O대기에 쓰는 시간이 많음
* 이러한 구분이 CPU scheduling algorithm implementation에 큰 영향을 미침

### CPU scheduler

* CPU가 유휴상태(idle)가 되면 OS는 실행할 프로세스를 *ready queue*에서 골라 CPU를 할당해주어야 한다. 이 역할을 하는 것이 *CPU scheduler*다.
* Ready queue를 어떻게 구현하는가가 scheduling의 핵심. FIFO, priority, tree 등 다양한 방법으로 구현될 수 있으며 queue의 element로는 일반적으로 PCB가 들어간다.

### Preemptive and Nonpreemptive Scheduling

* CPU scheduling decision이 필요한 상황
  1. 프로세스가 running state에서 waiting state로 바뀌는 경우(ex. I/O 요청이 발생한 경우)
  2. 프로세스가 running state에서 ready state로 바뀌는 경우(ex. interrupt가 발생한 경우)
  3.스가 waiting state에서 ready state로 바뀌는 경우(ex. I/O 요청 처리를 끝낸 경우)
  4. 프로세스가 끝난 경우(termination)
* 1,4번의 경우에는 선택의 여지 없이 새로운 프로세스가 선택되어야한다. 그러나 2,4번의 경우에는 ready state로 바뀐것이기 때문에 바로 CPU를 이어서 점유할 것인가를 선택할 수 있다.
* Scheduling을 1,4번의 상황에서만 하는 경우를 nonpreemptive(비선점), 그 외의 경우를 preemptive(선점)이라고 한다.
* 정리하면 CPU를 사용하고 있지 않은 프로세스가 사용중인 프로세스의 CPU를 뺏을 수 있으면 preemptive, 불가능하다면 nonpreemptive라고 할 수 있다.
* 대부분의 OS는 preemptive scheduling을 쓴다.

* Preemptive scheduling의 단점
  * Race condition 발생 가능성
  * 복잡한 커널 구조
* 그럼에도 preemptive scheduling을 쓰는 이유
  * Nonpreemptive는 매우 단순하고 구현하기 쉬운 커널구조를 가지지만 현재 작업이 끝날 때까지 CPU를 쓸 수 없다는 특징 때문에 현대 컴퓨팅의 가장 중요한 요소중 하나인 real-time computing면에서 성능이 매우 떨어질 수 밖에 없다.

### Dispatcher

* **Dispatcher** : CPU control을 scheduling된 프로세스에 넘겨주는 모듈
  * Switching context, Switching to user mode, Jumping to the proper execution location 등의 기능을 한다.

* * *

## 5.2 Scheduling Creteria

* Scheduling performance의 척도
  1. **CPU utilization** : CPU 사용률이 높을 수록 좋다
  2. **Throughput** : 단위 시간당 완료된 프로세스 수
  3. **Turnaround time** : 프로세스 submission ~ completion까지 걸린 시간
  4. **Waiting time** <br>: 프로세스가 ready queue에서 기다린 시간의 합. Scheduling algorithm은 프로세스의 실행,I/O에 소모되는 시간에는 영향을 미치지 않는다. 프로세스가 ready queue에서 얼마나 오래 기다리는지에만 영향을 미치기 때문에 이 waiting time이 중요한 척도가 된다.
  5. **Response time** <br>: 프로세스 submission ~ 첫번째 execution까지 걸린 시간. Interactive 시스템에서 중요한 척도로 쓰인다.

* 일반적으로는 평균값으로 평가를 하지만 상황에 따라 min, max를 가지고 평가를 하기도 한다.
  * ex) 데스크탑, 랩탑과 같은 interactive 시스템에서는 response time의 평균보다 variance를 최소화 하는것이 중요하다.

* * *

## 5.3 Scheduling Algorithms

### FIFO (FCFS)

* 가장 쉬운 알고리즘중 하나.
* 먼저 온 프로세스가 먼저 끝나야 하므로 nonpreemptive. 프로세스가 종료되거나 I/O wait을 하기 전까지는 CPU를 계속 점유한다.
* 문제점
  * Convoy effect : 실행 시간이 긴 큰 프로세스가 실행중이면 뒤따라오는 작은 프로세스는 한참을 기다려야한다.

### SJF (Shortest Job First)

* CPU burst가 가장 짧은 프로세스가 우선순위를 가지는 알고리즘.
* 짧은 프로세스를 먼저 실행하기 때문에 FIFO에 비해 평균 waiting time이 감소한다.
* Preemptive/nonpreemptive를 정할 수 있다.
* 문제점
  * 미래의 CPU burst를 예측할 방법이 없다.
    * *exponential average* 방식을 이용하여 예측을 하기도 한다.
    * $ \tau_{n+1} = \alpha t_n + (1-\alpha)\tau_n $, $ \tau $ : 예측값, $t$ : 실제 CPU_burst
    * $\alpha$값을 조정하여 이전의 CPU_burst를 얼마나 반영할 것인가를 정할 수 있음.
  * Starvation problem
    * 짧은 프로세스가 계속 들어오면 긴 프로세스는 무제한 대기를 하게 되는 상황이 발생

### RR (Round-Robin)

* FIFO에 preemption을 추가한 알고리즘. 적당한 **time quantum**(time slice) 값을 정하여 이 time quantum마다 CPU를 빼앗고 다시 FIFO queue에 할당하는 방식.
* 이 time quantum이 너무 크면 FIFO와 다를 바 없고, 너무 작으면 context switch가 너무 자주 일어나 overhead가 발생한다.
* 일반적으로는 time quantum값을 10~100ms로 쓴다. Context switch에 소모되는 시간이 보통 10 $\mu$s이므로 time quantum에 비해 매우 작은 비율인것을 알 수있다.

### Priority Scheduling

* 프로세스마다 우선순위를 정하여, 이 우선순위가 높은 프로세스를 먼저 실행하는 방식.
* 선점/비선점 여부를 선택할 수 있다.
* 문제점
  * Starvation :  
  SJF는 priority가 job의 길이인 우선순위 알고리즘과 의미가 같다. 이와 같이 priority가 낮은 프로세스가 무한히 대기하게 되는 상황이 발생할 수 있다.
    * 해결책으로 *aging*이 있는데, 나이를 먹듯이 시간이 지날 때마다 우선순위를 조금씩 높여주어 CPU를 획득 할 수 있도록 하는 방식이다.

### Multilevel Queue Scheduling

* Priority와 RR scheduling을 합친 방식.
* Queue를 여러개 만들어 각각에 절대적인 priority를 부여하여 관리하는 방식이다. 프로세스 타입(real-time, system, interactive, batch 등)에 따라 각각의 queue에 할당하여 처리.
* Queue마다 다른 scheduling algorithm을 적용할 수 있다.

### MFQ (Multilevel Feedback Queue)

* 앞서 소개된 multilevle queue는 scheduling overhead가 매우 낮지만 유연성이 낮다(inflexible).
* MFQ는 프로세스가 queue간 이동이 가능하도록 설계된 알고리즘이다.
* High-priority queue에서는 너무 많은 시간을 대기하지 않도록 짧은 time quantum을 적용하고, 프로세스가 많은 CPU burst time을 소비한다면 low queue로 옮겨준다.
* Low queue에서 계속 있게 되면 starvation이 발생할 수 있기 때문에 aging의 방식으로 프로세스를 high queue로 옮길 수 있도록 한다.
* MFQ의 유동성의 원천은 많은 변수에 있다.
  * 전체 queue의 수
  * 각각 queue에 적용되는 scheduling algorithm
  * Higher-priority queue로 upgrade하는 방식
  * Lower-priority queue로 demote하는 방식
  * 프로세스가 처음 queue에 들어갈 때 어떤 queue에 들어가게 할 것인가


* * *

### Reference

* Operating System Concepts 10th edition
