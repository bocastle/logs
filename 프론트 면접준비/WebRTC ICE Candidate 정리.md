# WebRTC ICE Candidate 정리

## 핵심 요약

- 서로 다른 NAT와 방화벽 환경에서 연결 가능한 경로를 탐색할 수 있다.
- TURN 구성이 없으면 기업망과 대칭 NAT에서 연결이 실패할 수 있다.
- STUN과 TURN 자격증명의 만료·회전 정책을 운영한다.

## 개념 설명

WebRTC ICE candidate는 두 peer가 직접 또는 중계 서버를 거쳐 도달할 수 있는 IP, port, transport 경로 후보를 표현하는 연결 정보다.

RTCPeerConnection은 host, server-reflexive, relay candidate를 수집하고 `icecandidate` 이벤트로 내보내며 상대는 받은 RTCIceCandidate를 `addIceCandidate`로 추가한다.

## 예시

```ts
pc.addEventListener("icecandidate", ({ candidate }) => {
  signaling.send({ type: "candidate", candidate });
});
signaling.on("candidate", async (payload) => {
  if (payload.candidate) {
    await pc.addIceCandidate(new RTCIceCandidate(payload.candidate));
  }
});
```

로컬 candidate를 signaling 채널로 보내고 상대 candidate를 RTCIceCandidate로 추가한다. 직접 연결이 막힌 환경에서는 TURN relay 후보가 필요하다.

## 면접 답변 예시

> ICE candidate는 두 peer가 직접 또는 TURN relay를 통해 연결할 수 있는 network 경로 후보입니다. Host, server-reflexive와 relay 후보를 signaling channel로 교환하고 remote description이 준비된 뒤 `addIceCandidate()`로 추가합니다. 기업망과 symmetric NAT에서는 TURN이 사실상 필요할 수 있어 candidate 유형별 연결 성공률과 relay 비용을 봅니다. Candidate 원문에는 network 정보가 들어가므로 log에는 유형과 오류 code만 남깁니다.

## 장점

- trickle ICE로 모든 후보 수집이 끝나기 전 연결을 시도할 수 있다.

## 단점

- remote description 전에 candidate를 잘못 처리하면 추가 순서 오류가 생길 수 있다.

## 주의사항 / 실무 팁

- remote description 준비 전 candidate를 queue에 보관한다.
