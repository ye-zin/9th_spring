# java의 Exception 종류들
Java의 예외(Exception)는 크게 **3가지로 나뉨**.

- Java의 예외(Exception)는 크게 **3가지로 나뉨.**

  ## **Checked Exception (체크 예외 ─ 강제 처리 필요)**

    - 컴파일 시점에 반드시 **try-catch 처리하거나 throws 선언해야 함**
    - 처리하지 않으면 컴파일 에러 발생
    - 개발자에게 "**이 상황은 반드시 대비해야 한다**"라고 알려줌

  ### ✔ 대표 예시

  | 예외 | 설명 |
      | --- | --- |
  | **IOException** | 파일, 네트워크 등 I/O 처리 실패 |
  | **SQLException** | DB 관련 처리 오류 |
  | **FileNotFoundException** | 파일 못 찾음 |
  | **ClassNotFoundException** | 클래스를 JVM이 찾지 못함 |
  | **InterruptedException** | 스레드 sleep 중 인터럽트 발생 |
  | **ParseException** | 문자열 파싱 실패 |

  ## **nchecked Exception (언체크/런타임 예외)**

    - **RuntimeException**을 상속
    - 컴파일러가 강제 처리하지 않음
    - 즉, try-catch 없이도 컴파일 됨
    - 개발자의 실수(버그)로 발생하는 경우가 대부분

  ### ✔ 대표 예시

  | 예외 | 설명 |
      | --- | --- |
  | **NullPointerException (NPE)** | null 객체 참조 |
  | **IndexOutOfBoundsException** | 배열/리스트 범위 초과 |
  | **IllegalArgumentException** | 잘못된 매개변수 전달 |
  | **IllegalStateException** | 객체 상태가 메서드 호출에 적절하지 않을 때 |
  | **ArithmeticException** | 0으로 나누기 등 산술 오류 |
  | **NumberFormatException** | 숫자로 변환할 수 없는 문자열 입력 |
  | **ClassCastException** | 잘못된 타입 캐스팅 |
  | **ConcurrentModificationException** | 컬렉션 반복 도중에 변경 |

  ## **Error (에러 ─ 복구 불가 시스템 레벨 오류)**

    - 예외(Exception)와는 다른 개념
    - 애플리케이션 코드에서 처리 or 복구 **불가능**
    - 보통 JVM이나 시스템 자원이 부족할 때 발생

  ### ✔ 대표 예시

  | 에러 | 설명 |
      | --- | --- |
  | **OutOfMemoryError (OOM)** | 메모리 부족 |
  | **StackOverflowError** | 재귀 호출 등으로 스택 초과 |
  | **InternalError** | JVM 내부 오류 |
  | **UnknownError** | 정의되지 않은 심각한 오류 |

  ## 언제 Checked / Unchecked 로 설계할까?

  | 종류 | 목적 | 예시 |
      | --- | --- | --- |
  | **Checked Exception** | 개발자가 꼭 처리해야 하는 상황 | 파일, 네트워크, DB |
  | **Unchecked Exception** | 프로그래밍 실수를 나타냄 | NPE, IndexOutOfBounds |
  | **Error** | 시스템 레벨 오류 | 메모리 부족, 스택오버플로우 |

  # 한 줄 요약

    - **Checked** = “반드시 처리해라(컴파일러 강제)”
    - **Unchecked** = “개발자가 잘못한 거니 알아서 고쳐라”
    - **Error** = “애플리케이션에서 못 고친다. JVM이 죽었다

# @Vaild
## **Valid 정리**

`@Valid`는 **DTO의 필드를 검증(Validation)** 하기 위해 사용하는 어노테이션

JPA, Spring, Controller, Service 어디서든 적용 가능하고 JSR-303/349 Bean Validation(자바 표준) 기반으로 동작함.

즉, **컨트롤러에 들어오는 값이 조건에 맞는지 자동으로 검사**해 주고, 조건에 맞지 않으면 **자동으로 예외(400 Bad Request)** 발생

## 1. `@Valid`가 하는 일

`@Valid`는 요청 DTO에 붙어있는 validation 애너테이션(예: `@NotNull`, `@Email`, `@Size`)을 스캔해서

조건 만족 여부를 확인한 뒤:

- 조건 만족 → 정상적으로 메서드 실행
- 조건 불만족 → `MethodArgumentNotValidException` 발생

그리고 이 예외는 Spring이 400 응답으로 자동 반환하거나 `@RestControllerAdvice`에서 잡아서 커스텀 메시지로 처리할 수 있음.

## 2. 사용 예시

### DTO

```java
public record SignupRequest(
    @NotBlank(message = "이메일은 필수입니다.")
    @Email(message = "이메일 형식이 아닙니다.")
    String email,

    @NotBlank(message = "비밀번호는 필수입니다.")
    @Size(min = 8, message = "비밀번호는 최소 8자 이상이어야 합니다.")
    String password
) {}
```

### Controller

```java
@PostMapping("/signup")
public ResponseEntity<?> signup(@Valid @RequestBody SignupRequest request) {
    return ResponseEntity.ok("ok");
}
```

➡️ 요청 데이터가 규칙을 하나라도 어기면, Controller 로직이 실행되지 않고 즉시 400 반환!

## 3. `@Valid` vs `@Validated` 차이

| 항목 | @Valid | @Validated |
| --- | --- | --- |
| 제공 | javax.validation | Spring |
| 그룹 기능 | ❌ 없음 | ✔ 지원 |
| 일반적 사용 | DTO 검증 | 특정 상황에서 그룹 검증 필요할 때 |

대부분의 경우 `@Valid`만으로 충분함.

## 4. 어떤 필드에 어떤 검증을 쓸 수 있나?

| 애너테이션 | 설명 |
| --- | --- |
| `@NotNull` | null 금지 |
| `@NotBlank` | 빈 문자열 금지 (`"   "`도 불가) |
| `@NotEmpty` | length>0 라면 통과 |
| `@Email` | 이메일 형식 검증 |
| `@Size(min, max)` | 길이 제한 |
| `@Min / @Max` | 숫자 최소/최대 |
| `@Positive / @PositiveOrZero` | 양수/0 이상 |
| `@Pattern(regexp=...)` | 정규식 검증 |
| `@Past / @Future` | 날짜 검증 |

## 5. Spring + `@Valid`의 동작 흐름

1. 클라이언트가 요청 DTO 전달
2. Spring 이 `@Valid` 확인
3. DTO 필드에 붙은 validation 애너테이션 읽음
4. 조건 불만족 → `MethodArgumentNotValidException`
5. `@RestControllerAdvice` 로 전역 예외 처리 가능

## 6. 실무에서 자주 쓰는 패턴

### 계층 구조

Controller → Service → Repository

보통 **Controller에서 한 번 검증**하는 것이 원칙.

하지만,

- 외부 API에서 들어온 값
- 내부 서비스 호출
- Job/Batch

같이 컨트롤러를 안 거치는 입력이 있다면, **Service 레이어에서 @Validated 적용**하기도 함.

## 7. Spring Boot에서 자동 Validation 하기 위한 조건

build.gradle에 반드시 있어야 함:

```
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

## 한 줄 요약

**@Valid = DTO 입력값을 자동 검증해주고, 조건을 어기면 바로 400 에러를 던져주는 Bean Validation 트리거 역할**