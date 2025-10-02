# 홈 화면
### 💡 **API Endpoint**

---
**GET** /api/home


### 💡 **Request Body**

---
GET 요청이므로 필요 X


### 💡 **Request Header**

---
Authorization : accessToken (String)


### 💡 **query String**

---
GET /api/home?status=NOT_STARTED&page=0&size=10&sort=deadline,ASC

regionId도 추가하면 좋을 듯?

### 💡 **Path variable**

---
X

# 마이 페이지

### 💡 **API Endpoint**

---
**GET** /api/users/mypage



### 💡 **Request Body**

---
GET 요청이므로 body 사용 X


### 💡 **Reponse Body**

---

```json
{
	"name": "박예진",
	"email": "dlapdlf@naver.com"
	...
}
```

### 💡 **Request Header**

---
Authorization : accessToken (String)


### 💡 **query String**

---
X

### 💡 **Path variable**

---
X

# 리뷰 작성
### 💡 **API Endpoint**

---
**POST** /api/stores/{storeId}/reviews


### 💡 **Request Body**

---

```json
{
	"missionId": 1
	"score": 5,
	"comment": "음식이 정말 맛있어요!",
	"reviewImage": [
		"URL",
		"URL"
		]
}
```

"storeId”는 path에서 받으니 body에서 필요X

missionId 추가 이유 : mission 탭에서 진행 완료 토글로 들어가면 진행 완료된 미션들이 뜨면서 거기에 리뷰 남기기 기능이 있으므로 해당 미션에 대한 리뷰를 남기는 것이니

### 💡 **Request Header**

---
Authorization : accessToken (String)
Content-Type: application/json


### 💡 **query String**

---
POST 생성이므로 필요 X

### 💡 **Path variable**

---
POST /api/stores/**{storeId}**/reviews

# 미션 목록 조회(진행중, 진행 완료)
## 진행 중
### 💡 **API Endpoint**

---
**GET** /api/missions/


### 💡 **Request Body**

---
GET 요청이므로 body 사용 X


### 💡 **Request Header**

---
Authorization : accessToken (String)


### 💡 **query String**

---
GET /api/missions?**status=IN_PROGRESS**&page=0&size=10&sort=updatedAt,DESC


### 💡 **Path variable**

---
특정 미션 조회가 아닌 목록 조회이므로 필요 X

## 진행 완료
### 💡 **API Endpoint**

---
**GET** /api/missions/

### 💡 **Request Body**

---
GET 요청이므로 body 사용 X


### 💡 **Request Header**

---
Authorization : accessToken (String)

### 💡 **query String**

---
GET /api/my/missions?**status=COMPLETED**&page=0&size=10&sort=updatedAt,DESC


### 💡 **Path variable**

---
특정 미션 조회가 아닌 목록 조회이므로 필요 X

 
# 미션 성공 누르기
### 💡 **API Endpoint**

---
**POST** /api/missions/{missionId}/completion


***POST** vs **PATCH***

단순한 상태 변경 작업이라면? POST

임의의 상태 편집이라면? PATCH. PATCH가 아예 안 되는 것은 아님. BUT 의미가 모호해질 수 있으므로 권장 X.

### 💡 **Request Body**

---
상태 변경 API이므로 Response Body만 필요

### 💡 **Reponse Body**

---

```json
{
	"missionId": 1,
	"storeId": 1,
	"isMissionCompleted": true,
	"missionPoint": 500 
}
```

isMissionCompleted → boolean으로 ERD 설계

IF enum이라면? “status”: “NOT_STARTED”, “IN_PROGRESS”, “COMPLETED”

### 💡 **Request Header**

---
Authorization : accessToken (String)

### 💡 **query String**

---
해당하는 미션은 {missionId}로 지정하므로 필요 X

### 💡 **Path variable**

---
POST /api/missions/**{missionId}**/completion


# 회원 가입 하기
### 💡 **API Endpoint**

---
**POST** /api/auth/signup


### 💡 **Request Body**

---

```json
{
	"name": "박예진",
	"gender": "FEMALE",
	"birth": "2002-06-18",
	"address": "서울특별시 노원구 광운로 20",
	"foodCategories": ["KOREAN", "JAPANESE", "CHINESE"],
	"terms": [
		{ "type": "AGE_OVER_14", "isAgreed" : true },
		{ "type": "SERVICE", "isAgreed" : true },
		{ "type": "PRIVACY", "isAgreed" : true },
		{ "type": "LOCATION", "isAgreed" : true },
		{ "type": "MARKETING", "isAgreed" : true },
		]
}
```

### 💡 **Request Header**

---
Content-Type: application/json


### 💡 **query String**

---
X

### 💡 **Path variable**

---
X