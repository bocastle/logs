# API Idempotency 저장소 설계 정리

## 핵심 요약

- timeout 후 재시도해도 같은 결과를 돌려줄 수 있다.
- 저장소 쓰기와 실제 부작용 순서가 어긋나면 중복이나 누락이 생긴다.
- idempotency key는 client와 route 범위를 함께 묶는다.

## 개념 설명

API Idempotency 저장소는 같은 idempotency key로 들어온 쓰기 요청을 한 번의 결과로 고정해 중복 생성을 막는 저장소다.

key, request hash, status, response snapshot, expires_at을 원자적으로 저장하고, 같은 key에 다른 request hash가 오면 충돌로 거절한다.

## 예시

```sql
INSERT INTO idempotency_keys(client_id, route, key, request_hash, status, expires_at)
VALUES (:client_id, :route, :key, :hash, 'processing', now() + interval '24 hours')
ON CONFLICT (client_id, route, key) DO NOTHING
RETURNING key;
```

row를 반환받은 최초 요청만 처리를 시작한다. 기존 row가 있으면 request hash와 상태를 읽어 같은 요청의 완료 결과인지, 처리 중인지, key 재사용 충돌인지 구분한다.

## 면접 답변 예시

> Idempotency 저장소는 같은 client와 route의 key로 들어온 쓰기 요청을 한 결과에 고정해 timeout 재시도의 중복 부작용을 막습니다. Unique constraint로 최초 요청만 processing claim을 얻고 나머지는 저장된 request hash와 상태를 확인하게 하겠습니다. 같은 key에 다른 payload가 오면 409로 거절하고 완료된 요청에는 같은 response를 돌려줍니다. 실제 부작용과 상태 저장의 원자성, 오래 남은 processing 복구와 response 개인정보 retention까지 함께 설계해야 합니다.

## 장점

- 결제나 주문 생성의 중복 부작용을 줄인다.

## 단점

- response snapshot을 너무 오래 보관하면 개인정보 보존 위험이 커진다.

## 주의사항 / 실무 팁

- request hash 불일치는 409 같은 안정적인 오류로 응답한다.
