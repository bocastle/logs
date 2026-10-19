# Database Checkpoint Tuning 정리

## 핵심 요약

- dirty page flush를 분산해 write p99를 낮춘다.
- 잦은 checkpoint는 같은 hot page를 반복 기록한다.
- checkpoint write time과 foreground latency를 같은 차트로 본다.

## 개념 설명

database checkpoint는 memory의 dirty page를 storage에 내리고 WAL recovery 시작점을 전진시키는 내구성 작업이다.

checkpoint interval과 completion target을 조정해 page write를 시간에 분산하면 짧은 write spike를 줄이되 crash recovery가 읽을 WAL 양과 균형을 맞춘다.

## 예시

```text
checkpoint interval=10m completion_target=0.9
checkpoint write spread across 9m
metrics: buffers_checkpoint, checkpoint_write_time, WAL bytes
```

checkpoint를 너무 드물게 하면 평소 I/O는 줄어 보여도 recovery 시간과 WAL 보관량이 늘고 마지막 checkpoint에 dirty page가 몰릴 수 있다.

## 면접 답변 예시

> Database checkpoint는 dirty page를 storage에 내리고 crash recovery가 시작할 지점을 전진시키는 작업입니다. 너무 자주 실행하면 hot page를 반복 write하고 너무 드물면 WAL 보관량과 recovery 시간이 늘며 마지막 flush가 몰릴 수 있습니다. Completion target으로 write를 interval에 분산하되 WAL 생성률, checkpoint write time과 foreground p99를 같은 timeline에서 보겠습니다. 설정 변경 후에는 정상 부하뿐 아니라 실제 restart drill로 recovery 시간도 확인합니다.

## 장점

- 복구에 필요한 WAL 범위를 제한한다.

## 단점

- 몰린 flush는 query I/O를 밀어낸다.

## 주의사항 / 실무 팁

- WAL 생성률을 기준으로 interval을 정한다.
