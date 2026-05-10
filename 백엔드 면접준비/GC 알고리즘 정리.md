# GC 알고리즘 정리

## 핵심 요약

- 자바 가비지 컬렉터(GC)는 목표(처리량 vs 지연시간)와 환경(힙 크기, CPU 코어 수)에 따라 여러 알고리즘이 존재합니다.
- 대표 GC: **Serial, Parallel, Parallel Old, CMS, G1, ZGC, Shenandoah, Epsilon**
- 실행 중인 JVM에서 어떤 GC가 선택되었는지는 `jcmd`, `jinfo` 등으로 확인할 수 있습니다.

## GC 알고리즘은 어떤 것이 있나요?

### Serial GC

- JDK에 도입된 최초의 가비지 컬렉터
- **단일 스레드**로 동작하는 가장 단순한 형태
- **작은 힙 메모리**와 **단일 CPU 환경**에 적합
- Stop-The-World(STW) 시간이 가장 길게 발생

### Parallel GC (Throughput GC)

- Java 5~8에서 기본(default) 가비지 컬렉터로 사용
- Serial GC와 달리 **Young 영역 GC를 멀티 스레드**로 수행
- **처리량(Throughput)** 에 초점을 두기 때문에 Throughput GC라고도 불림

### Parallel Old GC

- Parallel GC의 향상된 버전
- **Old 영역에서도 멀티 스레드**를 활용하여 GC 수행

### CMS(Concurrent Mark-Sweep) GC

- Java 5~8까지 사용된 가비지 컬렉터
- 애플리케이션 스레드와 **병렬(동시)로 실행**되어 STW 시간을 최소화하도록 설계
- 단, 메모리/CPU 사용량이 많고 **메모리 압축을 수행하지 않아 단편화 문제**가 있음
- Java 9부터 deprecated, Java 14에서 제거

### G1(Garbage First) GC

- Java 9부터 기본(default) 가비지 컬렉터
- 힙을 여러 개의 **region**으로 나누어 논리적으로 Young/Old를 구분
- 처리량과 STW 시간 사이의 **균형**을 유지
- **32GB보다 작은 힙 메모리**에서 가장 효과적이라고 설명됨
- GC 대상이 많은 region을 먼저 회수하기 때문에 “Garbage First”라는 이름이 붙음

### ZGC

- Java 11부터 도입
- **10ms 이하의 STW 시간**과 **대용량 힙 처리**를 목표로 설계

### Shenandoah GC

- Red Hat에서 개발, Java 12부터 도입
- G1처럼 힙을 region으로 나누어 처리
- ZGC처럼 **저지연(STW 최소화)** 과 **대용량 힙 처리**를 목표로 함

### Epsilon GC

- Java 11부터 도입
- **GC 기능이 없는(메모리 회수 없음) 실험용** 가비지 컬렉터
- GC 영향을 분리한 성능 테스트, GC 오버헤드 없이 메모리 한계 테스트 등에 사용
- 프로덕션 환경에는 적합하지 않음

## G1 GC에서 Humongous 객체란 무엇이며 어떻게 처리되나요?

### Humongous 객체란?

- **region 크기의 50% 이상**을 차지하는 큰 객체를 의미합니다.

### 처리 방식과 영향

- 크기에 따라 **하나 또는 여러 개의 연속된 region**을 차지할 수 있습니다.
- region 내 잉여 공간은 다른 객체에 할당되지 않아 **메모리 단편화**가 발생할 수 있습니다.
- Young 영역을 거치지 않고 바로 Old 영역에 할당되므로 **Full GC 가능성이 높아질 수** 있습니다.

### 대응 방법(예시)

- `-XX:G1HeapRegionSize` 옵션으로 region 크기를 조정
- 큰 객체를 더 작은 객체로 **분할**해 처리하는 전략 고려

```bash
# 예: G1 region 크기 조정(값은 환경에 맞게)
-XX:G1HeapRegionSize=8m
```

## 2vCPU, 1GB 메모리 Linux 서버에 JDK 17 설치 시 어떤 GC가 사용되나요?

- JDK 9부터 기본 GC는 G1이지만, **서버 스펙에 따라 자동으로 결정**될 수 있습니다.
- OpenJDK에서는 CPU 코어 수가 2개 이상이고 메모리가 2GB 이상이면 **Server-Class Machine**으로 인식합니다.
- 해당 조건을 만족하지 않으면 **Serial GC가 선택**될 수 있습니다.
- G1을 사용하려면 서버 스케일업 또는 `-XX:+UseG1GC` 옵션을 명시적으로 설정할 수 있습니다.

```bash
# G1 GC 명시
-XX:+UseG1GC
```

## 실행 중인 JVM의 GC 확인 방법

아래 명령으로 현재 JVM이 사용하는 GC 정보를 확인할 수 있습니다.

```bash
sudo jcmd {jar PID} VM.info
```

또는

```bash
sudo jinfo {jar PID}
```

## 장점

### (여러 GC 알고리즘을 제공하는 것의) 장점

- 처리량 중심(Parallel) vs 저지연 중심(ZGC/Shenandoah) 등 **요구사항에 맞춘 선택**이 가능합니다.
- 힙 크기/코어 수/서비스 특성에 따라 **튜닝 여지**가 있습니다.

## 단점

### (알고리즘 선택/튜닝의) 단점

- GC 종류에 따라 특성이 달라, 잘못 선택하면 **지연 증가(STW)** 또는 **리소스(메모리/CPU) 소모 증가**가 발생할 수 있습니다.
- 운영 환경에서 “기본값이 항상 최선”은 아니므로 관측/튜닝 비용이 들 수 있습니다.

## 주의사항 / 실무 팁

- GC는 “이름”만으로 결정하기보다, **서비스 요구(처리량/지연), 힙 크기, 코어 수**를 기준으로 선택하는 게 안전합니다.
- G1을 사용할 때 Humongous 객체가 많다면 단편화/Full GC 위험이 커질 수 있으니, 객체 크기 분포를 점검하고 필요하면 region 크기/객체 설계를 조정하세요.
- 스레드풀/컨테이너/클라우드 환경에서는 CPU/메모리 제한이 GC 선택에 영향을 줄 수 있으니, 배포 환경에서 실제 선택된 GC를 반드시 확인하세요.
