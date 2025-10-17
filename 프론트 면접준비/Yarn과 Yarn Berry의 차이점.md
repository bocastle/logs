# 🧶 Yarn과 Yarn Berry의 차이점

Yarn과 Yarn Berry는 모두 패키지 매니저이지만, 큰 차이점이 존재합니다.  
바로, **버전과 구조의 차이**입니다.

Yarn은 **version 1.x 시리즈**를 의미하며, **npm과 유사한 구조**를 가집니다.  
Yarn Berry는 **version 2.x 이상**을 의미하며, **완전히 새롭게 재설계된 버전**입니다.

---

## 🤔 Yarn과 Yarn Berry는 어떤 점에서 다른가요?

가장 큰 부분은 **Plug'n'Play(PnP)** 방식이라고 말씀드릴 수 있습니다.

Yarn은 **node_modules**를 통해 의존성을 관리합니다.  
반면, Yarn Berry는 **PnP 방식을 기본으로 채택**하여,  
`node_modules` 없이 `.pnp.cjs` 파일을 따로 생성하여 의존성을 관리합니다.

이로 인해 **설치 속도가 빠르고 디스크 공간을 절약**할 수 있습니다.

또한 **Zero-Install**을 지원한다는 점에서 다릅니다.  
Yarn Berry는 Zero-Install을 지원하여, **의존성을 저장소에 커밋**할 수 있습니다.  
이를 통해 **CI/CD에서 별도의 설치 과정 없이 바로 코드를 실행**할 수 있습니다.

---

## 🤔 CI/CD에서 별도의 설치 과정 없이 바로 코드를 실행할 수 있다고 하셨는데 어떻게 가능한 건가요?

Yarn Berry는 모든 의존성을 **.yarn/cache 폴더에 압축된 형태**로 저장합니다.  
이 압축 파일들을 **Git 저장소에 직접 커밋**합니다.  
`.pnp.cjs` 파일도 함께 커밋되어 **의존성 매핑 정보를 포함**합니다.

그리하여 **CI/CD 파이프라인에서의 동작에서 yarn install의 과정이 생략**될 수 있습니다.

---

## 💡 예시

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
      - run: yarn test # yarn install 없이 바로 실행 가능
```

## 기조방식

- run: yarn install #이 단계가 필요
- run: yarn test

그리하여 **CI/CD에서 `yarn install` 단계가 생략되어 빌드 시간이 크게 단축**되고,  
**모든 개발자와 CI 환경에서 정확히 동일한 의존성**을 사용할 수 있습니다.
