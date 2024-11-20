# MLFQ

이 프로그램 `mlfq.py`는 이 장에서 제시된 MLFQ 스케줄러가 어떻게 동작하는지 확인할 수 있게 해줍니다. 이전과 마찬가지로, 랜덤 시드를 사용하여 문제를 생성하거나, 다양한 상황에서 MLFQ가 어떻게 작동하는지 확인하기 위해 신중하게 설계된 실험을 구성할 수 있습니다. 프로그램을 실행하려면 다음을 입력하세요:

```sh
prompt> ./mlfq.py
```

도움말 플래그(-h)를 사용하여 옵션을 확인하세요:

```sh
Usage: mlfq.py [options]

Options:
  -h, --help            도움말 메시지를 표시하고 종료합니다
  -s SEED, --seed=SEED  랜덤 시드
  -n NUMQUEUES, --numQueues=NUMQUEUES
                        MLFQ의 큐 수 ( -Q를 사용하지 않는 경우)
  -q QUANTUM, --quantum=QUANTUM
                        시간 슬라이스 길이 ( -Q를 사용하지 않는 경우)
  -a ALLOTMENT, --allotment=ALLOTMENT
                        할당 길이 ( -A를 사용하지 않는 경우)
  -Q QUANTUMLIST, --quantumList=QUANTUMLIST
                        큐 레벨별 시간 슬라이스 길이, x,y,z,... 형식으로 지정
                        x는 가장 높은 우선순위 큐의 시간 슬라이스 길이, y는 그 다음 높은 우선순위, ...
  -A ALLOTMENTLIST, --allotmentList=ALLOTMENTLIST
                        큐 레벨별 할당 길이, x,y,z,... 형식으로 지정
                        x는 가장 높은 우선순위 큐의 시간 슬라이스 수, y는 그 다음 높은 우선순위, ...
  -j NUMJOBS, --numJobs=NUMJOBS
                        시스템의 작업 수
  -m MAXLEN, --maxlen=MAXLEN
                        작업의 최대 실행 시간 (랜덤 생성 시)
  -M MAXIO, --maxio=MAXIO
                        작업의 최대 I/O 빈도 (랜덤 생성 시)
  -B BOOST, --boost=BOOST
                        모든 작업의 우선순위를 높은 우선순위로 얼마나 자주 부스트할지
  -i IOTIME, --iotime=IOTIME
                        I/O가 지속되는 시간 (고정 상수)
  -S, --stay            I/O를 발행할 때 동일한 우선순위 레벨에서 재설정 및 유지
  -I, --iobump          지정된 경우, I/O를 완료한 작업이 현재 큐의 맨 앞으로 이동
  -l JLIST, --jlist=JLIST
                        실행할 작업 목록을 쉼표로 구분된 형식으로 지정
                        x1,y1,z1:x2,y2,z2:... 여기서 x는 시작 시간, y는 실행 시간, z는 작업이 I/O 요청을 발행하는 빈도
  -c                    정확한 결과를 계산
```

시뮬레이터를 사용하는 몇 가지 방법이 있습니다. 하나는 랜덤 작업을 생성하고 MLFQ 스케줄러가 주어진 상황에서 어떻게 동작하는지 알아보는 것입니다. 예를 들어, 세 개의 작업을 랜덤으로 생성하려면 다음을 입력하세요:

```sh
prompt> ./mlfq.py -j 3
```

그러면 다음과 같은 특정 문제 정의가 표시됩니다:

