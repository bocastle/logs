# Redis negative caching 적용 기준 정리

## 핵심 요약

- Negative caching은 실제로 존재하지 않는 결과를 짧게 저장해 반복 miss가 원본 저장소로 몰리는 일을 줄인다.
- Timeout과 권한 거절을 `NOT_FOUND`로 캐시하지 말고, 확정적인 부재 결과만 대상으로 삼는다.
- 새 데이터 생성 시 무효화하고 일반 데이터 캐시보다 짧은 TTL과 key 수 제한을 둔다.

## 개념 설명

Redis negative caching은 데이터가 존재하지 않는다는 결과를 짧게 캐시하는 기법이다. 잘못된 ID나 크롤러가 같은 대상을 반복 조회할 때 매번 데이터베이스까지 가지 않게 한다.

여기서 일시적인 DB timeout과 권한 때문에 숨긴 결과는 실제 부재와 구분해야 한다. 확정적인 not found만 저장하고, 데이터가 나중에 생성될 수 있다면 생성 트랜잭션 이후 negative key를 지운다. 삭제가 실패할 수 있으므로 TTL도 짧게 유지한다.

## 예시

```text
GET user:42 -> miss
DB: not found
SETEX neg:user:42 30s "NOT_FOUND"
create user:42 -> DEL neg:user:42
```

없는 값을 캐시한 뒤 `user:42`가 생성되면 TTL 동안 계속 없다고 응답할 수 있다. 생성 경로의 무효화와 짧은 TTL을 함께 사용한다. 임의 ID 요청으로 key가 폭증하지 않도록 입력 범위와 tenant별 한도도 고려한다.

## 면접 답변 예시

> Negative caching은 존재하지 않는 데이터에 대한 반복 조회를 짧게 캐시해 원본 저장소 부하를 줄이는 방법입니다. 저는 확정적인 not found만 저장하고 DB timeout이나 권한 거절은 같은 값으로 취급하지 않겠습니다. 일반 캐시보다 짧은 TTL을 두고 데이터 생성이 성공하면 해당 negative key를 삭제합니다. 운영에서는 hit ratio뿐 아니라 key 수, 생성 직후 stale not found와 원본 조회 감소량을 함께 보겠습니다.

## 장점

- 크롤러나 잘못된 ID의 반복 요청이 데이터베이스까지 도달하는 비용을 줄일 수 있다.
- 존재하지 않는 대상의 응답 지연을 짧고 일정하게 만들 수 있다.

## 단점

- 새 데이터가 생성돼도 negative key가 남아 있으면 TTL 동안 존재하지 않는 것처럼 보인다.
- 공격자가 무작위 key를 대량으로 만들면 Redis 메모리가 negative entry로 채워질 수 있다.

## 주의사항 / 실무 팁

- 생성 트랜잭션이 성공한 뒤 negative key를 삭제하고 실패 시 짧은 TTL로 회복되게 한다.
- tenant별 negative key 수, not-found 원본 조회량과 stale 응답 사례를 관찰한다.
