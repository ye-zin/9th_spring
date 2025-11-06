# QueryDSL에서 FetchJoin 하는 법

  ## **Fetch Join이란?**

  JPA에서 연관된 엔티티를 가져올 때, `LAZY` 로딩일 경우 실제로 접근하기 전까지는 쿼리가 나가지 않아 **N+1 문제**가 발생할 수 있음.

  → 이를 해결하기 위해 **JPQL의 fetch join** 을 사용하여 **한 번의 쿼리로 연관된 엔티티까지 모두 가져오는 방법.**

  ## QueryDSL에서 Fetch Join 사용 패턴

  ### 1) 기본 문법
    .from(엔티티)
    .join(엔티티.연관필드, 별칭).fetchJoin()

  ## 예시: Member → Team (다대일 연관)

  ### 엔티티 예시
    @Entity
    public class Member {
        @Id @GeneratedValue
        private Long id;
    
        private String username;
    
        @ManyToOne(fetch = FetchType.LAZY)
        private Team team;
    }

  ### **Fetch Join 적용 QueryDSL**
    public Member findMemberWithTeam(Long memberId) {
        return queryFactory
                .selectFrom(member)
                **.join(member.team, team).fetchJoin()**   
                .where(member.id.eq(memberId))
                .fetchOne();
    }
  → 이렇게 하면 **member**를 가져올 때 **team**도 함께 한 번에 조회됨 (N+1 발생 X)

