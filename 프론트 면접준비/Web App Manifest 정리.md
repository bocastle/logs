# Web App Manifest 정리

## 핵심 요약

- 설치 아이콘과 앱 이름을 운영체제 런처에 제공할 수 있다.
- scope 밖 링크는 브라우저 탭으로 열려 앱 흐름이 끊길 수 있다.
- manifest 응답의 Content-Type과 링크 경로를 배포 환경에서 확인한다.

## 개념 설명

Web App Manifest는 설치된 웹 앱의 이름, 아이콘, 시작 주소, 탐색 범위와 display 모드를 JSON으로 설명하는 애플리케이션 메타데이터다.

문서의 link rel=manifest가 파일을 가리키면 브라우저가 `start_url`, `scope`, `display`, icons를 읽어 홈 화면과 앱 창의 실행 방식을 결정한다.

## 예시

```json
{
  "id": "/",
  "name": "Bocelog",
  "start_url": "/?source=installed",
  "scope": "/",
  "display": "standalone",
  "icons": [
    { "src": "/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icon-maskable.png", "sizes": "512x512", "purpose": "maskable" }
  ]
}
```

manifest에 안정적인 id와 start_url, scope, standalone display, 일반·maskable icon을 명시한다. start_url은 인증되지 않은 실행도 처리해야 한다.

## 면접 답변 예시

> manifest id로 시작 URL query가 바뀌어도 앱 정체성을 유지할 수 있다. start_url이 네트워크나 로그인에만 의존하면 설치 직후 빈 화면이 된다. start_url에서 오프라인, 로그아웃, 갱신 중 상태를 모두 처리한다.

## 장점

- standalone 창의 시작 경로와 탐색 범위를 통제할 수 있다.

## 단점

- 불투명 영역이 없는 아이콘은 플랫폼 마스킹에서 잘릴 수 있다.

## 주의사항 / 실무 팁

- 일반 icon과 maskable icon을 실제 런처 모양으로 검사한다.
