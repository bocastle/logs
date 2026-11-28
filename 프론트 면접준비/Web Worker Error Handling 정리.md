# Web Worker Error Handling 정리

## 핵심 요약

- 작업이 무한 대기 상태로 남는 일을 막을 수 있다.
- error 이벤트만 기록하면 호출자의 Promise가 계속 pending일 수 있다.
- pending 작업 목록에 reject와 timeout 정리 함수를 함께 보관한다.

## 개념 설명

Web Worker 오류 처리는 worker 안의 실행 예외, 역직렬화 실패, 작업별 실패를 서로 다른 채널로 받아 해당 작업과 worker 수명을 정리하는 설계다.

처리하지 않은 실행 예외는 `error`, 메시지 역직렬화 실패는 `messageerror` 이벤트로 전달되고, 복구 불가능한 worker는 pending 작업을 거절한 뒤 `terminate`해야 한다.

## 예시

```ts
worker.addEventListener("error", (event) => {
  rejectPendingJobs(event.message);
  worker.terminate();
});
worker.addEventListener("messageerror", () => {
  rejectPendingJobs("messageerror");
  worker.terminate();
});
```

worker 전역 오류가 발생하면 연결된 pending Promise를 모두 실패시키고 손상 가능성이 있는 인스턴스를 종료한다. 예상 가능한 도메인 오류는 별도 실패 응답으로 처리한다.

## 면접 답변 예시

> 문제가 난 worker만 교체해 나머지 UI를 유지할 수 있다. worker stack과 요청 메타데이터를 버리면 재현 단서가 부족해진다. 오류 로그에는 worker 버전, 작업 type, requestId만 남기고 payload 원문은 피한다.

## 장점

- 실행 오류와 메시지 형식 오류를 다른 지표로 관찰할 수 있다.

## 단점

- 모든 오류마다 즉시 재시작하면 결정적 버그가 재시작 루프를 만든다.

## 주의사항 / 실무 팁

- 같은 입력의 연속 실패에는 재시작 횟수 제한을 둔다.
