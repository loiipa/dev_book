---
title: CPU Scheduling algorithm
type: docs
---

## CPU Scheduling algorithm 비교하기 위한 기준
### CPU utilization (CPU 사용률)
- 모든 CPU algorthm의 주요 목적은 CPU를 최대한 바쁘게 움직이는 것임.
- 이론적으로 cpu 사용률은 0~100%까지
- 실시간 시스템에서 시스템 부하에 따라 40~90%까지 달라짐.

### Throughput (처리량)
- 평균 CPU 성능 → 각 unit(단위)동안 수행 및 완료한 process의 수
- output은 프로세스의 길이나 기간에 따라 달라질 수 있음.

### Turn round Time (턴 라운드 시간)
- 특정 프로세스의 경우 중요한 조건
- 해당 프로세스를 수행하는 데 걸리는 시간.
- 프로세스 전달 시점부터 완료 시점까지 경과된 시간 → conversion time(변환 시간)

#### conversion time
- 메모리 엑세스 대기, 대기열 대기, CPU사용, I/O 대기 등에 소요되는 시간.

### Waiting Time (대기 시간)
- scheduling algorithm은 프로세스가 실행을 시작한 이후, 완료하는 데 필요한 시간에는 영향을 주지 않음.
- ready queue에서 대기중인 프로세스에 소요되는 시간에만 영향을 미침.

### Response Time (응답 시간)
- collaborative system에서 turn around time은 최선의 선택이 아님.
- 프로세스가 조기에 무언가를 생성하고 이전 결과가 사용자에게 공개되는 동안, 새로운 결과를 계속 계산할 수 있음.
- 신청 프로세스를 제출하는 데 걸리는 시간 측정하는 것 → 응답시간

### CPU Scheduling Algorithm Type
![선점형 / 비선점형으로 나누어져 있는 CPU Scheduling](/dev_book/cpu_scheduling.png) 

---

## Preemptive Scheduling (선점 스케줄링)
- process가 running → ready 상태로 전환되거나 waiting → ready로 전환될 때 사용됨.

### RR (Round Robin)
- 각 프로세스에게 동일한 할당 시간을 주고, 그 시간 안에 끝나지 않으면 다시 ready queue의 뒤로 가는 알고리즘
- 할당 시간이 너무 크면 FCFS가 됨
- 할당 시간이 너무 짧으면 context switching이 잦아져 overhead가 커짐
- 로드 밸런서에서 트래픽 분산 알고리즘으로도 쓰임.

### SRTF (Shortest Remaining Time First)
- 다른 프로세스가 실행되는 동안 더 짧은 프로세스가 도착하면, 현재 실행 중인 프로세스가 중단되고 더 짧은 프로세스 진행
- 추가 오버헤드 발생
- 작은 프로세스가 많이 실행되는 시스템에서는 기아 발생할 가능성 높음

### LRTF (Longest Remaining Time First)
- 다른 프로세스가 실행되는 동안 더 긴 프로세스가 도착 → 더 긴 프로세스 진행
- 높은 대기시간, 평균 처리시간 → convoy effect 발생

### Priority Scheduling (EDF, Earliest deadline first)
- 스케줄링 이벤트가 일어날 때마다 queue에서 마감시간이 가장 가까운 프로세스를 탐색하여 다음에 수행되도록 하는 것.
- 프로세스의 마감 시간은 프로세스가 생성되거나 예약될 때 명시적으로 지정됨.
- 현실적으로 프로세스의 마감시간을 예측하는 것은 어려움

---

## Non-Preemptive(비선점 스케줄링)
- process가 종료되거나 prcess가 running → waiting 상태로 전환될 때 사용됨.

### FCFS (First Come, First Served)
- FIFO
- 단순히 준비된 Queue에 도착하는 순서대로 프로세스를 queue에 대기시킴

### SJF (Shortest Job First)
- 실행시간이 가장 짧은 프로세스를 가장 먼저 실행시킴
- 평균 대기 시간이 가장 짧음
- 긴 시간을 가진 프로세스가 실행되지 않는 현상 발생 (starvation, 기아)
- 실제 실행 시간을 알 수 없음 → 과거 실행 시간을 토대로 추측함.

### LJF (Longest Job First)
- 실행시간이 가장 긴 프로세스를 가장 먼저 실행시킴
- 평균 대기 시간이 길다. → convoy effect 발생(장비의 사용률이 낮아짐)

### HRRN (Highest Response Ration Next)
- 프로세스 처리의 우선 순위를 CPU 처리 기간과 해당 프로세스의 대기 시간을 동시에 고려해 선정
- SJF 스케줄링의 문제점 보완
![Alt text](/dev_book/response_ratio.png)
- 즉, 대기 시간이 분자에 있기 때문에, 대기시간이 긴 프로세스가 추후에 우선순위를 가질 수 있음.

---

{{< hint info >}}
**참고 자료**  
[CPU Scheduling in Operating Systems - GeeksforGeeks](https://www.geeksforgeeks.org/cpu-scheduling-in-operating-systems)  
[Scheduling (computing)](https://en.wikipedia.org/wiki/Scheduling_(computing))  
[Highest response ratio next](https://en.wikipedia.org/wiki/Highest_response_ratio_next)  
[HRRN 스케줄링](https://ko.wikipedia.org/wiki/HRRN_%EC%8A%A4%EC%BC%80%EC%A4%84%EB%A7%81)  
[최단 마감 우선 스케줄링](https://ko.wikipedia.org/wiki/%EC%B5%9C%EB%8B%A8_%EB%A7%88%EA%B0%90_%EC%9A%B0%EC%84%A0_%EC%8A%A4%EC%BC%80%EC%A4%84%EB%A7%81)  
[로드 밸런싱이란 무엇인가요? - 로드 밸런싱 알고리즘 설명 - AWS (amazon.com)](https://aws.amazon.com/ko/what-is/load-balancing/)  
{{< /hint >}}