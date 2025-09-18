# 리뷰 작성하는 쿼리,  * 사진의 경우는 일단 배제

```sql
    INSERT INTO review (member_id, store_id, body, score, created_at)
    VALUES ({사용자id}, {가게id}, {리뷰 내용}, {별점});
```

# 마이 페이지 화면 쿼리

```sql
    SELECT
    	member.nickname,
    	member.email,
    	CASE
    		WHEN member.status='verified' THEN member.phone
    		ELSE '미인증' 
    	END AS phone,
    	member.point
    FROM member
    WHERE member.id={사용자id};
```

# 내가 진행중, 진행 완료한 미션 모아서 보는 쿼리(페이징 포함)

```sql
    SELECT 
    	member_mission.id, 
    	member_mission.status, 
    	mission.reward,
    	mission.mission_spec,
    	store.name
    FROM member_mission
      JOIN mission ON mission.id=member_mission.id
      JOIN store ON store.id=mission.store_id
    WHERE member_mission.member_id={사용자id}
    	AND member_mission.status IN {‘진행중’, '진행 완료'}
    LIMIT {페이지 당 갯수} OFFSET {건너 뛸 갯수};
```

# 홈 화면 쿼리  (현재 선택 된 지역에서 도전이 가능한 미션 목록, 페이징 포함)

```sql
    SELECT
    	mission.id,
    	mission.reward,
    	mission.deadline,
    	mission.mission_spec,
    	store.id,
    	store.name,
    	region.id,
    	region.name
    FROM mission
      JOIN store ON store.id=mission.store_id
      JOIN region ON region.id=store.region_id
      LEFT JOIN member_mission ON member_mission.mission_id = mission.id
                              AND member_mission.member_id = {사용자id}
      ## mission 테이블의 모든 행을 기준으로 member_mission 테이블 join                       
    WHERE region.id={선택된지역id}
    	AND member_mission.id IS NULL ## 사용자가 아직 도전하지 않은 미션만 출력
    	AND mission.deadline >= NOW() ## 마감 기한이 아직 끝나지 않은 미션만
    ORDER BY mission.deadline DESC, mission.id ASC
    LIMIT {페이지  갯수} OFFSET {건너 뛸 갯수};
```