# Password Hashing 전략 정리

## 핵심 요약

- DB 유출 시 원문 비밀번호를 바로 알기 어렵다.
- 빠른 hash 함수인 SHA-256만 쓰면 brute force에 약하다.
- 현재 권장 알고리즘과 비용을 주기적으로 재평가한다.

## 개념 설명

Password Hashing은 비밀번호 원문 대신 느린 단방향 함수 결과와 salt를 저장해 유출 피해를 줄이는 전략이다.

Argon2id, bcrypt, scrypt 같은 password hashing 함수를 쓰고, 비용 파라미터는 로그인 지연과 공격 비용을 함께 측정해 정한다.

## 예시

```text
hash = Argon2id(password, salt, memory=64MiB, iterations=3)
stored = algorithm + params + salt + hash
```

알고리즘과 파라미터를 같이 저장해야 나중에 로그인 시점에 더 강한 hash로 재해싱할 수 있다.

## 면접 답변 예시

> Password는 SHA-256 같은 빠른 일반 hash가 아니라 Argon2id, scrypt나 bcrypt처럼 공격 비용을 높이는 전용 password hashing 함수로 저장해야 합니다. 사용자별 random salt와 algorithm·parameter를 hash 옆에 저장하고 production hardware에서 login latency와 memory를 측정해 비용을 정하겠습니다. 로그인 성공 시 오래된 parameter를 새 값으로 재hash하면 점진적으로 강도를 올릴 수 있습니다. Pepper를 쓴다면 DB와 분리된 secret manager에서 관리하고 원문 password는 log, metric과 error에 남기지 않습니다.

## 장점

- 사용자별 salt로 rainbow table 공격을 줄인다.

## 단점

- 비용을 과하게 높이면 로그인 장애와 DoS 위험이 생긴다.

## 주의사항 / 실무 팁

- 로그인 성공 시 오래된 hash를 새 파라미터로 갱신한다.
