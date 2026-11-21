# React Server-Client Component 경계 정리

## 핵심 요약

- 상품 조회를 Server Component에 두면 데이터베이스 client와 fetch 로직이 브라우저 JavaScript에서 제외된다.
- 서버 컴포넌트에서 event handler나 browser API를 사용하려 하면 실행 환경과 기능 책임이 충돌한다.
- 기본 컴포넌트는 서버에 두고 useState, effect, event가 필요한 가장 작은 subtree만 별도 client module로 만든다.

## 개념 설명

React Server-Client Component 경계는 데이터 조회와 정적 렌더링은 서버에 두고 이벤트, 브라우저 API, local state는 클라이언트에 두는 구분이다.

서버에서 클라이언트로 넘기는 props는 직렬화 가능해야 하며 클라이언트 경계가 넓을수록 JavaScript 비용이 증가한다.

## 예시

```tsx
// Server
const product = await getProduct(id);
return <AddToCartClient productId={product.id} />;
```

상품 데이터 조회는 서버에 남기고 클릭 이벤트가 필요한 장바구니 버튼만 클라이언트로 보낸다.

## 면접 답변 예시

> 서버가 준비한 데이터를 serializable props로 전달해 클라이언트의 중복 fetch와 loading waterfall을 줄인다. 함수, stream, class instance를 Server-Client prop으로 넘기면 전송 가능한 RSC 값 계약을 위반한다. client shell의 children slot에 서버 렌더 결과를 합성해 상호작용 wrapper 때문에 콘텐츠까지 클라이언트화하지 않는다.

## 장점

- 장바구니 버튼만 클라이언트로 보내면 정적 상세 콘텐츠는 서버 HTML로 빠르게 표시되고 필요한 영역만 hydrate된다.

## 단점

- 클라이언트 경계를 layout 상단으로 올리면 아래 dependency와 렌더 코드가 큰 client bundle에 포함된다.

## 주의사항 / 실무 팁

- 경계 props 목록을 검토해 원시값과 plain data만 남기고 서버에서 계산 가능한 표현은 미리 계산한다.
