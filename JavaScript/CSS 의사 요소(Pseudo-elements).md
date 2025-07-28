# CSS 의사 요소(Pseudo-elements)

의사 요소는 **CSS 선택자에 추가하는 키워드**로, 특정 요소의 **일부분에만 스타일을 적용**할 때 사용됩니다.  
기본적으로 HTML 구조를 수정하지 않고도 꾸미기 요소를 추가하거나 특정 부분만 스타일링할 수 있게 해줍니다.

---

## 대표적인 의사 요소

- `::before`
- `::after`
- `::first-letter`
- `::first-line`
- `::marker` (리스트 항목 마커에 적용)

---

## `::before` / `::after` 예제

```css
button::before {
  content: "🔥";
  margin-right: 5px;
}
```

```css
h1::after {
  content: "✨";
  margin-left: 5px;
}
```

---

## 장점

- HTML 구조를 변경하지 않아도 됨
- 꾸미기 용도에 적합 (장식 아이콘, 구분선 등)
- 접근성과 구조에 영향을 주지 않음

---

## 의사 요소 vs 의사 클래스

| 구분            | 설명                                                         |
| --------------- | ------------------------------------------------------------ |
| **의사 요소**   | 요소의 **특정 부분**에 스타일 적용 (예: 앞부분, 첫 글자 등)  |
| **의사 클래스** | 요소의 **상태/조건**에 따라 스타일 적용 (예: 마우스 오버 등) |

### 예시 비교

```css
/* 의사 클래스 */
button:hover {
  background-color: lightblue;
}

/* 의사 요소 */
button::before {
  content: "👉";
}
```

---

## 유의사항

- `::before`와 `::after`는 `content` 속성이 반드시 필요합니다.
- 단일 요소에 여러 의사 요소를 동시에 적용할 수 있습니다.
- CSS에서는 `::`가 공식 문법이지만, 일부 구형 브라우저는 `:`를 인식합니다.

---

## 참고

- [MDN Web Docs - Pseudo-elements](https://developer.mozilla.org/en-US/docs/Web/CSS/Pseudo-elements)
