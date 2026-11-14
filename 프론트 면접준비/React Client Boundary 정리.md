# React Client Boundary 정리

## 핵심 요약

- 상호작용 버튼만 use client 경계로 두면 주변 설명과 데이터 조회 코드는 서버에 남아 초기 JavaScript가 줄어든다.
- 상위 layout에 use client를 붙이면 아래 import가 모두 클라이언트 번들 후보가 되어 경계가 예상보다 넓어진다.
- useState, event, browser API가 처음 필요한 leaf를 찾아 그 파일에서만 use client를 시작한다.

## 개념 설명

React Client Boundary는 서버 컴포넌트 트리 안에서 브라우저 상태와 이벤트가 필요한 지점을 클라이언트 번들 경계로 표시하는 기준이다.

`use client`가 붙은 파일 아래는 클라이언트로 전송되므로 경계를 넓히면 번들과 직렬화 제약이 커진다.

## 예시

```tsx
// Server Component
import LikeButton from "./LikeButton";
return <LikeButton postId={post.id} />;
```

상호작용이 필요한 버튼만 클라이언트 경계로 두면 주변 정적 콘텐츠는 서버에서 유지된다.

## 면접 답변 예시

> 서버가 렌더한 children을 클라이언트 shell의 slot으로 넘겨 정적 콘텐츠와 브라우저 state를 조합할 수 있다. 작은 파일마다 경계를 중복 선언하면 어느 모듈이 서버 전용인지 파악하기 어렵고 bundle 분할도 복잡해진다. 경계를 넘는 props를 serializable data와 server-rendered slot으로 제한하는 production build 테스트를 둔다.

## 장점

- 서버 전용 모듈과 자격 증명이 Client Boundary import graph에 들어가지 않아 비밀 노출을 막는다.

## 단점

- 함수와 class instance 같은 non-serializable prop을 서버에서 넘기면 RSC 전송 단계에서 실패한다.

## 주의사항 / 실무 팁

- bundle analyzer로 경계 아래 dependency를 확인하고 큰 서버용 패키지가 client chunk에 포함되지 않게 한다.
