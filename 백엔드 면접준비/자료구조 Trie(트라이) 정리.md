# 자료구조 Trie(트라이) 정리

## 핵심 요약

- **Trie(트라이)** 는 문자열을 저장하고 **효율적으로 탐색**하기 위한 **트리 형태의 자료구조**입니다.
- 문자열을 단순 비교로 찾는 방식보다 탐색이 효율적일 수 있지만, 각 정점이 자식 링크를 많이 가지게 되어 **메모리를 더 사용**하는 경향이 있습니다.
- **검색어 자동완성**, **사전(딕셔너리) 검색** 같은 기능에서 자주 고려됩니다.

## 개요

자료구조 트라이(Trie)는 문자열을 저장하고 효율적으로 탐색하기 위한 트리 형태의 자료 구조입니다.  
트라이는 문자열을 탐색할 때 단순히 비교하는 것에 비해서 효율적으로 찾을 수 있지만, 각 정점이 자식에 대한 링크를 모두 가지고 있기 때문에 저장 공간을 더욱 많이 사용한다는 특징이 있습니다.  
주로, 검색어 자동완성이나 사전 찾기 기능을 구현할 때 트라이 자료구조를 고려할 수 있습니다.

## 트라이는 어떻게 구현할 수 있나요?

- 트라이 자료구조에서 **루트는 항상 비어있습니다.**
- 각 **간선(edge)** 은 추가될 **문자**를 **키(key)** 로 가집니다.
- 각 **정점(node)** 은 `이전 정점의 값 + 간선의 키`를 더한 결과를 값으로 가집니다.
- 구현 시에는 이러한 구조를 염두에 두면서 **해시 테이블**과 **연결 리스트**(혹은 그에 준하는 구조)로 구현할 수 있습니다.

### 예시 코드 (Java)

```java
class TrieTest {
    @Test
    void trieTest() {
        Trie trie = new Trie();
        trie.insert("maeilmail");
        assertThat(trie.has("ma")).isTrue();
        assertThat(trie.has("maeil")).isTrue();
        assertThat(trie.has("maeilmail")).isTrue();
        assertThat(trie.has("mail")).isFalse();
    }

    class Trie {

        private final Node root = new Node("");

        public void insert(String str) {
            Node current = root;
            for (String ch : str.split("")) {
                if (!current.children.containsKey(ch)) {
                    current.children.put(ch, new Node(current.value + ch));
                }
                current = current.children.get(ch);
            }
        }

        public boolean has(String str) {
            Node current = root;
            for (String ch : str.split("")) {
                if (!current.children.containsKey(ch)) {
                    return false;
                }
                current = current.children.get(ch);
            }
            return true;
        }
    }

    class Node {

        public String value;
        public Map<String, Node> children;

        public Node(String value) {
            this.value = value;
            this.children = new HashMap<>();
        }
    }
}
```

## 장점

- 문자열 탐색을 단순 비교보다 **효율적으로** 수행할 수 있습니다.
- **접두사(prefix)** 기반 기능(예: 자동완성, 타입어헤드 검색)에 적합합니다.

## 단점

- 각 정점이 자식에 대한 링크를 많이 가지게 되어 **저장 공간(메모리)을 더 사용**할 수 있습니다.

## 주의사항 및 실무 팁

- 트라이는 문자열 길이가 길거나 데이터가 매우 많을 때 **메모리 사용량**이 빠르게 증가할 수 있습니다.
  - 알파벳/문자 집합이 크면 자식 포인터(또는 맵) 관리 비용이 커질 수 있습니다.
- “정확히 일치하는 단어”뿐 아니라 “단어의 끝”을 구분해야 하는 경우가 많으므로,
  - 노드에 `isEnd` 같은 플래그를 추가해 **단어 종료 여부**를 표시하는 구현도 자주 사용됩니다.
- 자동완성 시스템에서는 Trie 단독이 아니라 **랭킹/빈도(Top-K)**, **캐시**, **분산 저장** 같은 요소와 함께 설계되는 경우가 많습니다.
