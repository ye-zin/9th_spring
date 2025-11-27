### 객체 그래프 탐색(Object Graph Traversal)

**연관된 객체들 사이를 연결된 그래프 구조처럼 따라가며 접근하는 것**

---

## 객체 그래프(Object Graph)란?

엔티티들은 서로 연관 관계를 통해 연결되어 있음.

예를 들어:

```
회원(Member) → 주문(Order) → 주문상품(OrderItem) → 상품(Item)
```

이런 식으로 객체들이 연결되어 있는 구조를 **객체 그래프**라고 함.

---

## 객체 그래프 탐색이란?

`member.getOrders().get(0).getOrderItems().get(0).getItem().getName()`

👉 이렇게 객체 사이를 점(.)으로 타고 들어가며 필요한 데이터에 접근하는 걸 가리킴.

즉, **"연관된 객체에 접근할 수 있는가?"**가 핵심이라고 볼 수 있음.

---

## 문제가 왜 생기냐?

### 1) **지연 로딩(Lazy Loading)**

JPA는 성능 최적화를 위해 기본적으로 연관 객체 로딩을 **Lazy**로 설정함.

그래서 Member를 조회해도 Order는 바로 DB에서 가져오지 않고, `member.getOrders()`를 호출하는 순간 DB 쿼리가 실행됨.

→ 이것 때문에 **예상치 못한 시점에 여러 쿼리가 나가거나**,

→ **영속성 컨텍스트가 닫힌 상태(Session closed)**에서 탐색하면 `LazyInitializationException` 발생.

---

### 2) **N+1 문제**

예: 회원 목록을 10명 조회한 후 각각 주문 정보를 탐색한다면?

- 첫 번째 쿼리: 회원 목록 N개 조회
- 이후 각 회원에 대해 주문을 조회하는 쿼리가 N번 실행

→ 총 **N+1개의 쿼리**가 날아가 성능 문제가 발생.

---

## 💡 객체 그래프 탐색을 잘 하려면?

| 해결 방법 | 설명 |
| --- | --- |
| Fetch Join 사용 | JPQL에서 연관된 엔티티를 함께 가져오기 |
| EntityGraph 사용 | 조회 시점에 Fetch 전략 지정 |
| DTO Projection | 필요한 데이터만 SELECT |
| Batch Fetching 설정 | LAZY지만 한 번에 여러 레코드를 조회 |

---

## 예시 코드

```
Member member = memberRepository.findById(id)
        .orElseThrow();

// 객체 그래프 탐색
for (Order order : member.getOrders()) {
    for (OrderItem orderItem : order.getOrderItems()) {
        System.out.println(orderItem.getItem().getName());
    }
}
```

---

## 정리

객체 그래프 탐색이란:

> 연관된 객체 간 관계를 객체 모델 그대로 탐색해 필요한 데이터를 접근하는 과정
>

ORM이 추구하는 목표는 **SQL을 의식하지 않고 객체 그래프만 탐색해서 필요한 데이터를 사용할 수 있도록 하는 것**이지만,

- **지연 로딩**
- **N+1 문제**
- **영속성 컨텍스트 범위**

때문에 설계 없이 사용하면 문제가 생김.

그래서 설계 시 다음을 고려해야 함:

✔️ 어떤 연관관계를 LAZY로 둘 것인가

✔️ 어떤 시점에 fetch join 또는 EntityGraph를 사용할 것인가

✔️ 서비스/쿼리 조회용 DTO를 분리할 것인가