# DTO 매핑 방식 (+DTO안에 DTO)

  ## DTO(Data Transfer Object)란?
  - **계층 간 데이터 전달을 위한 전용 객체**
  - 엔티티(Entity)를 그대로 노출하지 않고, 필요한 값만 담아 전달
  - 엔티티 변경 → 외부 응답 구조가 바뀌지 않도록 **추상화 계층** 역할 수행

  ## 언제 DTO로 매핑할까?

  ### **DTO 매핑이 필요한 이유**
  - **API 응답/뷰 모델**: 엔티티 노출 방지, 필드 선별, 네이밍 변경
  - **쿼리 성능 최적화**: 필요한 필드만 select
  - **계층 분리**: 외부 계약(API 스펙)과 내부 엔티티 분리

  | 이유 | 설명 |
      | --- | --- |
  | 엔티티 직접 노출 위험 | Lazy 로딩 발생, 양방향 연관 → 순환 참조 → API 응답 문제 |
  | 성능 최적화 | 필요 필드만 SELECT 가능 (데이터 전송량 절약) |
  | 역할 분리 | 도메인 엔티티는 비즈니스 규칙, DTO는 표현/전달 역할 |

  ## DTO 안에 DTO (중첩 DTO) 구조

  연관된 객체를 **그대로 DTO로 변환해서 포함**하는 방식

  ## 상황 비유 (진짜 쉽게)

  **주문 정보**를 사용자에게 보여주고 싶다고 해보자.

  그런데 주문에는 이런 정보가 있어:
  - 주문 번호
  - 주문한 사람 정보
  - 주문한 상품 목록

  즉, **주문(Order)** 안에 **사람(Member)** 과 **상품들(List<Item>)** 이 들어있지?

  이걸 그대로 DTO로 표현하면,

  **OrderDTO 안에 MemberDTO + List<ItemDTO>** 가 들어가는 구조가 됨.

  ## 진짜 쉬운 DTO 예시 (일반적인 class 기반)

  ### 1) MemberDTO: 주문한 사람 정보
    public class MemberDTO {
        private String name;
        private String phone;
    
        public MemberDTO(String name, String phone) {
            this.name = name;
            this.phone = phone;
        }
    
        public String getName() { return name; }
        public String getPhone() { return phone; }
    }

  ### 2) ItemDTO: 상품 한 개 정보
    public class ItemDTO {
        private String itemName;
        private int price;
    
        public ItemDTO(String itemName, int price) {
            this.itemName = itemName;
            this.price = price;
        }
    
        public String getItemName() { return itemName; }
        public int getPrice() { return price; }
    }

  ### 3) OrderDTO: 주문 정보 (여기 **DTO 안에 DTO** 구조가 등장!)
    public class OrderDTO {
        private Long orderId;
        private MemberDTO member;          // ← DTO 안에 DTO
        private List<ItemDTO> items;       // ← DTO 안에 DTO 리스트
    
    		// 생성자
        public OrderDTO(Long orderId, MemberDTO member, List<ItemDTO> items) {
            this.orderId = orderId;
            this.member = member;
            this.items = items;
        }
    
    		// Getter/Setter
        public Long getOrderId() { return orderId; }
        public MemberDTO getMember() { return member; }
        public List<ItemDTO> getItems() { return items; }
    }

  ## 그럼 어떻게 **사용**하냐?

  ### 1) 내부 DTO들 먼저 생성
    MemberDTO member = new MemberDTO("홍길동", "010-1234-5678");
    
    List<ItemDTO> itemList = List.of(
            new ItemDTO("라면", 1200),
            new ItemDTO("김밥", 3000)
    );

  ### 2) OrderDTO 안에 넣어서 생성
    OrderDTO order = new OrderDTO(1001L, member, itemList);


  ### 3) 사용(출력 예시)
    System.out.println(order.getOrderId());             // 1001
    System.out.println(order.getMember().getName());    // 홍길동
    System.out.println(order.getItems().get(0).getItemName()); // 라면

  ## 정리 한 줄
  DTO 안에 DTO는, 한 객체가 다른 객체를 포함하는 관계를 API 응답 구조로 그대로 표현하는 것.
  - OrderDTO = 큰 틀(응답 상위)
  - MemberDTO + ItemDTO = 포함된 정보들
  - JSON으로 변환하면 자동으로 아래처럼 계층 구조가 만들어짐:
  ### 예시
    {
        "orderId": 1001,
        "member": {
        "name": "홍길동",
        "phone": "010-1234-5678"
        },
        "items": [
            { "itemName": "라면", "price": 1200 },
            { "itemName": "김밥", "price": 3000 }
        ]
    }

  ## **DTO 안에 DTO (중첩) 매핑 시 주의할 점**

  | 주의 사항 | 이유 |
      | --- | --- |
  | 연관 데이터를 무조건 Fetch Join/Projection으로 가져오기 | Lazy 초기화 예외 방지 |
  | 컬렉션 중첩 시 중복 row 발생 주의 | `distinct` 또는 메모리 그룹핑 필요 |
  | 페이징 시 컬렉션 fetch join 금지 | 데이터 폭증 + 메모리 페이징 발생 |

  -> **중첩 구조는 “쿼리 + 매핑 + 성능” 세 가지를 항상 함께 고려해야 함**

  ## 중첩 DTO 매핑 전략 (핵심만 정리)

  | 상황 | 추천 전략 |
      | --- | --- |
  | 단일 연관 (ManyToOne) | Fetch Join 후 DTO 생성 |
  | 컬렉션 포함 + 페이징 없음 | 단일 쿼리 → 결과를 메모리에서 그룹핑 |
  | 컬렉션 포함 + 페이징 필요 | 부모 페이징 → 자식 IN 조회 → 그룹핑 (표준 실무) |

  ## 최종 정리

  | 핵심 개념 | 정리 |
      | --- | --- |
  | DTO | 계층 간 전달 객체, 엔티티 직접 노출 방지 |
  | DTO 안에 DTO | 연관 구조를 표현하는 계층형 응답 모델 |
  | 매핑 방식 | 직접 new / ModelMapper / MapStruct / QueryDSL Projection |
  | 중첩 DTO 조회 | Lazy 이슈, 중복 Row, 페이징 고려 필요 |

