# RestContollerAdvice
  ## **RestControllerAdvice란?**
  `@RestControllerAdvice`는 **Spring에서 전역적으로 예외 처리, 바인딩 오류 처리, 공통 로직 처리 등을 담당하는 어노테이션**.

  쉽게 말해, **컨트롤러에서 발생하는 예외들을 한 곳에서 처리해주는 전역 핸들러 클래스**를 만들 때 사용함.

  ### 특징
  - **전역(Global)** 예외 처리
  - 모든 `@RestController`에서 발생하는 오류 처리
  - `@ExceptionHandler`, `@InitBinder`, `@ModelAttribute`와 함께 사용됨
  - 반환 형태를 JSON으로 처리 (RestController 성격)

  ### 장점
  - 코드 중복 제거 : 컨트롤러마다 try-catch를 넣을 필요 없어짐.
  - 일관된 예외 응답 구조 제공 : 프론트나 앱에서 예외 응답을 받을 때 **항상 같은 포맷**으로 받을 수 있음.
  - 유지보수성 향상 : 예외 처리 로직이 흩어져 있지 않고 한 곳에서 관리됨.
  - REST API 오류 처리에 최적화 : `@ControllerAdvice` + `@ResponseBody` 기능이 합쳐진 버전이기 때문에 JSON 응답이 기본.

  ### 결론
  `RestControllerAdvice`는 **REST API 예외 처리를 전역적으로 관리하며, JSON으로 응답을 반환하는 컨트롤러 전용 공통 예외 핸들러**.
  
  **API 개발에서 필수 요소!**

# lombok
  ## Lombok이란?
  **반복되는 보일러플레이트(boilerplate) 코드들을 자동으로 생성해주는 Java 라이브러리.**

  즉,
  - getter/setter
  - 생성자
  - equals/hashCode
  - toString
  - Builder 패턴
  등을 자동으로 만들어줘서 코드가 훨씬 간결해짐.

  ## 장점
  - 보일러플레이트 코드 제거 : 클래스에 필드만 선언해도 → getter, setter, toString, constructor 를 자동 생성.
  - 가독성 및 유지보수성 향상 : 중복 코드가 사라져 클래스가 깔끔해짐.
  - Builder 패턴을 쉽게 사용 : 복잡한 객체 생성 시 `@Builder` 하나로 해결.
  - 생성자 자동 생성 : `@RequiredArgsConstructor`, `@AllArgsConstructor` 등으로 의존성 주입도 깔끔해짐.
    
  ## Lombok 주요 어노테이션 정리
  ### **1. @Getter / @Setter**
  필드 혹은 클래스 단위로 getter/setter 생성.
  ```java
    @Getter
    @Setter
    public class Member {
        private String name;
        private int age;
    }
  ```
---
  ### **2. @NoArgsConstructor / @AllArgsConstructor / @RequiredArgsConstructor**
  - **@NoArgsConstructor**
  파라미터 없는 기본 생성자 생성
  - **@AllArgsConstructor**
  모든 필드를 파라미터로 받는 생성자 생성
  - **@RequiredArgsConstructor**
  `final` 필드 혹은 `@NonNull` 필드만 포함한 생성자 생성 → **Spring DI에 매우 자주 사용됨**
  
  - 예시:
    ```java
    @RequiredArgsConstructor
    @Service
    public class MemberService {
        private final MemberRepository memberRepository; // 자동 생성자 주입
    }
    ```
    
---
  ### **3. @ToString**
  `toString()` 자동 생성
   ```java
    @ToString
    public class Member {
        private String name;
    }
   ```
---
  ### **4. @EqualsAndHashCode**
  equals()와 hashCode() 자동 생성
---
  ### **5. @Data**
  아래 어노테이션을 한 번에 적용
  - @Getter
  - @Setter
  - @ToString
  - @EqualsAndHashCode
  - @RequiredArgsConstructor
    
  ⚠️ **엔티티(Entity)에는 잘 안 씀!**
  이유 → setter가 자동 생성돼 객체 안정성 저해.
    
---
  ### **6. @Builder**
  아이템 생성 시 Builder 패턴을 자동으로 적용 → **너무 많이 쓰는 Lombok 기능**
  ```java
    @Getter
    @Builder
    public class Member {
        private Long id;
        private String name;
        private int age;
    }
  ```
  사용:
  ```java
    Member member = Member.builder()
            .id(1L)
            .name("서빈")
            .age(24)
            .build();
  ```
---
  ### **7. @Value**
  Immutable 객체 생성 (불변 객체) → 모든 필드가 `private final`
  → Getter만 생성 / Setter 없음
  DTO나 외부 응답 모델에서 자주 쓰임.
---
  ### **8. @Slf4j**
  로깅 자동 주입

  ```java
    @Slf4j
    @RestController
    public class TestController {
    
        @GetMapping("/test")
        public String test() {
            log.info("테스트 로그");
            return "ok";
        }
    }
  ```
    
  ## Lombok 주의사항
  ### 1. entity(@Entity)에서는 @Data 쓰지 말라
  - setter 남발 → JPA의 상태 변화 추적 어려움
  - equals, hashCode가 문제 유발할 수 있음
  
    **그러므로 엔티티는 @Getter + @NoArgsConstructor 정도만 사용!**
        
  ### 2. @Builder는 생성자나 필드가 바뀌면 빌더도 바뀌므로 주의
  특히 엔티티 빌더는 잘 안 쓰는 편.
 
  ### 3. IDE Plugin 필수
  IntelliJ 사용하는 경우
  **Settings → Plugins → Lombok 설치 + annotation processing 활성화**
    
  ## Lombok 자주 사용하는 패턴 (실전용)
  ### Service 계층
    
  ```java
    @Service
    @RequiredArgsConstructor
    public class MemberService {
        private final MemberRepository repository;
    }
  ```
    
  ### DTO
  ```java
    @Getter
    @Builder
    public class MemberDTO {
        private Long id;
        private String name;
    }
  ```
    
  ### Entity 권장 방식
  ```java
    @Entity
    @Getter
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public class Member {
    
        @Id @GeneratedValue
        private Long id;
    
        private String name;
    }
  ```

