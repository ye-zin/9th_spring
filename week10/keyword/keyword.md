# Spring Security
## 1. Spring Security란 무엇인가

**Spring Security**는 Spring 기반 애플리케이션에서 **보안(Security)** 을 담당하는 표준 프레임워크입니다.

핵심 목적은 다음 두 가지입니다.

1. **인증(Authentication)**

   → “이 사용자가 누구인가?”

2. **인가(Authorization)**

   → “이 사용자가 무엇을 할 수 있는가?”


즉,

> 로그인 처리 + 권한 제어를 체계적으로 담당하는 보안 인프라
>

라고 이해하면 됩니다.

---

## 2. Spring Security가 해결하는 문제

Spring Security를 사용하지 않으면 개발자가 직접 처리해야 하는 것들:

- 로그인 로직 구현
- 비밀번호 암호화
- 요청마다 사용자 확인
- 권한별 접근 제한
- 세션 / 토큰 검증
- CSRF, XSS, 세션 고정 공격 방어

Spring Security는 이러한 보안 문제를 **검증된 구조와 흐름**으로 제공합니다.

---

## 3. Spring Security의 핵심 특징

### 1) 필터(Filter) 기반 보안 구조

Spring Security는 **서블릿 필터 체인(Filter Chain)** 위에서 동작합니다.

```
Client 요청
 →SecurityFilter Chain
   → 인증 필터
   → 권한 필터
   → 예외 처리 필터
 → Controller
```

즉,

- Controller에 도달하기 **전에**
- 사용자가 인증되었는지
- 해당 요청에 접근할 권한이 있는지를 검사합니다.

---

### 2) 보안은 “선 차단, 후 처리”

Spring Security의 기본 철학:

> 허용하지 않은 요청은 모두 차단
>

그래서 아무 설정을 하지 않으면:

- 모든 요청이 인증 필요
- 로그인 페이지가 자동 생성됨

---

### 3) 확장 가능한 구조

Spring Security는 다음 방식을 모두 지원합니다.

- Form Login (세션 기반)
- HTTP Basic
- OAuth2 Login
- JWT 기반 토큰 인증
- Custom Authentication Provider
- Custom Filter

→ **실무에서 사용하는 JWT 인증 구조도 Spring Security 위에서 구현**

---

## 4. Spring Security의 주요 구성 요소 (개념)

지금은 “이름만 익숙해지기” 단계로 보세요.

| 구성 요소 | 역할 |
| --- | --- |
| `SecurityFilterChain` | 보안 규칙의 시작점 |
| `Authentication` | 인증된 사용자 정보 객체 |
| `AuthenticationManager` | 인증 처리 총괄 |
| `UserDetailsService` | 사용자 조회 |
| `UserDetails` | 사용자 정보 추상화 |
| `PasswordEncoder` | 비밀번호 암호화 |
| `SecurityContext` | 현재 사용자 정보 저장 |
| `GrantedAuthority` | 사용자 권한(Role) |

이 중 **Authentication / Authorization / Token** 개념이 다음 키워드로 이어집니다.

---

## 5. Spring Security의 기본 인증 흐름

### ① 사용자가 요청을 보낸다

```
POST /login
```

### ② Security Filter가 요청을 가로챈다

### ③ 인증 정보(ID, PW)를 Authentication 객체로 만든다

### ④ AuthenticationManager가 인증을 시도한다

- UserDetailsService로 사용자 조회
- PasswordEncoder로 비밀번호 검증

### ⑤ 인증 성공 시

- `Authentication` 객체 생성
- `SecurityContext`에 저장

### ⑥ 이후 요청에서는

- SecurityContext를 기반으로 접근 허용/차단

---

## 6. “Spring Security = 로그인 라이브러리”라는 오해

❌ 잘못된 이해

> Spring Security는 로그인만 처리한다
>

⭕ 정확한 이해

> Spring Security는 애플리케이션 전반의 접근 통제 시스템
>
- 로그인은 인증의 한 형태일 뿐
- 핵심은 **누가 / 무엇을 / 어디까지** 할 수 있는가

---

## 7. 왜 Spring Security를 배워야 하는가

당신이 하고 있는 것들과 직접 연결됩니다.

