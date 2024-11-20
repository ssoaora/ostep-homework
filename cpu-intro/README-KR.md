
# 개요

이 프로그램, process-run.py는 프로세스 상태가 CPU에서 실행됨에 따라 어떻게 변하는지 보여줍니다. 챕터에서 설명한 바와 같이, 프로세스는 몇 가지 다른 상태에 있을 수 있습니다:

```sh
RUNNING - 프로세스가 현재 CPU를 사용 중입니다
READY   - 프로세스가 지금 당장 CPU를 사용할 수 있지만
      (아쉽게도) 다른 프로세스가 사용 중입니다
BLOCKED - 프로세스가 I/O를 기다리고 있습니다
      (예: 디스크에 요청을 보냈습니다)
DONE    - 프로세스가 실행을 완료했습니다
```

이 과제에서는 프로그램이 실행됨에 따라 이러한 프로세스 상태가 어떻게 변하는지 살펴보고, 이러한 것들이 어떻게 작동하는지 조금 더 잘 이해하게 될 것입니다.

프로그램을 실행하고 옵션을 확인하려면 다음을 수행하십시오:

```sh
prompt> ./process-run.py -h
```

이 명령이 작동하지 않으면, 명령어 앞에 `python`을 입력하십시오:

```sh
prompt> python process-run.py -h
```

다음과 같은 화면이 나타날 것입니다:

```sh
Usage: process-run.py [options]

Options:
  -h, --help            도움말 메시지를 표시하고 종료합니다
  -s SEED, --seed=SEED  랜덤 시드
  -l PROCESS_LIST, --processlist=PROCESS_LIST
            실행할 프로세스 목록을 쉼표로 구분하여 지정합니다.
            형식은 X1:Y1,X2:Y2,... 여기서 X는 프로세스가 실행할
            명령어 수이고, Y는 명령어가 CPU를 사용할 확률(0에서 100 사이)입니다.
  -L IO_LENGTH, --iolength=IO_LENGTH
            I/O가 완료되는 데 걸리는 시간
  -S PROCESS_SWITCH_BEHAVIOR, --switch=PROCESS_SWITCH_BEHAVIOR
            프로세스 간 전환 시점: SWITCH_ON_IO,
            SWITCH_ON_END
  -I IO_DONE_BEHAVIOR, --iodone=IO_DONE_BEHAVIOR
            I/O가 완료될 때의 동작 유형: IO_RUN_LATER,
            IO_RUN_IMMEDIATE
  -c                    답을 계산합니다
  -p, --printstats      통계를 출력합니다; -c 플래그와 함께 사용할 때만 유용합니다
            (그렇지 않으면 통계가 출력되지 않습니다)
```

가장 중요한 옵션은 PROCESS_LIST (-l 또는 --processlist 플래그로 지정)로, 각 실행 프로그램(또는 '프로세스')이 정확히 무엇을 할 것인지 지정합니다. 프로세스는 명령어로 구성되며, 각 명령어는 다음 두 가지 중 하나를 수행할 수 있습니다:
- CPU 사용
- I/O 요청 (그리고 완료될 때까지 대기)

프로세스가 CPU를 사용할 때 (그리고 전혀 I/O를 하지 않을 때), 단순히 CPU에서 RUNNING 상태와 READY 상태를 번갈아 가며 실행됩니다. 예를 들어, 여기에는 단순히 CPU만 사용하는 하나의 프로그램이 실행되는 간단한 실행이 있습니다.

```sh
prompt> ./process-run.py -l 5:100 
이 프로세스를 실행할 때 발생할 수 있는 추적을 생성합니다:
프로세스 0
  cpu
  cpu
  cpu
  cpu
  cpu

중요한 동작:
  현재 프로세스가 완료되거나 I/O를 요청할 때 시스템이 전환됩니다
  I/O 후, I/O를 요청한 프로세스는 나중에 실행됩니다 (자신의 차례가 되면)
```

여기서 지정한 프로세스는 "5:100"으로, 5개의 명령어로 구성되며 각 명령어가 CPU 명령어일 확률이 100%입니다.

-c 플래그를 사용하여 프로세스가 어떻게 되는지 확인할 수 있습니다. 이 플래그는 답을 계산해줍니다:

```sh
prompt> ./process-run.py -l 5:100 -c
시간     PID: 0        CPU        IOs
  1     RUN:cpu          1
  2     RUN:cpu          1
  3     RUN:cpu          1
  4     RUN:cpu          1
  5     RUN:cpu          1
```

이 결과는 그다지 흥미롭지 않습니다: 프로세스는 단순히 RUN 상태에 있다가 완료되며, 실행 내내 CPU를 사용하여 CPU를 계속 바쁘게 유지하고, I/O는 전혀 하지 않습니다.

두 개의 프로세스를 실행하여 약간 더 복잡하게 만들어 봅시다:

```sh
prompt> ./process-run.py -l 5:100,5:100
이 프로세스를 실행할 때 발생할 수 있는 추적을 생성합니다:
프로세스 0
  cpu
  cpu
  cpu
  cpu
  cpu

프로세스 1
  cpu
  cpu
  cpu
  cpu
  cpu

중요한 동작:
  현재 프로세스가 완료되거나 I/O를 요청할 때 시스템이 전환됩니다
  I/O 후, I/O를 요청한 프로세스는 나중에 실행됩니다 (자신의 차례가 되면)
```