# dto 형식 public static VS record 비교하기
  ## A. **public static class DTO**
  일반적인 Java 클래스 방식
  ```java
    public class MemberDTO {
        private Long id;
        private String name;
    
        public MemberDTO(Long id, String name) {
            this.id = id;
            this.name = name;
        }
        // getter, setter, toString, equals 등 필요할 때 추가
    }
  ```
  또는 내부 static class 로 만드는 방식도 있음:
  ```java
    public class MemberResponse {
        @Getter
        @AllArgsConstructor
        public static class DetailDTO {
            private Long id;
            private String name;
        }
    }
  ```
  ---
  ## B. **record DTO**
  Java 16+에서 등장한 **불변(immutable) 데이터 전달용 타입**
  ```java
    public record MemberDTO(Long id, String name) { }
  ```
  - 필드 → 모두 `private final`
  - 생성자/equals/hashCode/toString 자동 생성
  - getter는 `id()`처럼 컴팩트함

  ## 비교 구조표

  | 비교 항목 | public static class | record |
      | --- | --- | --- |
  | **불변성(immutable)** | ❌ 기본적으로 불변 아님 (setter 가능) | ✅ 완전 불변 |
  | **boilerplate 코드** | 많음 (getter/setter/constructor 필요) | 매우 적음 |
  | **JPA Entity에서 사용** | DTO로 사용하면 문제 없음 | DTO로 사용 적합함 |
  | **Jackson JSON 역/직렬화** | Getter/Setter 필요(또는 생성자 필요) | 자동으로 잘됨 |
  | **상속 가능 여부** | 가능 | ❌ 불가능 |
  | **필드 검증/로직 추가** | 자유롭게 가능 | 가능하나 제약 많음 |
  | **Builder 사용** | 가능 (lombok @Builder) | ❌ 기본 builder 없음 |
  | **복잡한 구조 표현** | 용이 | 다중 DTO → 서브 record 가능 |
  | **Nullable 필드** | 가능 | 가능하나 setter 없음 → 교체 불가 |
  | **Java에서 권장 용도** | 상태 변화가 필요하면 class | 순수 데이터 전달은 record |

  ## 실전 사용 관점에서 차이

  ### A. 코드 양 차이

  **class**
  ```java
    @Getter
    @AllArgsConstructor
    public static class MemberDTO {
        private Long id;
        private String name;
    }
  ```

  **record**
  ```java
    public record MemberDTO(Long id, String name) {}
  ```

  **→ record는 압도적으로 짧고 가독성 좋음.**

  ## 왜 record가 더 인기일까?

  ### 1. DTO의 본질 때문에

  DTO의 목적은 **순수 데이터 전달(data transfer)**

  → setter 필요 없음

  → equals/hashCode 필요

  → 불변성이 안전함

  record는 **DTO 목적에 완전히 부합**함.

  ### 2. 반복 작업 최소화

  테이블/엔티티/DTO가 많아질수록 record는 관리가 매우 쉬움.

  ## 하지만 class가 필요한 경우도 있다

  ### 1. **Builder 패턴을 쓰고 싶을 때**

  record는 builder를 제공하지 않는다.

  예:
  ```java
    UserDTO.builder()
      .id(1L)
      .name("홍길동")
      .build();
  ```

  이런 방식 쓰고 싶으면 class 사용해야 함.
    
  ---

  ### 2. **필드가 많은 복잡한 DTO**

  record의 생성자 파라미터는 **순서 기반.** 필드 많으면 가독성 떨어짐.
    
  ---

  ### 3. **필드 일부만 선택적으로 변경해야 할 때**

  record는 불변이므로 전체를 다시 생성해야 함.

  class는 setter 또는 builder로 쉽다.
    
  ---

  ### 4. **Spring Validation (@Valid)을 Multipurpose하게 쓰고 싶을 때**

  record는 다 되긴 하는데 검증 로직 추가하는 게 class보다 불편한 경우 있음.

  ## 실전 기준 결론

  ### **1. API 응답/요청 DTO: record 추천**
  - 불변 데이터 전달에 최적화
  - 코드 간결
  - 직렬화/역직렬화 자연스러움
  - equals/hashCode 자동 생성

  → REST API + DTO 많을 때 **record가 장점 극대화**
    
  ---

  ### **2. Builder 필요하거나 필드가 많다면 public static class**
  - Builder 구조가 필요할 때
  - 복잡한 필드 조합
  - 선택적 필드나 optional 구조
  - 내부 로직 조금 필요할 때

  특히 **Service → Controller** 로 많은 필드 요청에 너무 적합.

  ---

  ## 같은 DTO를 두 가지 방식으로 비교 예시

  ### Record 방식
  ```java
    public record OrderDTO(
        Long id,
        String status,
        MemberDTO member,
        List<OrderItemDTO> items
    ) {}
  ```

  ### public static class 방식
  ```java
    @Getter
    @Builder
    public static class OrderDTO {
        private Long id;
        private String status;
        private MemberDTO member;
        private List<OrderItemDTO> items;
    }
  ```

  ### 어떤 게 더 좋은가?
  - **정해진 구조의 응답 DTO → record**
  - **빌더가 필요한 복잡 DTO → class**