- JWT 로그인 구현
- Swagger 인증 문제 해결
- `/auth/login`, `/auth/signup` 보안 설정
- ADMIN / USER 권한 분리
- Refresh Token 재발급 구조

→ 이 모든 것이 **Spring Security 위에서 돌아감**

# 인증(Authentication)과 인가(Authorization)
## 1. 한 문장으로 구분하면

| 구분 | 핵심 질문 |
| --- | --- |
| **인증 (Authentication)** | “너는 **누구냐**?” |
| **인가 (Authorization)** | “너는 **무엇을 할 수 있냐**?” |

> 인증은 신원 확인,
>
>
> 인가는 **권한 확인**입니다.
>

---

## 2. 인증(Authentication)이란

### 1) 정의

**인증(Authentication)** 은 사용자가 주장하는 신원(ID, 토큰 등)이 **정말 맞는지 검증하는 과정**입니다.

즉,

- 로그인
- 토큰 검증
- OAuth 인증

모두 **인증**에 해당합니다.

---

### 2) 인증에서 다루는 정보

- 아이디 / 비밀번호
- JWT 토큰
- OAuth 인증 정보
- 인증서

👉 핵심은 **“이 사용자가 누구인지 확정하는 것”**

---

### 3) Spring Security에서 인증은 어디서 일어나는가

인증은 **Filter 단계**에서 이루어집니다.

대표 예시:

- `UsernamePasswordAuthenticationFilter`
- `JwtAuthenticationFilter` (커스텀)

인증이 성공하면:

- `Authentication` 객체 생성
- `SecurityContext`에 저장

---

### 4) 인증 성공/실패의 기준

| 결과 | 의미 |
| --- | --- |
| 성공 | 사용자가 **확인됨** |
| 실패 | 사용자가 **확인되지 않음** |

실패 시:

- 401 Unauthorized 반환
- 또는 로그인 페이지로 리다이렉트

---

## 3. 인가(Authorization)이란

### 1) 정의

**인가(Authorization)** 는

이미 인증된 사용자가 **특정 자원에 접근할 권한이 있는지**를 판단하는 과정입니다.

즉,

> “로그인은 했는데, 이 기능까지 써도 되냐?”
>

---

### 2) 인가에서 다루는 정보

- 역할(Role): `ROLE_USER`, `ROLE_ADMIN`
- 권한(Authority): `READ_POST`, `DELETE_USER`

👉 **인증된 사용자 + 권한 정보**를 기준으로 판단

---

### 3) Spring Security에서 인가는 언제 일어나는가

인가 역시 **Filter 단계**에서 수행됩니다.

인증 이후 흐름:

```
요청
 → 인증(Authentication)
 → 인가(Authorization)
 → Controller
```

---

### 4) 인가 실패 시

| 상황 | 결과 |
| --- | --- |
| 인증 O, 인가 X | 403 Forbidden |
| 인증 X | 401 Unauthorized |

---

## 4. 인증 vs 인가 — 비교 표 (핵심 정리)

| 항목 | 인증(Authentication) | 인가(Authorization) |
| --- | --- | --- |
| 목적 | 신원 확인 | 접근 권한 확인 |
| 기준 | ID, PW, Token | Role, Authority |
| 시점 | 가장 먼저 | 인증 이후 |
| 실패 코드 | 401 | 403 |
| 대상 | 사용자 | 자원(Resource) |

---

## 5. Spring Security 내부 객체로 보면

### 인증 결과물: `Authentication`

```java
Authentication authentication;
```

이 안에 들어있는 것:

- Principal (사용자 정보)
- Credentials (보통 인증 후 제거)
- Authorities (권한 목록)

👉 **인가 판단은 이 Authorities를 기준으로 수행**

---

## 6. 예시로 정리

### 예시 1: 로그인 API

```
POST /auth/login
```

- 인증: ID/PW 확인
- 인가: 없음 (permitAll)

---

### 예시 2: 마이페이지 조회

```
GET /users/me
```

- 인증: 로그인 여부 확인
- 인가: 로그인한 사용자만 가능

---

### 예시 3: 관리자 기능

```
DELETE /admin/users/{id}
```

- 인증: 로그인 필수
- 인가: ROLE_ADMIN만 허용

---

## 7. Spring Security 설정 예시 (개념용)

