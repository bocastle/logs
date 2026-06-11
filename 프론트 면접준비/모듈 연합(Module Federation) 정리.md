# 모듈 연합(Module Federation) 정리

## 핵심 요약

- 모듈 연합(Module Federation)은 여러 번들 간에 코드를 런타임에서 공유할 수 있게 해 주는 **마이크로프론트엔드용 번들링 전략**이다.
- 대표적으로 Webpack 5에서 `remotes`, `exposes`, `shared` 설정을 사용해 앱 간 모듈 의존성을 동적으로 불러온다.
- 장점은 팀 단위 배포 독립성과 번들 중복 감소, 단점은 초기 런타임 복잡도와 버전 충돌 위험이다.
- 면접에서는 "빌드 타임 의존성"과 "런타임 의존성"의 차이를 분명히 설명하면 점수를 받는다.

## 개념 정리

마이크로프론트엔드에서 여러 팀이 각자 빌드한 화면/컴포넌트를 한 페이지에서 묶어 보여줘야 할 때, 이전 방식은 아래처럼 큰 번들 하나를 다시 배포해야 했다.

- 번들 A: 로그인/상품 목록
- 번들 B: 장바구니
- 번들 C: 결제

모듈 연합은 각 번들을 독립 저장소/브랜치로 운영하되, 런타임에 필요한 모듈을 가져와 연결한다.

핵심 동작:

- `exposes`: 현재 앱이 외부에 제공하는 모듈을 선언
- `remotes`: 외부 앱의 모듈을 가져올 주소를 선언
- `shared`: React, 공통 라이브러리 같은 의존성을 한 번만 로드하도록 공유

```js
// host webpack.config.js
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',
      remotes: {
        cart: 'cart@https://cdn.example.com/cart/remoteEntry.js',
      },
      shared: ['react', 'react-dom'],
    }),
  ],
}

// remote webpack.config.js
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'cart',
      filename: 'remoteEntry.js',
      exposes: {
        './OrderWidget': './src/components/OrderWidget',
      },
      shared: ['react', 'react-dom'],
    }),
  ],
}
```

사용 측면에서 `import('cart/OrderWidget')`처럼 동적으로 호출해 화면에 삽입한다.

## 면접 답변 예시

"Module Federation은 여러 독립 번들 간 런타임 모듈 교환을 가능하게 해 마이크로프론트엔드에서 배포 단위를 느슨하게 묶는 방식입니다. 호스트는 remoteEntry를 통해 필요한 모듈만 로드하고, shared 옵션으로 공통 라이브러리 버전을 통제해 번들 중복을 줄입니다. 다만 동작은 런타임 의존성 해결에 의존하므로, 공유 의존성 버전 정책과 SSR/보안 정책을 먼저 정해 운영해야 합니다."

## 장점

- 팀별 독립 배포 가능: 수정 범위를 줄여서 배포 리스크 감소
- 초기 로딩 개선: 필요한 리모트만 지연 로딩 가능
- 공통 라이브러리 공유로 번들 크기 최적화
- 기능 단위로 점진적 마이그레이션 가능

## 단점

- 런타임 로딩 실패 지점이 늘어나 장애 분석이 어려움
- 공유 의존성 버전 충돌 시 동작 예측이 어려움
- remoteEntry URL/도메인 운영 실패 시 특정 기능 전체가 무력화될 수 있음
- 번들러 교차 구성(webpack 버전, loader 설정) 관리 비용 증가

## 주의사항 / 실무 팁

- shared 버전은 `requiredVersion`/`singleton` 규칙을 팀 공통 가이드로 고정한다.
- remoteEntry는 CDN 캐시 정책과 롤백 전략을 별도 문서화한다.
- 장애 전파를 막기 위해 폴백 UI(빈 상태/에러 배너)를 넣는다.
- 모듈 계약(공개 인터페이스) 변경은 버전표기와 배포 노트를 필수로 남긴다.