# 커스텀 페이지네이션

  ## 언제, 왜 커스텀 페이지네이션 사용하는가?
  - 기본 `Pageable`/`Page<T>`만으로는 **대용량·무한스크롤·정렬 안정성** 요구 충족이 어려움.
  - **카운트 쿼리 비용↓**, **무한스크롤 UX**, **정렬 컬럼 다중/복합키**, **필터 조합** 등을 제어하려고 직접 설계.

  ## 대표 방식 비교

  | 방식 | URL 형태 | 장점 | 단점 | 언제 쓰나 |
      | --- | --- | --- | --- | --- |
  | **Offset/Page 기반** | `?page=3&size=20` | 구현 쉬움, 임의 접근 | 오프셋 커지면 느림 | 관리 도구, 소량 데이터 |
  | **Keyset(Seek) 기반** | `?lastId=105&size=20` | 빠름(인덱스 탐색), 카운트 불필요 | 임의 페이지 점프 어려움 | 타임라인/피드 |
  | **Cursor 기반** | `?cursor=BASE64(...)` | 복합 정렬/필터 안전, 양방향 | 구현 복잡 | 무한스크롤, API 공개 |
  | **Slice** (스프링) | 다음 있음 여부만 | 카운트 쿼리 없음 | 전체 페이지수 불명 | 무한스크롤 리스트 |

  핵심: **오프셋은 간단하지만 느릴 수 있고**, **키셋/커서는 빠르고 안정적**이다(대신 임의 접근 희생).

  ## 설계 핵심 포인트
  - **정렬 안정성**: 정렬 컬럼이 중복될 수 있으면 **타이브레이커(보통 PK)**를 두고 복합 정렬 사용.
  - **불변 정렬**: 페이지 전환 중 데이터가 바뀌어도 순서가 흔들리지 않도록 정렬 컬럼은 **인덱스 + 단조 증가/감소**를 선호.
  - **카운트 최적화**: 전체 건수 필요 없으면 `Slice`/`hasNext`로 대체. 필요하면 **가벼운 count 쿼리**를 분리(필요 시 `approximate count` 전략).
  - **필터와 인덱스**: where절과 정렬 컬럼에 **복합 인덱스** 설계.
  - **경계조건**: `size+1` 로 조회 후 초과분으로 `hasNext` 판정.
  - **직렬화**: 커서는 **정렬 키들(예: createdAt, id)** 을 Base64(JSON)로 인코딩/복호화.
  - **양방향 이동**: 이전 페이지도 필요하면 **역방향 커서**를 함께 설계하거나, 두 번 조회 후 역정렬.

# transform - groupBy

  ## `groupBy()` + `transform()` 이 필요한 이유

  일반적으로 **조인 후 OneToMany 구조(부모 + 자식 컬렉션)** 를 그대로 DTO로 매핑하면 이렇게 문제가 생김:

  | 문제 | 설명 |
      | --- | --- |
  | 중복 Row 발생 | 부모 1행 + 자식 N → 결과는 N개의 Row로 늘어남 |
  | DTO를 그대로 만들면 | 부모 DTO가 N번 생성되고 items 리스트도 계속 덮어쓰기 위험 |
  | 결국 | **자바 코드로 다시 그룹핑해서** 부모 1개 + 자식 리스트 모아야 함 |

  → 이 **그룹핑 작업을 QueryDSL이 자동으로 해주는 기능**이 **`transform()` + `groupBy()`**.

  ## 핵심 개념

    ```
    // 구조
    query
        .from(...)
        .join(...)
        .transform(
            groupBy(그룹기준).as(그룹 결과 형태)
        );
    ```

    - `groupBy(키)` : **부모 기준으로 묶어라**
    - `.as()` : 묶인 데이터를 어떤 **DTO 형태로 조립할지 정의**
    - `transform()` : 최종 DTO 컬렉션을 **한 번에 return**

  ## 장점 & 단점

  | 장점 | 설명 |
      | --- | --- |
  | DTO 조립 자동화 | 자바에서 수동으로 Map/GroupBy 할 필요 없음 |
  | 읽기 쉽다 | “부모 1개 + 자식 리스트” 형태가 바로 보임 |
  | 복잡한 매핑 구조 대응 | 중첩 DTO 구조도 손쉽게 표현 가능 |

  | 단점 | 설명 |
      | --- | --- |
  | 성능 고려 필요 | 여전히 JOIN 기반 → Row 수가 많으면 메모리 사용됨 |
  | 페이징 불가 | 그룹핑은 DB가 아니라 **JVM 메모리에서 수행됨** |
  | 복잡 join이 많으면 코드 가독성 떨어질 수 있음 |  |

  즉, 목록 + 페이징 시 → groupBy 대신 두 번 조회(IN 전략) 을 쓴다 (실전 표준 패턴)

  ## 언제 쓰는 것이 좋은가?

  | 상황 | 선택 |
      | --- | --- |
  | 컬렉션 포함된 조회, 페이징 필요 없음 | ✅ `groupBy` + `transform` 추천 |
  | 부모 페이징 + 자식 포함해야 함 | ❌ `groupBy` X → **두 번 조회(IN)** 전략 |
  | 단순 단건 상세 조회 | ✅ fetch join + 단순 DTO 매핑 |

  ## 한 줄 요약

  transform(groupBy()) 는 조인으로 인해 뻥튀기된 조회 결과를 **자동으로 부모-자식 구조로 묶어주는 QueryDSL의 DTO 조립 도구**.
  단, **페이징에는 부적합**하므로 목록 API에서는 주의해야 한다.

