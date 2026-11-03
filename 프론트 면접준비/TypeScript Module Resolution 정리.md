# TypeScript Module Resolution 정리

## 핵심 요약

- moduleResolution을 실제 bundler와 맞추면 editor가 찾은 source와 production build가 실행할 파일이 일치한다.
- tsconfig paths는 출력 import를 다시 쓰지 않으므로 런타임 loader 설정이 없으면 컴파일만 통과하고 모듈을 찾지 못한다.
- `tsc --traceResolution`으로 문제 import가 어떤 후보와 package.json 조건을 거쳐 선택됐는지 확인한다.

## 개념 설명

Module Resolution은 import 경로가 실제 파일과 타입 선언으로 연결되는 방식을 정하는 규칙이다.

`moduleResolution`, `paths`, `exports` 조건이 런타임 번들러와 어긋나면 타입은 맞아도 실행에서 모듈을 못 찾을 수 있다.

## 예시

```json
{
  "compilerOptions": {
    "moduleResolution": "bundler",
    "paths": { "@/*": ["src/*"] }
  }
}
```

별칭과 해석 전략을 번들러 설정과 맞춰 타입 검사와 실제 빌드의 경로 판단을 일치시킨다.

## 면접 답변 예시

> paths 별칭을 일관되게 구성해 깊은 상대 경로를 줄이면서 프로젝트 내부 모듈 경계를 드러낸다. 동일 패키지의 중복 버전이 서로 다른 d.ts를 제공하면 구조가 비슷해도 private 타입 정체성이 충돌할 수 있다. 배포 package의 exports와 types 조건을 실제 Node 및 bundler fixture에서 import해 소비 환경을 검증한다.

## 장점

- package exports 조건을 이해하면 import와 require 및 types가 각 환경에 맞는 entry를 선택한다.

## 단점

- NodeNext에서 ESM 확장자와 package type을 무시하면 개발 도구와 Node 실행의 해석 결과가 달라진다.

## 주의사항 / 실무 팁

- TypeScript, test runner, bundler의 alias와 extension 설정을 하나의 source에서 파생하거나 동기화 테스트를 둔다.
