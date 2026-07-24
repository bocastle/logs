# Web Worker 메시지 프로토콜을 설계하는 법 정리

## 핵심 요약

- 동시 요청의 응답을 정확한 호출자에게 연결할 수 있다.
- 필드 검증이 없으면 손상된 payload가 worker 내부 예외로 이어진다.
- 요청과 응답 union을 type 기준으로 검증한다.

## 개념 설명

Web Worker 메시지 프로토콜은 메인 스레드와 worker가 작업 요청, 진행 상태, 결과, 취소를 서로 해석할 수 있도록 정한 버전 있는 데이터 계약이다.

각 `postMessage` payload에 `version`, `type`, `requestId`를 넣고 응답이 같은 requestId를 돌려주게 하면 동시에 진행되는 작업과 늦게 도착한 결과를 구분할 수 있다.

## 예시

```ts
worker.postMessage({ version: 1, type: "HASH", requestId, bytes });
worker.onmessage = ({ data }) => {
  if (data.version !== 1 || data.requestId !== requestId) return;
  if (data.type === "HASH_DONE") resolve(data.digest);
  if (data.type === "HASH_FAILED") reject(new Error(data.message));
};
```

요청과 성공·실패 응답에 같은 requestId와 version을 싣는다. 알 수 없는 type은 무시하지 말고 관측 가능한 프로토콜 오류로 기록해야 한다.

## 면접 답변 예시

> UI와 계산 코드를 직렬화 가능한 계약으로 독립 테스트할 수 있다. 서로 다른 배포 버전의 페이지와 worker가 메시지를 오해할 수 있다. 알 수 없는 version을 받은 양쪽 모두 갱신이나 재시작 경로로 보낸다.

## 장점

- 메시지 버전을 기준으로 점진적인 worker 교체가 가능하다.

## 단점

- 취소된 요청의 늦은 응답이 최신 화면을 덮을 수 있다.

## 주의사항 / 실무 팁

- 취소 메시지와 완료 상태를 requestId별로 기록한다.