# order by null

  **`ORDER BY NULL`**은 SQL에서 꽤 자주 쓰이는데 처음 보면 “NULL로 정렬하라고?” 라고 이해할 수 있지만**의도는 간단함 → “정렬을 아예 하지 말아라”** 라는 의미.

  ## order by null 의 의미

  | 표현 | 실제 의미 |
      | --- | --- |
  | `ORDER BY column` | **정렬 수행** (인덱스 or filesort) |
  | `ORDER BY NULL` | **정렬을 하지 않는다 (정렬 작업을 완전히 무시)** |

  즉, **DB가 정렬 단계(filesort)를 수행하지 않도록 강제하는 문법**

  ## 왜 쓰는가?

  ### ✔ 불필요한 정렬 → 성능 낭비이기 때문

  예를 들어, 우리가 이미 결과를 **GROUP BY** 등을 통해 요약했는데

  추가 정렬이 필요 없을 수 있음.

  하지만 SQL 엔진은 기본적으로 `GROUP BY` 하면 “정렬된 결과”를 만드는 경향이 있음.

  → 이때 **정렬을 무시하도록** 하는 명령이 `ORDER BY NULL`.

  ## 예시
    SELECT department, COUNT(*)
    FROM employees
    GROUP BY department
    ORDER BY NULL;   -- 정렬 없이 결과만 반환

  - `GROUP BY` 자체가 내부적으로 정렬을 수반할 수 있음 (MySQL, MariaDB 등)
  - 하지만 우리는 정렬이 필요 없다면 → **즉시 RETURN (I/O 감소, CPU 감소)**

  ## MySQL 실행 계획 관점

  `ORDER BY NULL` 을 쓰면:

  | 항목 | 있음 (`ORDER BY col`) | 없음 (`ORDER BY NULL`) |
      | --- | --- | --- |
  | filesort 단계 | 수행됨 | **제거됨** |
  | CPU 사용량 | 증가 | **감소** |
  | 결과 순서 | 정렬됨 | “원본/해시/그룹 결과 순서” 그대로 |

  →  특히, **대량 데이터**에서 매우 큰 차이를 만듦.

  ## 언제 유용한가?

  | 상황 | 이유 | 해결 |
      | --- | --- | --- |
  | GROUP BY 결과를 단순 조회 | 정렬 불필요 | `ORDER BY NULL` 로 sort 제거 |
  | COUNT(*) / SUM() 등 집계만 조회 | 순서 의미 없음 | 정렬 비용 줄일 수 있음 |
  | DISTINCT 결과 | DISTINCT도 내부적으로 정렬/해시 수행 → **정렬 제거 가능** |  |

  ## 반대로, 다음 상황에서는 쓰면 안 됨

  | 상황 | 이유 |
      | --- | --- |
  | API 응답에서 **순서가 중요**한 경우 | 정렬이 필요함 |
  | Keyset Pagination / Cursor 기반 | 정렬이 반드시 필요함 |
  | 결과 비교/처리 시 순서가 의미를 가지는 경우 | 정렬 제거하면 논리 깨짐 |

  → 즉, **목록 화면**이나 **정렬된 게시글 조회**에서는 절대 사용하면 안 됨.

  ## 한 줄 요약

  order by null은 **정렬을 완전히 제거하여 불필요한 인덱스로 정렬되지 않아 생기는 추가적인 정렬 과정을 없애고 성능을 최적화하기 위해 사용**한다.
  주로 **GROUP BY / DISTINCT / 집계 쿼리에 적용**된다.