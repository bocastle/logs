# CSS print media 정리

## 핵심 요약

- 계약서, 영수증, 리포트 출력 품질을 별도로 관리할 수 있다.
- 화면 CSS만 보고 배포하면 인쇄에서 표나 카드가 잘릴 수 있다.
- 대표 문서를 print preview와 PDF로 확인한다.

## 개념 설명

CSS print media는 인쇄와 PDF 출력에서 화면용 레이아웃과 다른 스타일을 적용하는 `@media print` 규칙이다.

인쇄에서는 배경, 내비게이션, sticky UI를 줄이고 page break 규칙으로 표나 카드가 어색하게 쪼개지지 않게 조절한다.

## 예시

```css
@media print {
  nav, .toolbar { display: none; }
  article {
    break-inside: avoid;
    page-break-inside: avoid;
  }
}
```

`@media print`에서 화면 조작 UI를 숨기고 `page-break` 관련 규칙으로 출력 흐름을 다듬는 예다.

## 면접 답변 예시

> Print CSS는 화면 layout을 그대로 축소하는 것이 아니라 계약서, 영수증과 report가 page 단위로 읽히게 만드는 별도 출력 설계입니다. Navigation과 sticky control을 숨기고 `break-inside`, widows와 orphans를 조정하되 긴 block에 avoid를 남발해 큰 빈 공간을 만들지 않겠습니다. Background와 color 출력은 사용자 설정에 따라 달라질 수 있어 상태를 색에만 의존하면 안 됩니다. 대표 data로 print preview와 PDF를 비교하고 날짜, URL과 서명처럼 출력에 필요한 정보가 남는지 확인합니다.

## 장점

- 화면 전용 컨트롤이 인쇄물에 섞이는 일을 줄인다.

## 단점

- 배경색과 그림은 브라우저 설정에 따라 빠질 수 있다.

## 주의사항 / 실무 팁

- page-break, widows, orphans 규칙을 긴 콘텐츠에서 본다.
