# JCF 자료구조 초기 용량 지정 정리

## 핵심 요약

JCF(Java Collection Framework)에서 `ArrayList`, `HashMap` 같은 **가변 크기 자료구조**를 사용할 때 **초기 용량(capacity)** 을 적절히 지정하면 **리사이징 횟수를 줄여** 메모리 낭비와 연산 비용을 절약할 수 있습니다.

## JCF 자료구조의 초기 용량을 지정하면 좋은 점

JCF에서 **ArrayList**를 기준으로 설명하겠습니다.

- ArrayList의 기본 용량(capacity)은 **10**입니다.
- 용량이 가득 차면 기존 크기의 **1.5배(`oldCapacity + (oldCapacity >> 1)`)** 로 증가합니다.

예를 들어, `MAX = 5,000,000`일 때 기본 설정으로 리스트를 생성하면 여러 번의 리사이징이 발생해 최종 `capacity`가 **6,153,400**까지 증가하고, 약 **70MB**의 메모리를 사용합니다.

반면, `new ArrayList(MAX)`로 **초기 용량을 설정**하면 불필요한 리사이징 없이 **5,000,000** 크기로 고정되며, 약 **20MB**의 메모리만 사용하게 됩니다.

```java
public class Main {

    private static final int MAX = 5_000_000;

    public static void main(String[] args) {
        MemoryMXBean memoryMXBean = ManagementFactory.getMemoryMXBean();
        printUsedHeap(1, memoryMXBean);

        List<String> arr = new ArrayList<>();
        for (int i = 0; i < MAX; i++) {
            arr.add("a");
        }

        printUsedHeap(2, memoryMXBean);
        printUsedHeap(3, memoryMXBean);
    }

    private static void printUsedHeap(int logIndex, MemoryMXBean memoryMXBean) {
        MemoryUsage heapUsage = memoryMXBean.getHeapMemoryUsage();
        long used = heapUsage.getUsed();
        System.out.println("[" + logIndex + "] " + "Used Heap Memory: " + used / 1024 / 1024 + " MB");
    }
}
```

정리하자면, JCF에서 가변 크기의 자료 구조를 사용하는 경우, 초기 용량을 설정하면 **리사이징을 줄이고** **메모리와 연산 비용을 절약**할 수 있습니다.

## 로드 팩터와 임계점이란 무엇인가요?

- **로드 팩터(load factor)**: 특정 크기의 자료 구조에 데이터가 어느 정도 적재되었는지 나타내는 비율입니다.
- 가변적인 크기를 가진 자료구조에서 크기를 증가시켜야 하는 **임계점(Threshold)** 을 계산하기 위해 사용됩니다.

예를 들어, JCF에서 **HashMap**의 경우:

- 내부적으로 배열을 사용하며, 초기 사이즈는 **16**입니다.
- HashMap의 기준 로드 팩터는 **0.75**이므로 임계점은 **12**입니다.
  - 계산: `threshold = capacity * load factor = 16 * 0.75 = 12`
- 만약 HashMap 내부 배열의 사이즈가 **12를 넘는 경우**, 내부 배열의 크기를 **2배** 늘리고 **재해싱(rehashing)** 을 수행합니다.

## 장점

- **리사이징 감소**: 동적 확장 과정(배열 재할당/복사) 횟수가 줄어듭니다.
- **메모리 효율 개선**: 필요 이상으로 capacity가 커지는 것을 방지할 수 있습니다.
- **성능 개선**: 리사이징/재해싱에 드는 연산 비용을 줄일 수 있습니다.

## 단점

- **과도한 초기 용량 지정 시 메모리 낭비**: 실제 데이터가 예상보다 적으면 사용하지 않는 공간이 커질 수 있습니다.
- **추정이 빗나갈 수 있음**: 데이터 규모를 정확히 예측하기 어려운 경우가 있습니다.

## 주의사항 및 실무 팁

- 데이터 규모가 **대략적으로라도 예측 가능**하면 `ArrayList(initialCapacity)`, `HashMap(initialCapacity)` 같은 생성자를 활용하는 편이 유리합니다.
- `HashMap`은 **초기 capacity** 뿐 아니라 **load factor(기본 0.75)** 에 의해 리사이징 시점(임계점)이 결정됩니다.
  - 대량 데이터를 넣을 예정이면 capacity와 임계점 관계를 고려하세요.
- 반대로, 데이터가 매우 유동적이거나 상한을 예측하기 어렵다면 **기본 설정**이 더 안전할 수 있습니다.
