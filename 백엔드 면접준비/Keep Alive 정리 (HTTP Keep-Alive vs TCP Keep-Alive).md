# Keep Alive 정리 (HTTP Keep-Alive vs TCP Keep-Alive)

## 핵심 요약

- **Keep Alive**는 커넥션을 **끊지 않고 유지**해 재사용하거나(HTTP), 유휴 상태에서도 **연결 생존을 확인**하기(TCP) 위한 기술/설정입니다.
- **HTTP Keep-Alive**: 하나의 **TCP 커넥션을 재사용**해 여러 HTTP 요청/응답을 처리(HTTP/1.1에서 기본 활성화).
- **TCP Keep-Alive**: 커넥션이 유휴일 때 **주기적으로 패킷을 보내** 연결이 살아있는지 확인하고 유지합니다.

## Keep Alive란?

Keep Alive 는 네트워크 또는 시스템에서 **커넥션을 지속해서 유지**하기 위해 사용되는 기술이나 설정을 의미합니다.

### HTTP 프로토콜의 Keep-Alive

HTTP 프로토콜 에서 Keep-Alive는 **하나의 TCP 커넥션으로 여러 개의 HTTP 요청과 응답**을 주고받을 수 있도록 하는 기능입니다.

- HTTP/1.0: **요청마다** 새로운 커넥션을 열고 닫는 경우가 많았음
- HTTP/1.1: Keep-Alive가 **기본적으로 활성화**되어 커넥션을 재사용할 수 있음

### TCP 프로토콜의 Keep-Alive

TCP 프로토콜 에서 Keep-Alive는 커넥션이 **유휴 상태**일 때 커넥션이 끊어지지 않도록 **주기적으로 패킷을 전송**하는 기능입니다.

## Keep Alive의 장점

### 장점

- 커넥션을 재사용하여 **네트워크 비용을 절감**할 수 있습니다.
- handshake에 필요한 **RTT(Round Trip Time)**가 감소하여 **네트워크 지연 시간(Latency)**을 줄일 수 있습니다.
- handshake 과정에서 발생하는 **CPU, 메모리 등의 리소스 소비**를 줄일 수 있습니다.

## Keep Alive의 단점

### 단점

- 유휴 상태일 때에도 커넥션을 점유하고 있기 때문에 **서버의 소켓이 부족**해질 수 있습니다.
- DoS 공격으로 다수의 연결을 길게 유지하여 **서버를 과부하시킬** 수 있습니다.
- 타임아웃 설정이 적절하지 않으면 **커넥션 리소스가 낭비**될 수 있습니다.

## HTTP와 TCP의 Keep Alive는 어떤 차이가 있나요?

### HTTP Keep-Alive

클라이언트에서 일정 시간 동안 요청이 없으면 **타임아웃만큼 커넥션을 유지**하고, 타임아웃이 지나면 **커넥션이 끊어집니다**.

### TCP Keep-Alive

커넥션이 유휴 상태일 때 **주기적으로 패킷을 전송**해서 커넥션이 살아있음을 확인하고, 살아있으면 커넥션을 **지속해서 유지**합니다.

## 주의사항 및 실무 팁

### 1) HTTP Keep-Alive 확인/활용 포인트

- 응답 헤더에서 커넥션 재사용 여부를 확인할 수 있습니다.
- 환경/프록시/서버 설정에 따라 타임아웃 값이 달라질 수 있으니, **지표(에러율/지연/연결 수)**와 함께 조정하는 편이 안전합니다.

```http
# 예시: HTTP/1.1 환경에서 자주 볼 수 있는 헤더
Connection: keep-alive
Keep-Alive: timeout=5, max=1000
```

### 2) 서버 자원(소켓/FD) 관점에서의 튜닝

Keep-Alive를 길게 잡으면 유휴 커넥션이 늘어 **FD(파일 디스크립터)/소켓**을 많이 점유할 수 있습니다.  
트래픽 패턴(짧고 잦은 요청 vs 긴 연결)과 서버 자원 한계를 고려해 타임아웃을 조절합니다.

### 3) TCP Keep-Alive(리눅스) 관련 설정 예시

TCP keepalive는 OS 레벨 설정의 영향을 받습니다(서비스/환경에 따라 다름).

```bash
# 현재 설정 확인
sysctl net.ipv4.tcp_keepalive_time
sysctl net.ipv4.tcp_keepalive_intvl
sysctl net.ipv4.tcp_keepalive_probes

# 예시: 값을 변경(운영 적용 전 반드시 영향도 검증 필요)
sudo sysctl -w net.ipv4.tcp_keepalive_time=600
sudo sysctl -w net.ipv4.tcp_keepalive_intvl=60
sudo sysctl -w net.ipv4.tcp_keepalive_probes=5
```

## 추가 학습 자료

- Stack Overflow - Does a TCP socket connection have a "keep alive"?
- MDN Web Docs - Keep-Alive
