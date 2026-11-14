# BroadcastChannel 보안 주의점 정리

## 핵심 요약

- 채널 데이터 분류 기준을 정하면 민감정보 전파를 예방할 수 있다.
- XSS가 발생한 같은 origin 문서는 채널 메시지를 읽고 위조할 수 있다.
- payload에는 UI 동기화에 필요한 최소 enum과 식별자만 둔다.

## 개념 설명

BroadcastChannel 보안은 같은 origin 안의 모든 참여 컨텍스트를 신뢰 경계로 보고 채널 payload가 비밀 저장소나 권한 부여 수단이 되지 않게 하는 일이다.

채널 이름은 접근 제어가 아니며 같은 origin의 iframe과 worker도 수신할 수 있으므로 token 대신 상태 변화 종류를 보내고 payload schema를 수신 시 검증해야 한다.

## 예시

```ts
const channel = new BroadcastChannel("bocelog:preferences:v1");
channel.onmessage = ({ data }) => {
  if (!data || data.type !== "theme" || !["light", "dark"].includes(data.value)) return;
  applyTheme(data.value);
};
// token, cookie, 개인 데이터는 payload에 넣지 않는다.
```

same origin 송신자라는 사실만으로 payload를 신뢰하지 않고 허용된 type과 value를 검사한다. 인증 token은 채널에 복제하지 않는다.

## 면접 답변 예시

> 기능별 채널 이름으로 우발적인 메시지 충돌을 줄인다. 채널 이름을 비밀값처럼 사용하면 잘못된 보안 가정이 생긴다. 권한이 필요한 동작은 수신 메시지가 아니라 서버 상태를 다시 검증해 결정한다.

## 장점

- schema 검증으로 손상되거나 오래된 메시지의 영향을 제한한다.

## 단점

- 사용자 식별 정보가 payload에 있으면 열린 모든 참여자에게 복제된다.

## 주의사항 / 실무 팁

- 메시지 type별 런타임 schema와 버전을 확인한다.
