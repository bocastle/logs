# JPA Fetch Join과 페이징을 함께 사용할 때 주의점 정리

## 핵심 요약

- `~ToMany` 관계에서 **Fetch Join + 페이징**을 함께 쓰면, JPA가 페이징을 DB에서 처리하지 못하고 **메모리에서 페이징(인메모리 페이징)** 을 수행할 수 있습니다.
- 이때 전체 결과를 메모리에 올린 뒤 가공하므로, 데이터가 많으면 **OutOfMemoryError(OOM)** 위험이 있습니다.
- 해결 방향: 컬렉션 Fetch Join을 피하고, `@BatchSize` 또는 `default_batch_fetch_size`로 **N+1을 완화**하면서 페이징은 DB에 맡깁니다.

## 문제 상황: `~ToMany`에서 Fetch Join + 페이징

예를 들어 `Product(1) - ProductCategory(N)` 관계가 있고, 아래처럼 컬렉션을 Fetch Join한 상태로 페이징(Slice)을 적용하면 문제가 생길 수 있습니다.

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToMany(mappedBy = "product", cascade = CascadeType.PERSIST)
    private List<ProductCategory> categories = new ArrayList<>();

    // ... 중략
}

@Entity
@Getter
@AllArgsConstructor
@NoArgsConstructor(access = AccessLevel.PROTECTED)
class ProductCategory {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    private Product product;

    @ManyToOne(fetch = FetchType.LAZY)
    private Category category;
}

interface ProductJpaRepository extends JpaRepository<Product, Long> {

    @Query(
        "select p " +
        "from Product p " +
        "join fetch p.categories pc " +
        "join fetch pc.category c " +
        "order by p.id desc"
    )
    Slice<Product> findProductWithSlice(Pageable pageable);
}
```

## 왜 OOM이 발생할 수 있나요?

### 경고 로그

`findProductWithSlice` 실행 시 아래 경고가 보일 수 있습니다.

```text
firstResult/maxResults specified with collection fetch; applying in memory
```

### 페이징 쿼리가 안 나가는 이유

실행된 쿼리를 확인해도 **limit/offset 같은 페이징이 적용되지 않을 수 있습니다.**

```sql
select
    p.id,
    pc.product_id,
    pc.id,
    c.id,
    c.name
from
   product p
join
   product_category pc
   on p.id = pc.product_id
join
   category c
   on c.id = pc.category_id
order by p.id desc
```

`ProductCategory`를 조인하면 `Product`의 결과도 함께 증가(카티션 프로덕트 성격)하여, 설정한 페이징 값이 의도대로 동작하기 어렵습니다.  
그래서 JPA가 **전체 결과를 메모리에 적재**한 뒤에 가공하여 페이징을 수행할 수 있고, 이때 데이터가 많으면 OOM이 발생할 가능성이 있습니다.

## 해결 방법

### 1) 컬렉션 Fetch Join을 피한다

단순히 Fetch Join을 사용하지 않으면 인메모리 페이징 문제는 피할 수 있습니다.  
다만 `categories`를 조회할 때 N+1 쿼리가 발생할 수 있습니다.

```java
Slice<Product> result = productJpaRepository.findProductWithSlice(pageRequest);
result.forEach(product -> System.out.println(product.getCategories())); // N + 1
```

### 2) `@BatchSize` / `default_batch_fetch_size`로 N+1을 완화한다

Fetch Join 대신 배치 페치 전략을 사용하면, Parent(Product)들의 식별자를 IN 절에 모아서 Child(ProductCategory)를 조회할 수 있습니다.

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @BatchSize(size = 10)
    @OneToMany(mappedBy = "product", cascade = CascadeType.PERSIST)
    private List<ProductCategory> categories = new ArrayList<>();

    // ... 중략
}
```

예를 들어 위처럼 `@BatchSize(size = 10)`을 적용하면 다음과 같은 쿼리가 발생할 수 있습니다.

```sql
select
    pc.product_id,
    pc.id,
    pc.category_id
from
    product_category pc
where
    pc.product_id in (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
```

## 장점

### `@BatchSize` / 배치 페치의 장점

- 컬렉션 Fetch Join을 피하면서도, N+1을 **완화**할 수 있습니다.
- 페이징은 DB에서 수행되고, 컬렉션 로딩은 배치로 처리되어 **OOM 위험을 줄일 수** 있습니다.

## 단점

### 배치 페치의 단점/한계

- `size` 설정에 따라 IN 절이 커질 수 있어, 상황에 맞는 튜닝이 필요합니다.
- “완전한 단일 쿼리로 한 번에 끝내기”와는 다른 전략이므로, 쿼리 패턴이 달라집니다.

## 주의사항 / 실무 팁

- `~ToMany` 컬렉션에 대해 **Fetch Join + 페이징**을 같이 쓰면 “인메모리 페이징”이 발생할 수 있음을 항상 염두에 두세요.
- 경고 로그 `applying in memory`가 보이면, 실제로 limit/offset이 DB에서 적용되는지 쿼리를 확인하세요.
- 페이징이 필요한 목록 조회는 “목록은 페이징으로 가져오고, 연관 컬렉션은 배치 페치로 가져오기”처럼 역할을 분리하는 전략이 안전합니다.
- `@BatchSize` 대신 전역 설정(`default_batch_fetch_size`)을 쓰는 방식도 있으니, 프로젝트 정책에 맞춰 일관성 있게 적용하세요.
