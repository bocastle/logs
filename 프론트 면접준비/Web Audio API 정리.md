# Web Audio API 정리

## 핵심 요약

- 커스텀 믹서와 시각화, 필터를 만들 수 있다.
- 자동 재생 정책 때문에 소리가 바로 나지 않을 수 있다.
- 사용자 제스처 후 AudioContext를 resume한다.

## 개념 설명

Web Audio API는 브라우저에서 오디오 노드 그래프를 만들어 생성, 믹싱, 필터링, 분석을 수행하는 저수준 오디오 API다.

`AudioContext` 안에서 source, GainNode, analyser, destination 같은 노드를 연결하고, 자동 재생 정책 때문에 사용자 제스처 후 resume해야 할 수 있다.

## 예시

```ts
const ctx = new AudioContext();
const source = ctx.createMediaElementSource(audio);
const gain = ctx.createGain();
source.connect(gain).connect(ctx.destination);
gain.gain.value = 0.7;

playButton.addEventListener("click", async () => {
  try {
    await ctx.resume();
    await audio.play();
  } catch (error) {
    showAudioError(error);
  }
});
```

media element를 오디오 그래프에 연결해 볼륨을 제어한다. 처음에는 AudioContext가 suspended일 수 있어 resume 흐름이 필요하다.

## 면접 답변 예시

> Web Audio API는 source, gain, filter와 analyser를 node graph로 연결해 실시간 audio를 처리하는 저수준 API입니다. Browser autoplay 정책 때문에 context가 suspended일 수 있어 사용자가 재생한 동작 안에서 `resume()`하고 실패도 처리하겠습니다. 같은 media element로 source node를 반복 생성하지 않고 graph와 resource 수명을 명확히 관리해야 합니다. 저지연 품질은 device와 browser마다 달라 `latencyHint`, sample rate와 실제 출력 지연을 측정하고 화면 이탈 시 node를 disconnect하거나 context를 정리합니다.

## 장점

- 오디오 처리 그래프를 노드 단위로 구성할 수 있다.

## 단점

- 노드 연결 해제를 놓치면 메모리와 CPU가 낭비된다.

## 주의사항 / 실무 팁

- 화면 이탈 시 source와 노드를 disconnect한다.
