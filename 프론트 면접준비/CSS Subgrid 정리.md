# CSS Subgrid 정리

## 핵심 요약
- `subgrid`는 CSS Grid의 하위 그리드가 부모의 `grid-template-rows`/`grid-template-columns`를 그대로 상속받아 정렬되는 기능이다.
- 카드 목록, 폼 필드 정렬, 복잡한 카드형 레이아웃에서 **열/행 정렬 일관성**을 크게 줄인다.
- 기존에는 자식마다 수동으로 동일한 track 크기·간격을 맞췄지만, subgrid는 선언 위치만 맞추면 자동으로 맞물린다.
- 브라우저 지원을 고려해 progressive enhancement(점진적 적용)가 중요하다.

## 개념 설명
CSS Grid는 부모 컨테이너가 정의한 열/행 라인을 기반으로 자식 요소를 배치한다.  
`subgrid`는 이 `track` 정보를 하위 그리드가 상속해 쓰게 해주는 값이다.

주요 포인트는 다음과 같다.
- 하위 그리드의 `grid-template-columns: subgrid` 또는 `grid-template-rows: subgrid`를 사용한다.
- 하위 요소는 부모의 라인 번호, 간격(`gap`), track 크기 규칙을 그대로 따른다.
- 기본 `grid`보다 선언량이 줄고, 구조 변경 시 유지보수성이 좋아진다.
- 단, Safari 계열은 오래전 버전 지원이 제한적이므로 실서비스에서 전역 기본 적용 전에 브라우저 정책 점검이 필요하다.

## 예시
```css
.card-list {
  display: grid;
  grid-template-columns: repeat(3, minmax(220px, 1fr));
  gap: 16px;
}

.product-card {
  display: grid;
  grid-template-rows: subgrid;
  grid-row: span 3;
  gap: inherit;
}
```

`product-card`의 하위 행(예: 이미지 영역, 제목, 가격, 버튼)은 부모가 정한 행 라인을 기준으로 정렬되어
카드들 간의 라인 정합성이 맞는다.

## 면접 답변 예시
- 질문: "`subgrid`를 쓰는 장점이 뭔가요?"  
  답변: "기존처럼 각 카드마다 `grid-template-columns`를 개별 계산하지 않아도 돼요. 부모의 track 규칙을 재사용해 라인 정렬을 일관되게 유지할 수 있고, 구조 변경 시 레이아웃 수정 범위도 줄어듭니다."
- 질문: "`grid`만으로도 충분한데 왜 `subgrid`가 필요한가요?"  
답변: "형제/자식 컴포넌트가 반복되는 패턴에서 공통 정렬 규칙을 강제할 때 효율적입니다. 특히 카드 리스트에서 제목 길이/가격 위치가 들쑥날쑥한 UI 품질 이슈를 줄일 수 있습니다."

## 장점
- 동일 패턴 UI에서 레이아웃 일관성 확보
- 반복 컴포넌트 수정 비용 감소
- 부모-자식 레이아웃 책임이 분리되어 읽기 쉬운 CSS

## 단점
- 브라우저 호환성 이슈(특히 Safari 일부 버전)로 폴백 필요
- 팀 내 공통 그리드 규칙이 약하면 오히려 의존성이 커질 수 있음

## 주의사항 / 실무 팁
- `subgrid`를 지원하지 않는 브라우저에서는 `@supports (grid-template-columns: subgrid)`로 fallback 스타일을 둔다.
- `gap`은 부모/자식 모두에서 중복 정의하지 않도록 한 곳에서 통일한다.
- 디버깅 시 DevTools에서 그리드 라인 번호와 실제 콘텐츠 흐름을 같이 확인해야 의도한 정렬을 빠르게 잡을 수 있다.
