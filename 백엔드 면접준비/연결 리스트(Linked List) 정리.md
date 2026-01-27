# 연결 리스트(Linked List) 정리

## 개요

연결 리스트는 **노드(Node)** 들이 **포인터(참조)** 로 서로 연결된 선형 자료구조입니다. 각 노드는 `데이터`와 `다음 노드에 대한 참조`(단일), 또는 `이전/다음 노드 참조`(이중)를 가집니다. 첫 노드는 **HEAD**, 마지막 노드는 **TAIL**이라 부릅니다. 배열과 달리 **메모리 상 연속될 필요가 없고**, 삽입/삭제 시 **재배치가 필요 없습니다**.

- 탐색: `O(n)` — 앞에서부터 순차 접근
- (위치가 주어진) 삽입/삭제: `O(1)` — 참조만 바꿔 연결
- 메모리: 포인터 오버헤드 존재
- 캐시 친화성: 배열보다 낮음

## 변형

- **Singly Linked List(단일)**: `next` 만 보유. 구현 간단, 역방향 탐색 불가.
- **Doubly Linked List(이중)**: `prev`, `next` 보유. 양방향 이동·중간 삭제 용이, 메모리/연산 오버헤드 증가.
- **Circular Linked List(환형)**: 마지막이 처음을 가리킴. 순환 순회 편리, 종료 조건 주의.

## 단일 연결 리스트의 핵심 연산

> 아래 복잡도는 “삽입/삭제 **위치의 노드**를 이미 알고 있는” 경우 기준입니다.

| 연산      | 설명                                                   | 시간 복잡도 |
| --------- | ------------------------------------------------------ | ----------- |
| 탐색      | 값/인덱스를 찾기 위해 앞에서부터 순회                  | `O(n)`      |
| 머리 삽입 | `new.next = head; head = new` (비어있다면 tail도 갱신) | `O(1)`      |
| 중간 삽입 | `prev.next` 를 `new` 로, `new.next` 를 기존 다음으로   | `O(1)`      |
| 머리 삭제 | `head = head.next` (비어지면 tail = null)              | `O(1)`      |
| 중간 삭제 | `prev.next = prev.next.next`                           | `O(1)`      |
| 꼬리 삽입 | `tail.next = new; tail = new` (빈 리스트 처리)         | `O(1)`      |

> **주의**: “3번 위치에 삽입”처럼 **위치 탐색이 동반**되면 총 시간은 `O(n)` 입니다.

## 제공하신 예제 코드 개선 포인트 (Java)

```java
class Node {
    Node next;
    int value;
    Node(Node next, int value) { this.next = next; this.value = value; }
}

class SinglyLinkedList {
    public Node head;
    public Node tail;

    // 꼬리 삽입: O(1)
    public Node insert(int newValue) {
        Node newNode = new Node(null, newValue);
        if (head == null) {
            head = tail = newNode;       // 빈 리스트 처리
        } else {
            tail.next = newNode;
            tail = newNode;
        }
        return newNode;
    }

    // 탐색: O(n) — NPE 방지
    public Node find(int findValue) {
        for (Node cur = head; cur != null; cur = cur.next) {
            if (cur.value == findValue) return cur;
        }
        return null; // 없으면 null 반환
    }

    // prev 뒤에 삽입: O(1)
    public void appendNext(Node prevNode, int value) {
        if (prevNode == null) throw new IllegalArgumentException("prevNode is null");
        Node newNode = new Node(prevNode.next, value);
        prevNode.next = newNode;
        if (tail == prevNode) tail = newNode; // tail 갱신 누락 방지
    }

    // prev 다음 노드 삭제: O(1)
    public void deleteNext(Node prevNode) {
        if (prevNode == null || prevNode.next == null) return;
        Node toDelete = prevNode.next;
        prevNode.next = toDelete.next;
        if (toDelete == tail) tail = prevNode; // tail 갱신
        // GC가 처리하므로 명시 해제는 불필요
    }

    // 머리 삭제: O(1)
    public void deleteHead() {
        if (head == null) return;
        head = head.next;
        if (head == null) tail = null;
    }
}
```

### 주요 버그/주의사항

1. **`find`의 NPE 위험**: `currentNode.value` 접근 전에 `currentNode` null 체크 필요.
2. **검증-복사 순서**: 불변성 검증 중 외부에서 구조가 바뀌지 않도록, **먼저 복사(방어적 복사) 후 검증**이 안전합니다.
3. **tail 갱신 누락**: `appendNext`, `deleteNext`, `deleteHead` 등에서 `tail` 변경 케이스 반영 필요.
4. **빈 리스트/단일 노드 처리**: 경계 조건을 분기 처리해야 합니다.
5. **반복 종료 조건 명확화**: 순환 리스트에서는 무한 루프 방지 장치 필요.

## 배열 vs 연결 리스트

| 항목           | 배열(Array)       | 연결 리스트(Linked List)                       |
| -------------- | ----------------- | ---------------------------------------------- |
| 인덱스 접근    | `O(1)`            | `O(n)`                                         |
| 중간 삽입/삭제 | `O(n)` (밀어내기) | `O(1)` (노드 참조 교체, 위치를 알고 있는 경우) |
| 메모리 배치    | 연속              | 비연속(포인터 추가 메모리)                     |
| 캐시 효율      | 높음              | 낮음                                           |
| 크기 변경      | 재할당/복사 필요  | 동적 확장 용이                                 |

## 언제 연결 리스트를 쓸까요?

- **빈번한 중간 삽입/삭제**가 있고, **해당 위치 노드**를 빠르게 참조할 수 있을 때
- **크기 가변** 자료구조가 필요할 때
- 하지만, 일반적인 탐색/순회·캐시 효율이 중요한 경우 **배열/동적 배열(ArrayList)** 이 실무에서 더 효율적일 때가 많습니다.

## 시간 복잡도 요약

- 탐색/접근: `O(n)`
- 머리/꼬리 삽입/삭제: `O(1)` (꼬리 참조 유지 시)
- 중간 삽입/삭제: `O(1)` (이전 노드 참조를 이미 알고 있을 때)