```sh
Here is the list of inputs:
OPTIONS jobs 3
OPTIONS queues 3
OPTIONS allotments for queue  2 is   1
OPTIONS quantum length for queue  2 is  10
OPTIONS allotments for queue  1 is   1
OPTIONS quantum length for queue  1 is  10
OPTIONS allotments for queue  0 is   1
OPTIONS quantum length for queue  0 is  10
OPTIONS boost 0
OPTIONS ioTime 5
OPTIONS stayAfterIO False
OPTIONS iobump False

For each job, three defining characteristics are given:
  startTime : 작업이 시스템에 들어오는 시간
  runTime   : 작업이 완료되기 위해 필요한 총 CPU 시간
  ioFreq    : ioFreq 시간 단위마다 작업이 I/O를 발행
              (I/O는 ioTime 단위로 완료)

Job List:
  Job  0: startTime   0 - runTime  84 - ioFreq   7
  Job  1: startTime   0 - runTime  42 - ioFreq   3
  Job  2: startTime   0 - runTime  51 - ioFreq   4

주어진 작업 부하에 대한 실행 추적을 계산합니다.
원하는 경우 각 작업의 응답 시간과 반환 시간을 계산합니다.

작업이 끝나면 -c 플래그를 사용하여 정확한 결과를 얻을 수 있습니다.
```

이것은 세 개의 작업을 랜덤으로 생성하며, 기본 설정으로 큐의 수와 여러 기본 설정을 사용합니다. solve 플래그(-c)를 사용하여 다시 실행하면 위와 동일한 출력과 함께 다음과 같은 결과가 표시됩니다:

```sh
Execution Trace:

[ time 0 ] JOB BEGINS by JOB 0
[ time 0 ] JOB BEGINS by JOB 1
[ time 0 ] JOB BEGINS by JOB 2
[ time 0 ] Run JOB 0 at PRIORITY 2 [ TICKS 9 ALLOT 1 TIME 83 (of 84) ]
[ time 1 ] Run JOB 0 at PRIORITY 2 [ TICKS 8 ALLOT 1 TIME 82 (of 84) ]
[ time 2 ] Run JOB 0 at PRIORITY 2 [ TICKS 7 ALLOT 1 TIME 81 (of 84) ]
[ time 3 ] Run JOB 0 at PRIORITY 2 [ TICKS 6 ALLOT 1 TIME 80 (of 84) ]
[ time 4 ] Run JOB 0 at PRIORITY 2 [ TICKS 5 ALLOT 1 TIME 79 (of 84) ]
[ time 5 ] Run JOB 0 at PRIORITY 2 [ TICKS 4 ALLOT 1 TIME 78 (of 84) ]
[ time 6 ] Run JOB 0 at PRIORITY 2 [ TICKS 3 ALLOT 1 TIME 77 (of 84) ]
[ time 7 ] IO_START by JOB 0
IO DONE
[ time 7 ] Run JOB 1 at PRIORITY 2 [ TICKS 9 ALLOT 1 TIME 41 (of 42) ]
[ time 8 ] Run JOB 1 at PRIORITY 2 [ TICKS 8 ALLOT 1 TIME 40 (of 42) ]
[ time 9 ] Run JOB 1 at PRIORITY 2 [ TICKS 7 ALLOT 1 TIME 39 (of 42) ]
...

최종 통계:
  Job  0: startTime   0 - response   0 - turnaround 175
  Job  1: startTime   0 - response   7 - turnaround 191
  Job  2: startTime   0 - response   9 - turnaround 168

  Avg  2: startTime n/a - response 5.33 - turnaround 178.00
```

이 추적은 밀리초 단위로 스케줄러가 결정한 내용을 정확히 보여줍니다. 이 예에서는 Job 0이 7ms 동안 실행되다가 I/O를 발행하는 것으로 시작합니다. 이는 Job 0의 I/O 빈도가 7ms로 설정되어 있기 때문에 예측 가능합니다. 즉, 7ms마다 실행되면 I/O를 발행하고 완료될 때까지 기다린 후 계속 실행됩니다. 그 시점에서 스케줄러는 Job 1로 전환되며, Job 1은 I/O를 발행하기 전에 2ms만 실행됩니다. 스케줄러는 이 방식으로 전체 실행 추적을 출력하며, 마지막으로 각 작업의 응답 시간과 반환 시간을 계산하고 평균을 구합니다.

