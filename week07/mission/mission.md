# RestControllerAdvice의 장점, 그리고 없을 경우 어떤 점이 불편한지도 조사하여 **미션 기록란**에 기재하기
   
   ## RestControllerAdvice가 해주는 역할
   **@RestControllerAdvice**는 모든 REST 컨트롤러에서 발생하는 예외를 한 곳에서 받아서, 공통 형식의 에러 응답 + 로깅 + 후처리를 하는 전역 예외 처리기.
   
   ## RestControllerAdvice를 썼을 때 장점
   ### 예외 처리 코드 중복 제거
   **@RestControllerAdvice**가 없으면, 컨트롤러마다 try-catch 써야 함.
   반면에 있으면, 컨트롤러 코드가 깔끔해지고 예외는 전역 핸들러에서 한 번만 작성하면 됨.
   
   ### 에러 응답 형식을 “일관되게” 맞출 수 있음
   프론트/앱에서 쓰기 편하게, 예외 응답 포맷을 하나로 맞출 수 있음. **@RestControllerAdvice** 안에서만 이렇게 만들어주면, 모든 컨트롤러에서 예외가 나도 같은 JSON 모양으로 떨어짐.
   
   ### @Valid, Binding 오류들 한 방에 처리 가능 요청 
   DTO에 @Valid 걸어두면, validation 깨질 때 MethodArgumentNotValidException, BindException 같은 예외가 터질 때, 이걸 컨트롤러마다 직접 처리하면 복잡해짐.
   이때, **@RestControllerAdvice**를 사용하면 어디에서 @Valid 터져도 같은 형식의 에러 응답을 받을 수 있음.

   ## RestControllerAdvice가 없으면 불편한 점
   ### 컨트롤러마다 try-catch 지옥 
   - 같은 예외(예: MemberNotFoundException)에 대해 여러 컨트롤러에서 똑같이 try-catch 반복
   - 예외 처리 로직이 여러 파일에 흩어짐
   
   → 코드가 길어지고, 나중에 정책 바꾸고 싶을 때, 전부 찾아서 바꿔야 함 (실수로 몇 개 빼먹기 딱 좋음)

   ### 에러 응답 포맷이 제각각 될 가능성이 큼.
   프론트 입장에서는 "이 API는 에러 나면 뭘 기준으로 파싱해야 하지?” 라는 문제가 생김.
   
   ### @Valid 등 검증 에러를 곳곳에서 수동 처리해야 함 
   @Valid 붙인 DTO가 10개라면, 검증 실패 시 에러 응답 만드는 코드도 10군데 등장할 수 있음. 
   ㄹ
   RestControllerAdvice가 있으면:

   검증 실패 예외 한 번만 처리 → 전체에 적용

   없으면 → 컨트롤러마다 하나씩 만들어야 함

# 지금까지 진행하면서 작성한 API들 모두 응답 통일 처리하기 / 성공 메서드, 성공 Enum 제작하기
[GitHub]
https://github.com/ye-zin/umc9th/tree/Feat/Chapter7