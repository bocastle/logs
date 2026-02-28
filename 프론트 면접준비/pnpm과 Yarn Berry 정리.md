# pnpm과 Yarn Berry 정리

## 핵심 요약

- **pnpm**: 전역 저장소(**content-addressable store**)에 패키지를 보관하고, 프로젝트에서는 **하드 링크/심볼릭 링크**로 참조해 **설치 속도**와 **디스크 효율**을 개선한 패키지 매니저입니다.
- **Yarn Berry(Yarn 2+)**: **PnP(Plug'n'Play)** 와 **Zero Install**을 통해 `node_modules` 없이 의존성을 관리하고, 캐시를 **zip 형태**로 보관해 **정확성/속도/용량**을 개선한 패키지 매니저입니다.

## pnpm

pnpm은 npm에 비해 성능과 디스크 효율성을 개선한 패키지 매니저입니다.

일반적으로 npm이나 yarn을 사용하면 프로젝트의 `node_modules` 폴더에 각 패키지의 실제 복사본이 저장됩니다. 하지만 pnpm은 **content-addressable store**라는 전역 저장소에 패키지를 보관한 후, 각 프로젝트에서는 **하드 링크**와 **심볼릭 링크**를 활용해 해당 패키지의 **링크만 참조**하도록 합니다. 이로 인해 동일한 패키지를 여러 프로젝트에서 공유할 수 있어 설치 속도가 크게 향상되고 스토리지 용량이 줄어듭니다.

이외에도, **유령 의존성(phantom dependency)** 문제가 발생하지 않으며, **workspace** 기능을 통해 한 프로젝트에 여러 패키지가 있는 **모노레포(monorepo)** 를 구성하는 데 활용할 수 있다는 장점이 있습니다.

### 예시 명령어

```bash
# 의존성 설치
pnpm install

# 패키지 추가
pnpm add <package>

# 개발 의존성 추가
pnpm add -D <package>

# 전역 저장소(스토어) 경로 확인
pnpm store path

# 워크스페이스(모노레포)에서 패키지 실행 예시
pnpm -r run build
```

## Yarn Berry (Yarn 2+)

Yarn Berry는 Yarn의 2.0 버전 이상을 가리키며, **PnP(Plug'n'Play)** 와 **Zero Install**을 도입하여 패키지 관리 방식을 개선한 패키지 매니저입니다.

기존 `node_modules` 폴더를 없애고 `.pnp.cjs` 파일을 이용하여 의존성을 관리하는 것이 특징입니다. `.pnp.cjs`에는 패키지의 위치와 의존성 정보들이 완전하게 기술되어 있어 패키지를 찾는 과정이 효율적이고 정확하게 이뤄집니다.

그리고 패키지들이 `.yarn/cache` 디렉토리에 **zip 압축 파일 형태**로 관리되어 용량이 작다는 장점이 있습니다. 이로 인해 패키지 설치 속도가 빠르며 스토리지 용량을 아낄 수 있습니다.

또한 용량이 크게 줄어들었기 때문에 의존성들을 버전 관리에 포함시켜 git으로 관리할 수 있게 되었습니다. 이렇게 의존성을 버전 관리에 포함시키는 것을 **Zero Install**이라고 부릅니다. 이를 통해 저장소를 새로 복제하거나 브랜치를 이동할 때 의존성을 설치할 필요가 없게 되었습니다.

이외에도 Yarn Berry 역시 **유령 의존성**이 발생하지 않으며, **workspace** 기능을 지원한다는 장점을 가지고 있습니다.

### 예시 명령어/설정

```bash
# Yarn Berry(2+)로 전환(프로젝트 기준)
yarn set version berry

# 의존성 설치
yarn install
```

```yaml
# .yarnrc.yml (예시)
nodeLinker: pnp

# Zero Install을 하려면 보통 캐시/설정 파일들을 VCS에 포함합니다.
# (프로젝트 정책에 따라 .yarn/cache 등을 커밋)
```

## 장점

### pnpm 장점

- 전역 저장소 + 링크 기반 구조로 **설치 속도 향상** 및 **스토리지 절감**
- **유령 의존성 문제 방지**
- **workspace**로 모노레포 구성에 유용

### Yarn Berry 장점

- `node_modules` 없이 `.pnp.cjs`로 의존성 정보를 명확하게 관리해 **정확하고 효율적인 패키지 탐색**
- `.yarn/cache`에 zip 형태로 관리되어 **용량 절감** 및 **설치 속도 향상**
- **Zero Install**로 의존성을 버전 관리에 포함 가능(클론/브랜치 이동 시 설치 부담 감소)
- **유령 의존성 문제 방지**, **workspace** 지원

## 단점

### pnpm 단점(실무에서 흔히 겪는 포인트)

- 링크 기반 구조로 인해 일부 툴/스크립트가 `node_modules`의 “물리적 복사본” 구조를 가정할 경우 추가 설정이 필요할 수 있음

### Yarn Berry 단점(실무에서 흔히 겪는 포인트)

- PnP는 `node_modules`를 전제로 만든 일부 라이브러리/도구와의 호환 이슈가 발생할 수 있어, 필요 시 별도 설정이나 대응이 필요할 수 있음
- Zero Install 도입 시 캐시 파일을 저장소에 포함시키는 정책/용량/리뷰 부담이 생길 수 있음

## 주의사항 및 실무 팁

- **팀 합의가 먼저**: 패키지 매니저 변경은 CI, 캐시, 린트/테스트, IDE 설정까지 영향을 줄 수 있으므로 표준을 정해 일관되게 운영합니다.
- **모노레포 운영 시**: `workspace` 기능(공통 스크립트, 의존성 호이스팅/격리 정책 등)을 함께 정리하면 생산성이 올라갑니다.
- **호환성 체크**: 특정 번들러/테스트 러너/IDE 플러그인 등이 `node_modules`를 가정하는지 확인하고, 문제가 생기면 프로젝트 문서에 해결 방법을 남기는 것이 좋습니다.

## 한눈에 비교

- 저장 방식
  - pnpm: 전역 스토어에 보관 후 프로젝트에서 링크로 참조
  - Yarn Berry: PnP + `.pnp.cjs`로 의존성 해석, `.yarn/cache`에 zip 보관
- `node_modules`
  - pnpm: 존재(링크 중심 구성)
  - Yarn Berry: 기본적으로 제거(PnP)
- 대표 키워드
  - pnpm: content-addressable store, hard link/symlink, workspace
  - Yarn Berry: PnP, `.pnp.cjs`, `.yarn/cache`, Zero Install, workspace
