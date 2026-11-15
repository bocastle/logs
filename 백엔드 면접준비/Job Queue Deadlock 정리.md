# Job Queue Deadlock 정리

## 핵심 요약

- 큐 정체의 구조적 원인을 worker 증설 전에 찾을 수 있다.
- 무한 retry가 worker를 점유해 정상 작업까지 막을 수 있다.
- 작업 lock 획득 순서를 고정한다.

## 개념 설명

Job Queue Deadlock은 작업들이 서로의 완료나 lock을 기다리며 큐 진행이 멈추는 상태다.

작업 dependency, lock 순서, worker pool 공유, retry 점유를 확인하고, 대기 그래프나 오래된 RUNNING 작업을 기준으로 끊는다.

## 예시

```text
job A waits file from job B
job B waits lock held by job A
oldest_running_age=45m
resolve: timeout A -> release lock -> retry B
```

큐가 비어 있지 않은데 throughput이 0에 가까우면 단순 worker 부족보다 dependency 교착을 의심해야 한다.

## 면접 답변 예시

> Job queue deadlock은 작업이 남아 있는데 서로의 완료나 lock을 기다려 처리량이 멈춘 상태입니다. 단순 backlog와 구분하려고 queue depth뿐 아니라 completed rate, oldest running age, worker별 wait reason으로 wait-for graph를 만들겠습니다. 같은 worker pool의 child job을 동기적으로 기다리지 않고 여러 lock이 필요하면 획득 순서를 고정합니다. Lease와 max execution time으로 멈춘 작업을 회수하되 재실행 전 중간 상태를 검사하고 멱등 처리나 보상 절차를 적용합니다.

## 장점

- lock 순서와 dependency 규칙을 문서화하게 된다.

## 단점

- 강제 timeout이 중간 상태를 남길 수 있다.

## 주의사항 / 실무 팁

- RUNNING lease와 max execution time을 둔다.
