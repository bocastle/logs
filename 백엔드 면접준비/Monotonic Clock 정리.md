# Monotonic Clock 정리

## 핵심 요약

- NTP 역행에도 timeout이 음수가 되지 않는다.
- 서로 다른 process의 monotonic 값을 비교할 수 없다.
- duration API가 monotonic source를 쓰는지 확인한다.

## 개념 설명

monotonic clock은 시스템 시작 이후 한 방향으로 증가해 NTP 보정이나 wall clock 변경에 영향받지 않는 elapsed time 측정 기준이다.

timeout과 duration은 monotonic timestamp 차이로 계산하고 사용자 표시와 영구 event time은 UTC wall clock으로 분리한다.

## 예시

```text
start = monotonic_now()
call_remote()
elapsed = monotonic_now() - start
if elapsed > 500ms: timeout
```

monotonic 값은 재부팅과 다른 host 사이에서 비교할 수 없으므로 분산 event 순서나 저장 timestamp로 사용하면 안 된다.

## 면접 답변 예시

> Monotonic clock은 NTP 보정이나 사용자의 시계 변경과 무관하게 한 방향으로 증가하므로 timeout과 elapsed time 측정에 사용합니다. 반면 사용자 표시와 영구 event 시각은 UTC wall clock을 사용합니다. Monotonic 값은 process·host와 reboot 경계를 넘어 비교할 수 없어 분산 event 순서나 저장 timestamp로 쓰면 안 됩니다. Log에는 wall clock과 해당 작업의 elapsed time을 함께 남기겠습니다.

## 장점

- latency 측정을 안정적으로 만든다.

## 단점

- 재부팅 뒤 기준점이 바뀐다.

## 주의사항 / 실무 팁

- 로그에는 wall clock과 elapsed를 함께 남긴다.