이 경우, 두 개의 다른 프로세스가 실행되며, 각각 다시 CPU만 사용합니다. 운영 체제가 이들을 실행할 때 무슨 일이 일어날까요? 알아봅시다:

```sh
prompt> ./process-run.py -l 5:100,5:100 -c
시간     PID: 0     PID: 1        CPU        IOs
  1     RUN:cpu      READY          1
  2     RUN:cpu      READY          1
  3     RUN:cpu      READY          1
  4     RUN:cpu      READY          1
  5     RUN:cpu      READY          1
  6        DONE    RUN:cpu          1
  7        DONE    RUN:cpu          1
  8        DONE    RUN:cpu          1
  9        DONE    RUN:cpu          1
 10        DONE    RUN:cpu          1
```

위에서 볼 수 있듯이, 먼저 "프로세스 ID" (또는 "PID") 0이 실행되고, 프로세스 1은 READY 상태로 대기합니다. 0이 완료되면 DONE 상태로 이동하고, 1이 실행됩니다. 1이 완료되면 추적이 완료됩니다.

질문으로 넘어가기 전에 한 가지 예를 더 살펴보겠습니다. 이 예에서는 프로세스가 단순히 I/O 요청만 합니다. 여기서는 -L 플래그를 사용하여 I/O가 완료되는 데 5시간 단위가 걸린다고 지정합니다.

```sh
prompt> ./process-run.py -l 3:0 -L 5
이 프로세스를 실행할 때 발생할 수 있는 추적을 생성합니다:
프로세스 0
  io
  io_done
  io
  io_done
  io
  io_done

중요한 동작:
  현재 프로세스가 완료되거나 I/O를 요청할 때 시스템이 전환됩니다
  I/O 후, I/O를 요청한 프로세스는 나중에 실행됩니다 (자신의 차례가 되면)
```

실행 추적이 어떻게 될 것 같습니까? 알아봅시다:

```sh
prompt> ./process-run.py -l 3:0 -L 5 -c
시간    PID: 0       CPU       IOs
  1         RUN:io             1
  2        BLOCKED                           1
  3        BLOCKED                           1
  4        BLOCKED                           1
  5        BLOCKED                           1
  6        BLOCKED                           1
  7*   RUN:io_done             1
  8         RUN:io             1
  9        BLOCKED                           1
 10        BLOCKED                           1
 11        BLOCKED                           1
 12        BLOCKED                           1
 13        BLOCKED                           1
 14*   RUN:io_done             1
 15         RUN:io             1
 16        BLOCKED                           1
 17        BLOCKED                           1
 18        BLOCKED                           1
 19        BLOCKED                           1
 20        BLOCKED                           1
 21*   RUN:io_done             1
```

보시다시피, 프로그램은 단순히 세 번의 I/O를 요청합니다. 각 I/O가 요청되면, 프로세스는 BLOCKED 상태로 이동하고, 장치가 I/O를 처리하는 동안 CPU는 유휴 상태가 됩니다.

I/O 완료를 처리하기 위해 한 번 더 CPU 동작이 발생합니다. I/O 시작 및 완료를 처리하는 단일 명령어는 특히 현실적이지 않지만, 여기서는 단순화를 위해 사용되었습니다.

통계를 출력해 봅시다 (위와 동일한 명령을 실행하되 -p 플래그를 추가하여) 전체 동작을 확인해 봅시다:

```sh
통계: 총 시간 21
통계: CPU 사용 6 (28.57%)
통계: IO 사용  15 (71.43%)
```

보시다시피, 추적은 21 클럭 틱이 걸렸지만, CPU는 30% 미만의 시간 동안 바빴습니다. 반면에 I/O 장치는 매우 바빴습니다. 일반적으로 모든 장치를 바쁘게 유지하는 것이 자원을 더 잘 활용하는 것입니다.

다음은 몇 가지 중요한 플래그입니다:
```sh
  -s SEED, --seed=SEED  랜덤 시드  
  이는 무작위로 다양한 작업을 생성할 수 있는 방법을 제공합니다

  -L IO_LENGTH, --iolength=IO_LENGTH
  이는 I/O가 완료되는 데 걸리는 시간을 결정합니다 (기본값은 5 틱)

  -S PROCESS_SWITCH_BEHAVIOR, --switch=PROCESS_SWITCH_BEHAVIOR
            프로세스 간 전환 시점: SWITCH_ON_IO, SWITCH_ON_END
  이는 다른 프로세스로 전환할 시점을 결정합니다:
  - SWITCH_ON_IO, 프로세스가 I/O를 요청할 때 시스템이 전환됩니다
  - SWITCH_ON_END, 현재 프로세스가 완료될 때만 시스템이 전환됩니다 

  -I IO_DONE_BEHAVIOR, --iodone=IO_DONE_BEHAVIOR
            I/O가 완료될 때의 동작 유형: IO_RUN_LATER, IO_RUN_IMMEDIATE
  이는 프로세스가 I/O를 요청한 후 실행되는 시점을 결정합니다:
  - IO_RUN_IMMEDIATE: 지금 바로 이 프로세스로 전환합니다
  - IO_RUN_LATER: 프로세스 전환 동작에 따라 자연스럽게 이 프로세스로 전환합니다
```

이제 챕터 뒷부분의 질문에 답하여 더 많은 것을 배우십시오.

