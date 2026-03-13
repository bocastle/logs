# Next.js에서 기본 `<img>` 대신 `next/image`를 쓰는 이유 정리

## 핵심 요약

- `next/image`는 **placeholder**, **자동 이미지 최적화(포맷/리사이징)**, **기본 lazy loading**을 제공해 **UX와 성능을 빠르게 개선**할 수 있습니다.
- 반면 **설정이 다소 복잡**할 수 있고(외부 도메인 허용 등), **서버 측 이미지 최적화** 특성상 호스팅/운영 환경 제약이 생길 수 있습니다.

## 배경

프론트엔드에서 이미지는 **로딩 UX**와 **초기 성능(전송량/렌더링)**에 큰 영향을 줍니다.  
기본 `<img>` 태그만으로도 구현은 가능하지만, `next/image`는 이런 최적화들을 **프레임워크 레벨에서 기본 제공**해 개발 비용을 줄입니다.

## 왜 `next/image`를 사용하나요?

### 1) 로딩 UX 개선: `placeholder`

- 기본 `<img>`는 로딩 중 **빈 공간**이 노출될 수 있습니다.
- `next/image`는 `placeholder` 옵션을 통해 로딩 전/중에 **대체 UI(예: blur)** 를 보여줘 시각적 일관성을 유지할 수 있습니다.

```tsx
import Image from "next/image";

export default function Example() {
  return (
    <Image
      src="/banner.jpg"
      alt="banner"
      width={1200}
      height={600}
      placeholder="blur"
      blurDataURL="data:image/jpeg;base64,..." // 또는 자동 생성된 blur 데이터 사용
    />
  );
}
```

### 2) 자동 이미지 최적화: 포맷/크기 최적화

- 브라우저가 `WebP`를 지원하면 `JPG/PNG` 대신 **`WebP`를 자동 제공**할 수 있습니다.
- 요청된 크기에 맞춰 **적절한 사이즈로 서빙**해 불필요한 데이터 전송을 줄여줍니다.
- 개발자가 별도 파이프라인을 구축하지 않아도 **기본 성능 개선**을 기대할 수 있습니다.

### 3) 기본 lazy loading

- `next/image`는 기본적으로 **뷰포트에 들어올 때 필요한 이미지만 로드**하도록 동작합니다.
- 별도 설정 없이도 초기 로딩 속도 개선 및 네트워크 리소스 절약에 도움이 됩니다.

## 장점

- **빠른 성능 최적화 구현**: 포맷 변환/리사이징/지연 로딩 등을 기본 제공
- **UX 개선**: `placeholder`로 로딩 중 레이아웃/시각적 일관성 유지에 유리
- **운영 비용 절감**: 이미지 최적화를 직접 구현/관리하는 부담이 감소

## 단점

- **설정이 상대적으로 복잡**할 수 있음
  - 외부 도메인 이미지 사용 시 `next.config.js`에서 **허용 목록 설정** 필요
- **서버 측 최적화 제약**
  - `next/image`는 기본적으로 서버 사이드에서 최적화를 수행하므로, 자체 서버 운영이 어렵거나 환경이 제한적이면 **Vercel 같은 호스팅**을 고려해야 할 수 있음

## `next/image` 없이도 최적화할 수 있나요?

가능합니다. 예를 들어:

- 이미지 크기를 미리 조절해 **정적 파일로 제공**
- `<img loading="lazy" />`로 **지연 로딩** 적용
- Cloudinary, Imgix 같은 **이미지 최적화 서비스** 활용

다만 `next/image`는 이러한 기능을 **추가 노력 없이 기본 제공**한다는 점에서, 시간/리소스 제약을 극복하기 위한 선택지가 될 수 있습니다.

## 주의사항 및 실무 팁

### 외부 도메인 이미지를 쓸 때: `next.config.js` 설정

외부 이미지 호스트를 사용할 경우, 도메인(또는 remotePatterns)을 허용해야 합니다.

```js
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    domains: ["images.example.com"], // 간단 허용
    // 또는 더 엄격히:
    // remotePatterns: [
    //   { protocol: "https", hostname: "images.example.com", pathname: "/**" },
    // ],
  },
};

module.exports = nextConfig;
```

### `"null"` 같은 잘못된 src를 주지 않기

`src` 값이 조건에 따라 바뀌는 경우, **빈 문자열/잘못된 URL**이 들어가지 않도록 방어 로직을 두는 것이 좋습니다.

```tsx
const src = imageUrl ?? "/fallback.png";
return <Image src={src} alt="thumbnail" width={320} height={180} />;
```

## 참고 자료

- Next.js Image 문서(한글): nextjs-ko.org Image
- Kakao Entertainment: Next/Image를 활용한 이미지 최적화 (fe-developers.kakaoent.com)
