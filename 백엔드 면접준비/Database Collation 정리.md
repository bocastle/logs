# Database Collation 정리

## 핵심 요약

- 언어에 맞는 문자열 정렬을 제공한다.
- 서로 다른 collation 비교는 오류나 추가 sort를 만든다.
- schema에 collation을 명시한다.

## 개념 설명

database collation은 locale 규칙에 따라 문자열의 sort 순서와 대소문자·악센트 비교 의미를 결정하는 설정이다.

column 또는 expression의 collation이 index key ordering과 일치해야 ORDER BY와 equality 비교에서 해당 index를 올바르게 사용할 수 있다.

## 예시

```sql
SELECT name FROM customer
ORDER BY name COLLATE "ko-KR-x-icu";
```

OS나 ICU locale 버전이 바뀌면 정렬 결과와 index ordering이 달라질 수 있어 collation version 확인과 reindex가 필요하다.

## 면접 답변 예시

> Database collation은 문자열의 정렬과 대소문자·accent 비교 의미를 locale 규칙으로 정하는 설정입니다. 업무마다 필요한 언어 규칙이 다르면 column이나 expression 수준에서 명시하되 query의 collation과 index ordering이 맞아야 index를 제대로 활용할 수 있습니다. 자연어 비교는 단순 byte 비교보다 비용이 크고 서로 다른 collation을 섞으면 오류나 추가 sort가 생길 수 있습니다. OS나 ICU version을 올린 뒤에는 영향받는 index와 정렬 결과를 확인하고 필요하면 reindex하겠습니다.

## 장점

- 대소문자와 악센트 비교 규칙을 일관되게 둔다.

## 단점

- locale upgrade 뒤 index 순서가 stale할 수 있다.

## 주의사항 / 실무 팁

- 운영·개발의 ICU와 locale version을 맞춘다.