```java
http
  .authorizeHttpRequests(auth -> auth
      .requestMatchers("/auth/**").permitAll()// 인증 X
      .requestMatchers("/admin/**").hasRole("ADMIN")// 인증 O + 인가
      .anyRequest().authenticated()// 인증만 필요
  );
```

- `authenticated()` → 인증만 확인
- `hasRole()` / `hasAuthority()` → 인가 확인

---

## 8. 아주 중요한 오해 정리

❌ “로그인 했으니까 다 접근 가능”

⭕ **로그인 = 인증만 통과**

→ 이후 모든 요청은 **인가 검증 대상**

# 세션과 토큰
## 1. 세션과 토큰이 왜 필요한가

인증(Authentication)은 **한 번** 하고 끝나지 않습니다.

> 로그인 이후 요청마다
>
>
> “이 사용자가 누구인지”를 **계속 증명**해야 합니다.
>

이를 위한 **인증 상태 유지 방식**이:

- **세션(Session)**
- **토큰(Token)**

---

## 2. 세션(Session) 방식

### 2-1. 세션이란

**세션(Session)** 은

서버가 사용자별 **로그인 상태를 직접 저장**하는 방식입니다.

- 인증 정보: **서버 메모리 / 세션 저장소**
- 클라이언트는 **세션 ID만 보관**

---

### 2-2. 세션 기반 인증 흐름

```
1. 로그인 요청 (ID / PW)
2. 서버 인증 성공
3. 서버가 Session 생성
4. Session ID 발급
5. Session ID를 Cookie에 저장
6. 이후 요청마다 Cookie 자동 전송
7. 서버는 Session ID로 사용자 조회
```

---

### 2-3. 세션 방식의 특징

| 항목 | 설명 |
| --- | --- |
| 상태 관리 | 서버가 상태(State)를 가짐 |
| 저장 위치 | 서버 |
| 인증 정보 | Session ID |
| 클라이언트 | Cookie 자동 전송 |

---

### 2-4. 장점

- 구현이 단순
- 보안 개념 직관적
- 서버에서 즉시 로그아웃 가능

---

### 2-5. 단점

- 서버 메모리 부담
- 서버 확장(Scale-out) 어려움
- 모바일 / API 서버에 부적합
- 서버 장애 시 세션 손실 가능

---

## 3. 토큰(Token) 방식

### 3-1. 토큰이란

**토큰(Token)**은 서버가 인증 결과를 **암호화된 문자열**로 발급하고, 클라이언트가 이를 보관하며 요청마다 전달하는 방식입니다.

- 서버는 로그인 상태를 저장하지 않음
- 토큰 자체가 “신분증” 역할

---

### 3-2. 토큰 기반 인증 흐름 (JWT 기준)

```
1. 로그인 요청
2. 서버 인증 성공
3. Access Token 발급
4. 클라이언트에 전달
5. 이후 요청마다 Authorization 헤더에 토큰 포함
6. 서버는 토큰 검증 후 사용자 식별
```

---

### 3-3. 토큰 방식의 특징

| 항목 | 설명 |
| --- | --- |
| 상태 관리 | Stateless |
| 저장 위치 | 클라이언트 |
| 인증 정보 | Token 자체 |
| 서버 | 검증만 수행 |

---

### 3-4. 장점

- 서버 확장에 유리
- 모바일 / SPA / API에 적합
- 마이크로서비스 구조에 유리

---

### 3-5. 단점

- 토큰 탈취 시 위험
- 즉시 로그아웃 어려움
- 만료 관리 필요

---

## 4. 세션 vs 토큰 비교

| 항목 | 세션(Session) | 토큰(Token) |
| --- | --- | --- |
| 상태 저장 | 서버 | 클라이언트 |
| 서버 부하 | 큼 | 적음 |
| 확장성 | 낮음 | 높음 |
| 보안 통제 | 서버 중심 | 토큰 관리 필요 |
| 로그아웃 | 즉시 가능 | 만료까지 유효 |
| 사용 환경 | 웹 중심 | API / 모바일 |

---

## 5. Spring Security 관점에서의 차이

### 세션 기반

- `SecurityContext` → `HttpSession` 저장
- 기본 Form Login 구조

### 토큰 기반

