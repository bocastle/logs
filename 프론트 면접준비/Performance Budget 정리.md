# Performance Budget 정리

## 핵심 요약

- 성능 회귀를 release 전에 감지한다.
- 모든 page에 같은 예산을 적용하면 업무 중요도를 반영하지 못한다.
- page type과 하위 75분위 사용자 조건을 명시한다.

## 개념 설명

Performance Budget은 page type·device·network 조건별로 JavaScript byte, image byte, request 수, LCP·INP·CLS 최대치를 정하고 배포 관문에서 검증하는 성능 계약이다.

bundle analyzer와 CI lab test로 예방 예산을, RUM percentile로 실사용자 결과 예산을 본다. 목표와 차단 threshold를 따로 두고 예외에는 만료일을 둔다.

## 예시

```yaml
product-page:
  javascript_gzip_kb: 180
  lcp_p75_ms: 2500
  inp_p75_ms: 200
  cls_p75: 0.1
```

상품 페이지의 전송량과 Core Web Vitals p75 예산을 수치로 고정한다.

## 면접 답변 예시

> Performance budget은 page type과 device·network 조건별로 JavaScript 크기, request 수와 LCP·INP·CLS 상한을 합의한 성능 계약입니다. CI의 lab test와 bundle 분석은 release 전 회귀를 막고, RUM p75는 실제 사용자 분포에서 목표를 지키는지 확인하는 서로 다른 역할을 합니다. 모든 page에 같은 수치를 강제하지 않고 핵심 route와 사용자 segment에 맞춰 예산을 정하겠습니다. 초과 예외에는 owner, 이유와 만료일을 두어 임시 우회가 영구화되지 않게 합니다.

## 장점

- 팀 간 성능 트레이드오프를 숫자로 합의한다.

## 단점

- lab 수치만으로 실제 기기와 네트워크 분포를 놓친다.

## 주의사항 / 실무 팁

- byte budget과 user-centric metric budget을 함께 둔다.
