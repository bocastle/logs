# React Controlled Input 성능 정리

## 핵심 요약

- controlled input은 React state가 표시 값의 단일 원본이어서 validation과 submit payload를 같은 값에서 계산한다.
- 상위 페이지가 input state를 소유하고 큰 목록을 함께 렌더하면 한 글자마다 전체 tree의 계산 비용을 지불한다.
- input의 원본 state는 동기 갱신하고 서버 검색이나 차트처럼 느린 소비자에만 deferred 값을 전달한다.

## 개념 설명

React Controlled Input 성능은 입력 값이 상태를 거쳐 렌더링될 때 타이핑 반응성을 유지하는 문제다.

무거운 결과 계산을 입력 컴포넌트와 분리하거나 deferred value, transition으로 낮은 우선순위로 보낸다.

## 예시

```tsx
const [text, setText] = useState("");
const deferredText = useDeferredValue(text);
return <Results query={deferredText} />;
```

input 값은 즉시 갱신하고 검색 결과는 지연시켜 타이핑 끊김을 줄인다.

## 면접 답변 예시

> 필드 state를 작은 경계에 두면 키 입력마다 다시 렌더되는 컴포넌트 수를 제한할 수 있다. controlled value 업데이트 자체를 transition으로 보내면 DOM 입력과 state 반영 시점이 어긋난다. 한글 IME, 긴 붙여넣기, 모바일 키보드에서 caret과 글자 조합이 유지되는지 실제 브라우저로 검사한다.

## 장점

- useDeferredValue로 검색 결과만 늦추면 input value와 caret은 즉시 갱신하면서 비싼 subtree 작업을 분리한다.

## 단점

- onChange 값을 비동기로 변환해 다시 설정하면 중간 렌더에서 caret 위치와 IME 조합 입력이 깨질 수 있다.

## 주의사항 / 실무 팁

- Profiler의 키 입력 interaction에서 어떤 ancestor가 재렌더되는지 확인해 state를 가장 가까운 필드로 내린다.
