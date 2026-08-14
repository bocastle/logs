# Media Session API 정리

## 핵심 요약

- 잠금 화면과 외부 미디어 키에서 자연스럽게 제어할 수 있다.
- 재생 상태와 metadata가 어긋나면 사용자 혼란이 생긴다.
- 재생, 일시정지, 트랙 변경 때 metadata를 같이 갱신한다.

## 개념 설명

Media Session API는 브라우저와 운영체제의 미디어 컨트롤 영역에 제목, 아티스트, 앨범아트, 재생 액션을 연결하는 API다.

`navigator.mediaSession.metadata`와 action handler를 설정하면 잠금 화면이나 헤드셋 버튼에서 웹 앱 재생을 제어할 수 있다.

## 예시

```ts
navigator.mediaSession.metadata = new MediaMetadata({
  title: "Daily Briefing",
  artist: "Bocelog",
});
navigator.mediaSession.setActionHandler("pause", () => player.pause());
```

웹 플레이어 상태와 OS 미디어 컨트롤을 맞추는 예다. 실제 재생 상태 변경과 metadata 갱신이 함께 움직여야 한다.

## 면접 답변 예시

> Media Session API는 현재 media의 metadata와 재생 action을 browser·OS control에 연결하는 기능입니다. 실제 player의 재생·일시정지와 track 변경 때 `metadata`, playback state와 position을 함께 갱신하겠습니다. Platform마다 지원 action과 표시 방식이 달라 app 내부 player를 기본 조작 경로로 유지합니다. 잠금 화면에 제목과 artwork가 노출돼도 되는 콘텐츠인지 개인정보와 저작권 정책도 확인합니다.

## 장점

- 브라우저 탭 밖에서도 현재 콘텐츠를 설명할 수 있다.

## 단점

- 지원 액션과 표시 방식이 플랫폼마다 다르다.

## 주의사항 / 실무 팁

- 잠금 화면 노출 가능한 정보인지 확인한다.
