# Web Storage Quota 정리

## 핵심 요약

- 오프라인 데이터의 용량 한계를 미리 감지할 수 있다.
- quota가 환경마다 달라 정확한 용량 보장이 어렵다.
- 중요 데이터와 재생성 가능한 캐시를 다른 저장소와 키로 나눈다.

## 개념 설명

Web Storage Quota는 origin이 IndexedDB, CacheStorage, OPFS 같은 브라우저 저장소에 사용할 수 있는 용량과 삭제 정책을 다루는 주제다.

`navigator.storage.estimate`로 사용량과 quota를 확인하고, `persist` 요청으로 중요한 데이터를 eviction에서 보호할 수 있는지 확인한다.

## 예시

```ts
const { usage, quota } = await navigator.storage.estimate();
if (usage && quota && usage / quota > 0.8) {
  await pruneOldDrafts();
}
```

저장소가 꽉 차기 전에 오래된 draft를 정리한다. quota는 고정값이 아니라 기기와 브라우저 정책에 따라 달라진다.

## 면접 답변 예시

> Browser storage quota는 origin이 IndexedDB, Cache Storage와 OPFS 등에 저장할 수 있는 용량과 eviction 가능성을 다루는 문제입니다. `navigator.storage.estimate()` 값은 정확한 예약량이 아니라 환경에 따른 추정치이므로 임계값을 참고하되 실제 write 실패도 반드시 처리해야 합니다. 중요한 사용자 draft와 다시 받을 수 있는 cache를 구분해 cache부터 정리하고, `persist()` 요청 결과와 `persisted()` 상태도 확인하겠습니다. 민감정보를 무기한 보관하지 않으며 공간 부족이나 삭제 가능성은 사용자에게 복구 방법과 함께 안내합니다.

## 장점

- 캐시와 사용자 데이터 정리 기준을 나눌 수 있다.

## 단점

- eviction으로 캐시나 임시 데이터가 사라질 수 있다.

## 주의사항 / 실무 팁

- 저장 전 남은 용량과 실패 예외를 처리한다.
