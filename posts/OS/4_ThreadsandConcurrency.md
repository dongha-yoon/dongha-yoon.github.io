<link rel="stylesheet" type="text/css" href="../../css/theme.css">

# [Chapter 4] Threads & Concurrency


## 4.1 Overview
* **Thread** : Basic unit of CPU utilization.

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



<br>
<hr>

### Reference
* Operating System Concepts 10th edition