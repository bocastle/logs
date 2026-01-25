# Node와 Element의 차이 정리

## 1) 핵심 개념

- **Node**: DOM을 구성하는 가장 기본 단위의 _상위 개념_. 문서(document), 요소(element), 텍스트(text), 주석(comment) 등 **모든** 노드 타입을 포함합니다.
- **Element**: Node 타입 중 하나로 **HTML/XML 태그 자체**를 의미합니다. 모든 Element는 Node이지만, 모든 Node가 Element는 아닙니다.

## 2) DOM 트리에서의 관계

```
Document (Node)
└─ Element (e.g., <div>)
   ├─ Text (e.g., "Hello")
   └─ Comment (e.g., <!-- 주석 -->)
```

- `Document`, `Element`, `Text`, `Comment`는 모두 Node입니다.
- `Element`만이 **속성(attributes)**(id, class, style 등)을 가질 수 있습니다.

## 3) 주요 API/속성 차이

| 구분                                              | 대상    | 설명                                                              | 예시                    |
| ------------------------------------------------- | ------- | ----------------------------------------------------------------- | ----------------------- |
| `textContent`                                     | Node    | 노드 및 자식 노드의 텍스트를 가져오거나 설정                      | 모든 노드에서 사용 가능 |
| `innerHTML`                                       | Element | HTML 문자열로 자식 요소를 읽고/치환                               | Element에서만 사용 가능 |
| `childNodes`                                      | Node    | **모든 자식 노드**(Element, Text, Comment 포함)를 반환 (NodeList) | 공백 텍스트/주석 포함   |
| `children`                                        | Element | **자식 요소 노드만** 반환 (HTMLCollection)                        | 텍스트/주석 제외        |
| `attributes`                                      | Element | 요소의 속성 컬렉션                                                | `el.attributes.id` 등   |
| 탐색자(`firstChild`, `nextSibling`)               | Node    | 모든 타입 대상으로 형제/자식 접근                                 | 텍스트/주석 포함 가능   |
| 탐색자(`firstElementChild`, `nextElementSibling`) | Element | 요소 노드만 대상으로 탐색                                         | 텍스트/주석 제외        |

## 4) 선택/탐색 메서드 차이

- **Element 전용**: `querySelector`, `querySelectorAll`, `getElementsByClassName`, `getElementsByTagName`
- **Node 공통**: `parentNode`, `childNodes`, `contains`, `cloneNode`, `appendChild`, `removeChild` 등
  - 단, 일부는 Document/Element에서 주로 의미 있게 사용됩니다.

## 5) 예시로 보는 차이

```html
<div id="wrap">Hello<!-- 주석 -->World<span>!</span></div>
<script>
  const el = document.getElementById("wrap"); // Element
  console.log(el instanceof Node); // true
  console.log(el instanceof Element); // true

  // 1) children vs childNodes
  console.log(el.children.length); // 1  -> <span>만 포함
  console.log(el.childNodes.length); // 3  -> Text("Hello"), Comment, Text("World"), 그리고 <span>까지면 4가 될 수 있음
  // 공백/줄바꿈 여부에 따라 달라질 수 있습니다.

  // 2) innerHTML vs textContent
  console.log(el.innerHTML); // 'Hello<!-- 주석 -->World<span>!</span>'
  console.log(el.textContent); // 'HelloWorld!'  (주석/태그 제외 순수 텍스트)

  // 3) Element 전용 속성/메서드
  el.setAttribute("data-x", "1"); // OK (Element)
  // document.setAttribute('data-x','1'); -> TypeError (Document는 Element가 아님)

  // 4) 노드 탐색
  console.log(el.firstChild.nodeType); // 3(Text) 또는 공백 여부에 따라 달라짐
  console.log(el.firstElementChild.tagName); // 'SPAN'
</script>
```

### 참고: `nodeType` 상수

- `1`: Element
- `3`: Text
- `8`: Comment
- `9`: Document

## 6) 한눈에 비교

| 항목        | Node                                         | Element                                                   |
| ----------- | -------------------------------------------- | --------------------------------------------------------- |
| 정의        | DOM의 모든 단위를 아우르는 상위 개념         | 태그(요소) 자체를 나타내는 노드 타입                      |
| 예          | Document, Element, Text, Comment             | div, span, a, img 등 HTML 요소                            |
| 속성/메서드 | `textContent`, `childNodes`, `firstChild` 등 | `innerHTML`, `attributes`, `children`, `querySelector` 등 |
| 속성 보유   | 일반 속성 없음                               | id, class, style 등 **속성 보유**                         |
| 탐색 범위   | 모든 노드 대상                               | 요소 노드 대상(주석/텍스트 제외)                          |

## 7) 실무 팁

- **텍스트/주석까지 포함해 순회**가 필요하면 `childNodes`, 요소만 다루면 `children`을 사용합니다.
- 문자열 치환·삽입이 필요하면 `innerHTML`(Element 전용), 안전하게 텍스트만 필요하면 `textContent`(Node 공통)를 사용합니다.
- 공백/줄바꿈이 `childNodes`에 **Text 노드**로 포함될 수 있으므로 DOM 비교/테스트 시 주의합니다.