- `SecurityContext` → 요청마다 재생성
- JWT Filter에서 직접 설정

---

## 6. 왜 요즘은 토큰 방식을 쓰는가

현대 서비스 특징:

- 프론트엔드와 백엔드 분리
- 모바일 앱
- 다중 서버
- 클라우드 환경

👉 **세션은 한계**, 토큰은 구조적으로 적합

# 액세스 토큰(Access Token)과 리프레시 토큰(Refresh Token)
## 1. 왜 토큰을 두 개로 나누는가

한 줄 요약:

> Access Token은 “짧게 쓰는 출입증”,
Refresh Token은 “출입증을 다시 발급받는 신분증”
>

토큰을 하나만 쓰면:

- 유효기간을 짧게 → 계속 로그인 필요
- 유효기간을 길게 → 탈취 시 치명적

👉 **보안과 사용성의 균형**을 위해 분리합니다.

---

## 2. Access Token (액세스 토큰)

### 2-1. 정의

**Access Token**은

API 요청 시 **사용자 인증·인가를 증명하는 토큰**입니다.

- 매 요청마다 사용
- 만료 시간 짧음

---

### 2-2. 주요 특징

| 항목 | 설명 |
| --- | --- |
| 사용 목적 | API 접근 |
| 유효기간 | 짧음 (수분 ~ 수십 분) |
| 저장 위치 | 클라이언트 메모리 / 로컬 |
| 노출 위험 | 상대적으로 높음 |

---

### 2-3. 요청 예시

```
GET /posts
Authorization: Bearer {accessToken}
```

서버는:

- 서명 검증
- 만료 확인
- 권한(Role) 확인

---

### 2-4. 만료되면?

- 401 Unauthorized
- 새 Access Token 필요

---

## 3. Refresh Token (리프레시 토큰)

### 3-1. 정의

**Refresh Token**은 **새 Access Token을 재발급받기 위한 토큰**입니다.

- API 접근에는 사용하지 않음
- 서버가 비교적 엄격히 관리

---

### 3-2. 주요 특징

| 항목 | 설명 |
| --- | --- |
| 사용 목적 | Access Token 재발급 |
| 유효기간 | 김 (수일 ~ 수주) |
| 전송 빈도 | 매우 낮음 |
| 노출 위험 | 낮음 (설계에 따라) |

---

### 3-3. 재발급 흐름

```
1. Access Token 만료
2. 클라이언트 → Refresh Token 전송
3. 서버가 Refresh Token 검증
4. 새로운 Access Token 발급
5. (선택) Refresh Token 재발급
```

---

## 4. 전체 인증 흐름 (로그인 → 사용 → 재발급)

```
[로그인]
ID/PW → 인증 성공
 →Access Token 발급
 →Refresh Token 발급

[API 요청]
Access Token 사용

[만료 시]
Refresh Token →Access Token 재발급
```

---

## 5. Access vs Refresh Token 비교 표

| 항목 | Access Token | Refresh Token |
| --- | --- | --- |
| 사용 위치 | 모든 API 요청 | 재발급 요청 |
| 유효기간 | 짧음 | 김 |
| 서버 저장 | 안 함 | 보통 저장 |
| 탈취 위험 | 높음 | 상대적으로 낮음 |
| 역할 | 인증·인가 | 연장 |

---

## 6. Refresh Token은 왜 서버에 저장하는가

Access Token은:

- Stateless
- 서버 저장 ❌

Refresh Token은:

- **탈취 대응 필요**
- 강제 로그아웃
- 중복 로그인 제어

그래서 보통:

- DB
- Redis

에 저장합니다.

---

## 7. 만약 Refresh Token이 탈취되면?

대응 전략:

- 서버 저장된 토큰과 비교
- 불일치 시 전체 토큰 무효화
- 강제 재로그인

---

## 8. Spring Security에서의 위치

- Access Token
    - `JwtAuthenticationFilter`
    - 매 요청마다 검증
- Refresh Token
    - `/auth/refresh` 전용 API
    - Filter 또는 Controller 처리

---

## 11. 전체 키워드 연결 요약

```
SpringSecurity
 → 인증(Authentication)
 → 인가(Authorization)
 → 토큰(Token)
   → Access Token
   → Refresh Token
```