# useLayoutEffect와 useEffect 차이 정리

## 핵심 요약

- `useEffect`는 브라우저가 화면을 그린 뒤 실행되고, `useLayoutEffect`는 DOM 변경 직후 페인트 전에 실행된다.
- 그래서 핵심 차이는 `언제 실행되느냐`, 그리고 `화면 깜빡임에 영향을 줄 수 있느냐`다.
- 면접에서는 "`레이아웃 측정이나 즉시 보정이 필요하면 useLayoutEffect, 그 외 대부분의 부수 효과는 useEffect`"라고 정리하면 깔끔하다.
- `useLayoutEffect`를 남발하면 렌더링을 막아 성능에 불리할 수 있다는 점까지 같이 말해야 답변이 단단해진다.

## 둘 다 왜 필요한가

React에서 상태가 바뀌면 렌더링이 일어나고, 그 뒤에 로그를 찍거나 API를 호출하거나 DOM을 다루는 부수 효과가 필요할 때가 있다.  
이때 아무 훅이나 쓰는 게 아니라, `효과가 실행되는 시점`에 따라 훅을 나눠 둔 게 `useEffect`와 `useLayoutEffect`다.

즉 둘의 차이는 기능 자체보다 `브라우저 렌더링 타이밍과의 관계`에 있다.

## useEffect는 언제 실행되나

`useEffect`는 React가 변경 내용을 화면에 반영한 뒤 실행된다.

그래서 아래 같은 작업에 잘 맞는다.

- 데이터 요청
- 이벤트 리스너 등록
- 타이머 설정
- 로깅, 분석 스크립트 호출

이런 작업은 보통 "화면을 먼저 보여 주고 나서 해도 되는 일"이라서 `useEffect`가 기본 선택지가 된다.

## useLayoutEffect는 언제 실행되나

`useLayoutEffect`는 DOM 변경이 끝난 직후, 브라우저가 실제로 화면을 그리기 전에 실행된다.

그래서 다음처럼 "지금 DOM 상태를 읽고 바로 수정해야 하는 경우"에 쓴다.

- 요소 크기 측정
- 스크롤 위치 즉시 보정
- 툴팁, 모달 위치 계산
- 첫 페인트 전에 스타일이나 레이아웃을 맞춰야 하는 경우

이 훅 안에서 오래 걸리는 작업을 하면 브라우저가 페인트를 못 해서 사용자가 화면을 늦게 보게 된다.

## 왜 깜빡임 차이가 생기나

예를 들어 어떤 박스의 높이를 측정해서 위치를 다시 계산해야 한다고 해 보자.

- `useEffect`를 쓰면 일단 화면이 먼저 그려지고, 그다음에 위치를 수정한다.
- `useLayoutEffect`를 쓰면 그리기 전에 위치를 맞춘다.

그래서 `useEffect`는 잘못된 위치가 잠깐 보였다가 바뀌는 깜빡임이 생길 수 있고, `useLayoutEffect`는 그런 현상을 줄이는 데 유리하다.

## 언제 어떤 걸 쓰는 게 맞나

실무에서는 기본값을 `useEffect`로 두고, 정말 레이아웃 타이밍이 중요할 때만 `useLayoutEffect`로 올리는 편이 안전하다.

- 기본 부수 효과: `useEffect`
- 레이아웃 측정/즉시 보정: `useLayoutEffect`

면접에서 이 기준을 말하면 "둘 다 아는데 아무 데나 쓰진 않는다"는 인상을 줄 수 있다.

## 서버 렌더링에서는 어떤 점을 조심해야 하나

`useLayoutEffect`는 브라우저 레이아웃과 관련된 훅이라 서버 렌더링 환경에서는 경고 포인트가 될 수 있다.  
그래서 SSR 환경에서는 클라이언트에서만 실행되도록 분기하거나, 정말 필요한지 다시 보는 경우가 많다.

이 지점을 같이 말하면 단순 API 암기가 아니라 실제 사용 맥락을 이해하고 있다는 느낌을 준다.

## 간단한 예시

```tsx
function Tooltip() {
  const ref = useRef<HTMLDivElement | null>(null);
  const [top, setTop] = useState(0);

  useLayoutEffect(() => {
    if (!ref.current) return;
    const rect = ref.current.getBoundingClientRect();
    setTop(rect.bottom + 8);
  }, []);

  return <div ref={ref} style={{ top }}>툴팁</div>;
}
```

이 예시는 DOM 위치를 읽고 바로 보정해야 해서 `useLayoutEffect`가 더 자연스럽다.  
반대로 API 호출이었다면 `useEffect`가 맞다.

## 면접 답변 예시

> `useEffect`와 `useLayoutEffect`의 가장 큰 차이는 실행 시점입니다. `useEffect`는 브라우저가 화면을 그린 뒤 실행되고, `useLayoutEffect`는 DOM 변경 직후 페인트 전에 실행됩니다. 그래서 대부분의 부수 효과는 `useEffect`로 처리하고, 요소 크기 측정이나 위치 보정처럼 첫 화면이 그려지기 전에 맞춰야 하는 작업은 `useLayoutEffect`를 사용합니다. 다만 `useLayoutEffect`는 렌더링을 잠깐 막을 수 있어서 꼭 필요한 경우에만 쓰는 게 좋습니다.

## 장점

- React 렌더링 타이밍 이해도를 보여 주기 좋다.
- 브라우저 페인트, 깜빡임, DOM 측정까지 연결해서 설명하기 좋다.
- 단순 훅 사용법보다 한 단계 깊게 답변할 수 있다.

## 단점

- 둘을 "동일한데 이름만 다르다" 수준으로 말하면 바로 약점이 드러난다.
- `useLayoutEffect`를 과하게 쓰면 성능상 불리한데, 이 점을 빼면 설명이 반쪽짜리가 된다.
- SSR 경고 포인트를 모르면 실무 질문에서 흔들릴 수 있다.

## 주의사항 / 실무 팁

- 면접에서는 `화면 그리기 전/후` 타이밍 차이부터 말하면 설명이 안정적이다.
- 대부분은 `useEffect`로 충분하고, `useLayoutEffect`는 레이아웃 보정이 필요할 때만 쓴다고 정리하면 좋다.
- DOM 측정, 스크롤 보정, 위치 계산 같은 예시 하나를 준비해 두면 답변이 자연스럽다.
- 성능과 SSR 이슈를 한 문장 덧붙이면 실무 감각이 살아난다.
