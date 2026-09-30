# CSS text-overflow와 overflow-wrap 정리

## 핵심 요약

- 텍스트 오버플로우 요구를 생략과 줄바꿈으로 구분한다.
- ellipsis는 전체 값을 숨겨 업무 판단을 방해할 수 있다.
- 전체 값이 필요한 필드에 tooltip만 의존하지 않고 복사·세부 경로를 제공한다.

## 개념 설명

CSS `text-overflow`는 inline 진행 방향으로 clip된 텍스트의 표시 방식을, `overflow-wrap`은 긴 단어·URL을 새 줄에서 쪼개 overflow를 피할지 정한다.

single-line ellipsis는 보통 `white-space: nowrap`, `overflow: hidden`, `text-overflow: ellipsis`를 조합한다. `overflow-wrap: anywhere`는 배치할 일반 break point가 없을 때 임의 위치에서 줄바꿈을 허용한다.

## 예시

```css
.filename { white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.comment { overflow-wrap: anywhere; }
```

파일명은 한 줄에서 ellipsis로 생략하고 댓글의 긴 URL은 줄바꿈해 컨테이너를 유지한다.

## 면접 답변 예시

> `text-overflow`는 잘린 text를 ellipsis로 표시하는 정책이고 `overflow-wrap`은 긴 token을 다음 줄로 나누는 정책이라 요구가 다릅니다. 한 줄 ellipsis에는 실제 width 제약, `white-space: nowrap`과 overflow 설정이 함께 필요합니다. 전체 값이 업무 판단에 중요하면 hover tooltip만 두지 않고 focus 가능한 펼치기, 복사나 상세 경로를 제공하겠습니다. URL, hash, filename과 한국어 문장을 각각 넣어 생략과 줄바꿈 결과를 확인합니다.

## 장점

- 긴 URL과 token이 card 폭을 깨뜨리는 문제를 줄인다.

## 단점

- `anywhere`는 읽기 어려운 위치에서 단어를 쪼갤 수 있다.

## 주의사항 / 실무 팁

- 일반 문장은 언어 기반 줄바꿈을 유지한다.
