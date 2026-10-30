# CSS overflow-anchor 정리

## 핵심 요약

- 늦게 로드된 이미지·배너 때문에 읽던 콘텐츠가 튀는 일을 줄인다.
- 전체 page에 `none`을 쓰면 기본 scroll anchoring 보호를 잃는다.
- 기본 `auto`를 유지하고 잘못 anchor로 선택되는 작은 UI만 제외한다.

## 개념 설명

CSS `overflow-anchor`는 scroller 안에서 viewport 위쪽 콘텐츠의 크기가 바뀐 때 브라우저 scroll anchoring 조정에 해당 요소가 참여할지 정하는 속성이다.

브라우저는 scroller 안의 anchor node를 골라 레이아웃 변화 후에도 그 node의 화면 위치를 유지하려 한다. `overflow-anchor: none`은 해당 subtree를 anchor 후보에서 제외한다.

## 예시

```css
.chat__loading-indicator {
  overflow-anchor: none;
}
```

chat 상단의 잠깐 보이는 loading indicator가 scroll anchor로 선택되지 않게 해 메시지 읽던 위치를 유지한다.

## 면접 답변 예시

> Scroll anchoring은 viewport 위쪽 content 크기가 바뀔 때 browser가 anchor node의 화면 위치를 유지해 읽던 곳이 튀지 않게 합니다. `overflow-anchor: none`은 특정 subtree를 후보에서 제외하는 수단이라 page 전체에 적용하면 기본 보호를 잃습니다. Loading indicator처럼 잘못 anchor로 선택되는 작은 UI에만 제한하고 image size 예약과 font metric 개선을 먼저 하겠습니다. Virtualization, scroll snap과 수동 restoration이 동시에 위치를 보정하지 않는지 실시간 삽입과 느린 resource 조건에서 확인합니다.

## 장점

- JS로 scrollTop을 보정하는 코드를 줄인다.

## 단점

- 잘못된 subtree를 제외하면 새 컨텐츠 삽입 시 화면이 튀는다.

## 주의사항 / 실무 팁

- Layout Shift source와 anchor node를 함께 관찰한다.
