# Database Page Split 정리

## 핵심 요약

- B-tree가 key 증가에도 균형을 유지한다.
- random UUID는 여러 page split을 유발한다.
- key 분포와 split count를 함께 측정한다.

## 개념 설명

page split은 B-tree leaf page에 새 key를 넣을 공간이 없어 page를 둘로 나누고 parent pointer를 갱신하는 구조 변경이다.

random key insert와 가득 찬 page는 split 빈도를 높이며 fillfactor로 빈 공간을 남기면 update와 중간 insert 여유를 확보할 수 있다.

## 예시

```text
leaf page [10,20,30,40] full
insert 25 -> split [10,20,25] + [30,40]
fillfactor=80 -> future insert 여유 확보
```

split은 정상 B-tree 동작이지만 빈번하면 write I/O, WAL, fragmentation이 늘어나므로 key locality와 page density를 같이 본다.

## 면접 답변 예시

> B-tree page split은 leaf page에 새 key를 넣을 공간이 없을 때 page를 나누고 parent pointer를 갱신하는 정상 동작입니다. Random UUID처럼 중간 위치 insert가 넓게 퍼지면 split과 cache miss가 늘 수 있고 순차 key는 locality가 좋지만 오른쪽 끝 page 경합을 만들 수 있습니다. Index fillfactor로 중간 insert 여유를 남기면 split을 줄일 수 있지만 index 크기는 커집니다. Key 분포, split과 WAL 양, write p99와 replica lag를 함께 측정해 UUIDv7 같은 대안을 판단하겠습니다.

## 장점

- index fillfactor로 향후 중간 key insert 공간을 예약한다.

## 단점

- 낮은 fillfactor는 index 크기를 늘린다.

## 주의사항 / 실무 팁

- UUIDv7 같은 시간 정렬 key를 검토한다.