시뮬레이션의 다양한 측면을 제어할 수도 있습니다. 예를 들어, 시스템에 몇 개의 큐를 두고 싶은지 (-n)와 모든 큐의 시간 슬라이스 길이를 (-q) 지정할 수 있습니다. 각 큐의 시간 슬라이스 길이를 다양하게 지정하려면 -Q를 사용하여 각 큐의 시간 슬라이스 길이를 지정할 수 있습니다. 예를 들어, -Q 10,20,30은 세 개의 큐가 있는 스케줄러를 시뮬레이션하며, 가장 높은 우선순위 큐는 10ms의 시간 슬라이스를, 그 다음 높은 우선순위 큐는 20ms의 시간 슬라이스를, 낮은 우선순위 큐는 30ms의 시간 슬라이스를 가집니다.

각 큐의 할당 시간을 별도로 제어할 수도 있습니다. 이는 -a를 사용하여 모든 큐에 대해 설정하거나 -A를 사용하여 각 큐에 대해 설정할 수 있습니다. 예를 들어, -A 20,40,60은 각 큐의 할당 시간을 각각 20ms, 40ms, 60ms로 설정합니다. 장에서는 할당 시간을 시간 단위로 설명하지만, 여기서는 시간 슬라이스 수로 설명됩니다. 예를 들어, 주어진 큐의 시간 슬라이스 길이가 10ms이고 할당이 2인 경우, 작업은 해당 큐 레벨에서 2개의 시간 슬라이스(20ms) 동안 실행된 후 우선순위가 낮아집니다.

작업을 랜덤으로 생성하는 경우, 작업이 얼마나 오래 실행될지 (-m) 또는 얼마나 자주 I/O를 생성할지 (-M)를 제어할 수 있습니다. 그러나 시스템에서 실행되는 작업의 정확한 특성을 더 많이 제어하려면 -l (소문자 L) 또는 --jlist를 사용하여 시뮬레이션할 정확한 작업 세트를 지정할 수 있습니다. 목록은 x1,y1,z1:x2,y2,z2:... 형식으로 지정되며, 여기서 x는 작업의 시작 시간, y는 실행 시간(즉, 필요한 CPU 시간), z는 I/O 빈도(즉, z ms마다 작업이 I/O를 발행; z가 0이면 I/O가 발행되지 않음)입니다.

예를 들어, 그림 8.3의 예를 재현하려면 다음과 같이 작업 목록을 지정합니다:

```sh
prompt> ./mlfq.py --jlist 0,180,0:100,20,0 -q 10
```

이렇게 시뮬레이터를 실행하면 각 레벨에 10ms의 시간 슬라이스가 있는 세 레벨 MLFQ가 생성됩니다. 두 개의 작업이 생성됩니다: Job 0은 0ms에 시작하여 총 180ms 동안 실행되며, I/O를 발행하지 않습니다; Job 1은 100ms에 시작하여 완료되기 위해 20ms의 CPU 시간이 필요하며, 역시 I/O를 발행하지 않습니다.

마지막으로, 관심 있는 세 가지 매개변수가 더 있습니다. -B 플래그를 설정하면 모든 작업을 N 밀리초마다 가장 높은 우선순위 큐로 부스트합니다. 다음과 같이 사용합니다:

```sh
  prompt> ./mlfq.py -B N
```

스케줄러는 이 기능을 사용하여 장에서 논의된 대로 기아 상태를 방지합니다. 그러나 기본적으로는 꺼져 있습니다.

-S 플래그는 이전 규칙 4a 및 4b를 호출하며, 작업이 시간 슬라이스를 완료하기 전에 I/O를 발행하면, 실행을 재개할 때 동일한 우선순위 큐로 돌아가고 할당이 그대로 유지됩니다. 이는 스케줄러를 속이는 것을 가능하게 합니다.

마지막으로, -i 플래그를 사용하여 I/O가 지속되는 시간을 쉽게 변경할 수 있습니다. 기본적으로 이 단순한 모델에서는 각 I/O가 5ms 또는 이 플래그로 설정한 시간만큼 지속됩니다.

또한, I/O를 완료한 작업이 큐의 맨 앞으로 이동할지 또는 뒤로 이동할지를 -I 플래그로 조정할 수 있습니다. 확인해보세요, 재미있습니다!
