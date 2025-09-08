# dependency, devDependency, peerDependency 정리

## 1. dependency

- **프로덕션 환경에서도 필요한 패키지**
- 애플리케이션 실행에 필수적
- 배포 환경에서도 설치되며, 실행에 직접적인 영향을 줌  
  ➡️ 예시: `react`

---

## 2. devDependency

- **개발 과정에서만 필요한 패키지**
- 코드 품질 검증, 테스트, 번들링 등에 사용됨
- 실제 서비스 운영(프로덕션)에는 필요하지 않음  
  ➡️ 예시: `eslint`, `jest`, `webpack`

---

## 3. peerDependency

- **호환성을 위해 특정 버전의 다른 패키지를 요구**
- 주로 **라이브러리 개발** 시 중요
- 특정 패키지가 특정 버전의 다른 패키지와 함께 사용되어야 할 때 지정
- 동일한 패키지가 여러 번 설치되는 문제를 방지함  
  ➡️ 예시: `react-router-dom`이 `react` 특정 버전을 peerDependency로 지정

---

## 4. dependencies와 devDependencies를 잘못 설정했을 때 문제 🤔

- **devDependencies에 추가해야 할 패키지를 dependencies에 넣은 경우**

  - 불필요한 패키지가 프로덕션 환경에 포함됨
  - 애플리케이션 크기 증가 → 배포 속도 저하, 서버 부하 증가
  - 보안 취약점 증가 가능성

- **dependencies에 있어야 할 패키지를 devDependencies에 넣은 경우**
  - 실제 운영 환경에서 해당 패키지가 누락됨
  - 애플리케이션이 정상적으로 실행되지 않을 수 있음 🚨
