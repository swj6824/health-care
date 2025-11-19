# 🏥 Health Care API Guide

AI 기반 운동·식단·요약·목표·피드백 기능을 제공하는 REST API입니다.  
각 섹션은 **간단 설명 + 표 요약 + 주요 액션 엔드포인트 상세 블록**으로 구성되어 있습니다.

모든 인증 필요 API는 헤더에 다음 값을 포함합니다.

```http
Authorization: Bearer <access_token>
Content-Type: application/json
Accept: application/json
```

---

## 0. 기본 도메인

| 환경 | 기본 도메인 |
|------|-------------|
| 로컬 | `http://localhost:8000/api/` |
| 프로덕션 | 배포 도메인(`/api/` 기준) |

---

## 1. AUTH — 인증 & 토큰

사용자 회원가입과 JWT 발급/검증을 담당합니다.

| 기능 | 메서드 | 엔드포인트 | 성공 코드 | 실패 코드 | 비고 |
|------|--------|------------|-----------|-----------|------|
| 회원가입 | POST | `/auth/register/` | 201 | 400 | 비로그인 허용, 토큰 즉시 발급 |
| JWT 로그인 | POST | `/auth/token/` | 200 | 401 | 아이디/비밀번호 로그인 |
| 토큰 재발급 | POST | `/auth/token/refresh/` | 200 | 401 | refresh 토큰 필요 |
| 토큰 검증 | POST | `/auth/token/verify/` | 200 | 401 | JWT 유효성 검사 |
| 로그아웃 | POST | `/logout/` | 302 | - | 세션 로그아웃 (템플릿용) |

---

## 2. USERS — 사용자 관리

관리자용 사용자 목록/생성 + 일반 유저의 본인 정보 조회/수정을 제공합니다.

| 기능 | 메서드 | 엔드포인트 | 성공 코드 | 실패 코드 | 비고 |
|------|--------|------------|-----------|-----------|------|
| 사용자 목록 | GET | `/api/users/` | 200 | 403 | 관리자 전용 |
| 사용자 생성 | POST | `/api/users/` | 201 | 400 | 관리자 전용 |
| 사용자 상세 | GET | `/api/users/{id}/` | 200 | 404 | |
| 사용자 수정 | PATCH | `/api/users/{id}/` | 200 | 400/404 | |
| 사용자 삭제 | DELETE | `/api/users/{id}/` | 204 | 404 | 관리자 전용 |

---

## 3. TASKS — 할 일(Task) & 일일 Todo

날짜별 운동/할 일(Task)을 관리합니다. 완료 토글은 멱등하게 동작합니다.

| 기능 | 메서드 | 엔드포인트 | 성공 코드 | 실패 코드 | 비고 |
|------|--------|------------|-----------|-----------|------|
| Task 목록 | GET | `/api/tasks/` | 200 | 401 | `?date=YYYY-MM-DD` 필터 |
| Task 생성 | POST | `/api/tasks/` | 201 | 400 | JWT 필요 |
| Task 상세 | GET | `/api/tasks/{id}/` | 200 | 404 | 소유자만 |
| Task 수정 | PATCH | `/api/tasks/{id}/` | 200 | 400/404 | 제목/완료 상태 변경 |
| Task 삭제 | DELETE | `/api/tasks/{id}/` | 204 | 404 | 소유자만 |
| 완료 토글 | POST | `/api/tasks/{id}/toggle-complete/` | 200 | 404 | 멱등 토글 액션 |

**완료 토글 예시**

`POST /api/tasks/502/toggle-complete/`

- 설명: `completed` 값을 `true/false`로 토글  
- 응답 예시:
  ```json
  { "id": 502, "completed": true }
  ```

---

## 4. WORKOUT — 운동 종목 & Workout Plan

운동 종목(Exercises)과 날짜별 Workout Plan을 관리합니다.

### 4-1. 운동 종목 (Exercises)

| 기능 | 메서드 | 엔드포인트 | 성공 코드 | 실패 코드 | 비고 |
|------|--------|------------|-----------|-----------|------|
| 운동 목록 | GET | `/api/exercises/` | 200 | 401 | |
| 운동 생성 | POST | `/api/exercises/` | 201 | 400 | 관리자 전용 |
| 운동 상세 | GET | `/api/exercises/{id}/` | 200 | 404 | |
| 운동 수정 | PATCH | `/api/exercises/{id}/` | 200 | 400/404 | 관리자 전용 |
| 운동 삭제 | DELETE | `/api/exercises/{id}/` | 204 | 404 | 관리자 전용 |
| 기본 운동 로드 | GET | `/api/fixtures/exercises/` | 200 | 403/404 | 개발/관리용 시드 |

### 4-2. Workout Plan

| 기능 | 메서드 | 엔드포인트 | 성공 코드 | 실패 코드 | 비고 |
|------|--------|------------|-----------|-----------|------|
| 계획 목록 | GET | `/api/workoutplans/` | 200 | 401 | |
| 계획 생성 | POST | `/api/workoutplans/` | 201 | 400 | |
| 계획 상세 | GET | `/api/workoutplans/{id}/` | 200 | 404 | |
| 계획 수정 | PATCH | `/api/workoutplans/{id}/` | 200 | 400/404 | |
| 계획 삭제 | DELETE | `/api/workoutplans/{id}/` | 204 | 404 | |
| 일일 요약 | GET | `/api/workoutplans/summary/` | 200 | 400/401 | `?date=YYYY-MM-DD` |
| 운동 추천 | GET | `/api/recommendations/` | 200 | 401 | `?date=YYYY-MM-DD` |
| 오늘 인사이트 | GET | `/api/insights/today/` | 200 | 401 | |

**일일 Workout Summary**

`GET /api/workoutplans/summary/?date=2025-11-11`

- 설명: 특정 날짜의 Workout Plan 요약(완료/총 개수 등)을 반환  
- 응답 예시:
  ```json
  {
    "date": "2025-11-11",
    "tasks_completed": 5,
    "tasks_total": 8
  }
  ```

**운동 추천**

`GET /api/recommendations/?date=2025-11-11`

- 설명: 최근 활동/목표를 기반으로 AI 추천 운동 목록을 반환  
- 응답 예시:
  ```json
  [
    { "title": "상체 루틴", "reason": "최근 하체 위주로 진행했습니다." }
  ]
  ```

---

## 5. INTAKES — 식단 & 영양 로그

식품(Foods), 식사(Meals), 식사 항목(MealItems), 일일 영양 집계(NutritionLogs)를 관리합니다.

### 5-1. 식품 / 식사 / 식사 항목

| 기능 | 메서드 | 엔드포인트 | 성공 코드 | 실패 코드 | 비고 |
|------|--------|------------|-----------|-----------|------|
| 식품 목록 | GET | `/api/foods/` | 200 | 401 | `?q=` 검색 |
| 식사 목록 | GET | `/api/meals/` | 200 | 401 | `?log_date=YYYY-MM-DD` |
| 식사 생성 | POST | `/api/meals/` | 201 | 400 | |
| 식사 상세 | GET | `/api/meals/{id}/` | 200 | 404 | |
| 식사 수정 | PATCH | `/api/meals/{id}/` | 200 | 404 | |
| 식사 삭제 | DELETE | `/api/meals/{id}/` | 204 | 404 | |
| 날짜별 식사 | GET | `/api/meals/by-date/` | 200 | 400 | `?log_date=YYYY-MM-DD` |
| 식사 항목 생성 | POST | `/api/mealitems/` | 201 | 400 | |
| 식사 항목 수정 | PATCH | `/api/mealitems/{id}/` | 200 | 404 | |
| 식사 항목 삭제 | DELETE | `/api/mealitems/{id}/` | 204 | 404 | |

### 5-2. 영양 로그 (NutritionLogs)

| 기능 | 메서드 | 엔드포인트 | 성공 코드 | 실패 코드 | 비고 |
|------|--------|------------|-----------|-----------|------|
| 영양 로그 목록 | GET | `/api/nutritionlogs/` | 200 | 401 | 별칭 `/api/intakes/` |
| 영양 로그 상세 | GET | `/api/nutritionlogs/{id}/` | 200 | 404 | |
| 로그 ensure | POST | `/api/nutritionlogs/ensure/` | 200/201 | 400 | `?log_date=` |
| 로그 재계산 | POST | `/api/nutritionlogs/{id}/recalc/` | 200 | 404 | |

**영양 로그 보장 (ensure)**  
`POST /api/nutritionlogs/ensure/`

- 쿼리: `?log_date=YYYY-MM-DD` (없으면 오늘)  
- 설명: 해당 날짜의 NutritionLog가 없으면 생성하고, 있으면 그대로 반환  
- 응답 예시:
  ```json
  { "id": 402, "date": "2025-11-12" }
  ```

**영양 로그 재계산 (recalc)**  
`POST /api/nutritionlogs/401/recalc/`

- 설명: 해당 로그에 연결된 Meal/MealItem을 기준으로 칼로리·탄단지 집계를 다시 계산  
- 응답 예시:
  ```json
  { "id": 401, "kcal_total": 1350.0 }
  ```

---

## 6. AI — 식단 이미지 분석 파이프라인

AI 모델(HuggingFace ViT)을 호출해 음식 이미지를 분석하고, 식약처 CSV와 매칭해 영양 정보를 계산합니다.

| 기능 | 메서드 | 엔드포인트 | 성공 코드 | 실패 코드 | 비고 |
|------|--------|------------|-----------|-----------|------|
| 식단 분석 프리뷰 | POST | `/api/ai/meal-analyze/` | 200 | 400/422 | Multipart 권장 |
| 분석 결과 저장 | POST | `/api/ai/meal-commit/` | 200 | 400 | JWT 필요 |
| AI 항목 삭제 | DELETE | `/api/ai/meal-entry/{item_id}/` | 204 | 404 | |

**AI 식단 분석 (프리뷰)**  
`POST /api/ai/meal-analyze/` (multipart/form-data)

- 필드  
  - `image`: 음식 이미지 파일  
  - 또는 `text`: 텍스트 설명  
- 설명:  
  - ViT 모델로 음식 라벨 + confidence 추출  
  - 식약처 CSV(14,584건)와 매칭하여 per 100g 기준 macros 계산  
- 응답 예시:
  ```json
  {
    "label_ko": "샐러드",
    "confidence": 0.87,
    "macros_total": {"calories": 220, "protein": 12, "carb": 20, "fat": 9},
    "save_payload": { "...": "..." }
  }
  ```

**AI 분석 결과 저장 (commit)**  
`POST /api/ai/meal-commit/` (application/json)

- 필수: `save_payload` (meal-analyze 응답에서 그대로 전달)  
- 설명: 프리뷰 결과를 실제 Meal/MealItem + NutritionLog 에 반영  
- 응답 예시:
  ```json
  {
    "created": true,
    "meal_id": 550,
    "item_id": 551
  }
  ```

---

## 7. SUMMARY — 오늘의 통합 요약

하루 단위로 운동/식단/목표 진행률을 모아서 요약합니다.

| 기능 | 메서드 | 엔드포인트 | 성공 코드 | 실패 코드 | 비고 |
|------|--------|------------|-----------|-----------|------|
| 오늘의 요약 | GET | `/api/today/summary/` | 200 | 401 | 메인 Dashboard 데이터 |
| (레거시) 오늘의 요약 | GET | `/api/today/` | 200 | 401 | 구버전 별칭 |

**오늘의 요약 예시**

`GET /api/today/summary/`

```json
{
  "tasks_completed": 5,
  "kcal": 1200,
  "tips": ["수분 섭취를 늘려보세요"],
  "streak_days": 7
}
```

---

## 8. GOALS — 장기 목표 & 일일 목표

체중/운동/습관 등 장기 목표와 데일리 체크(일일 목표)를 관리합니다.

### 8-1. Goals

| 기능 | 메서드 | 엔드포인트 | 비고 |
|------|--------|------------|------|
| 목표 목록 | GET | `/api/goals/` | |
| 목표 생성 | POST | `/api/goals/` | |
| 목표 상세 | GET | `/api/goals/{id}/` | |
| 목표 수정 | PATCH | `/api/goals/{id}/` | |
| 목표 삭제 | DELETE | `/api/goals/{id}/` | |

### 8-2. DailyGoals

| 기능 | 메서드 | 엔드포인트 | 비고 |
|------|--------|------------|------|
| 일일 목표 목록 | GET | `/api/dailygoals/` | |
| 일일 목표 생성 | POST | `/api/dailygoals/` | |
| 일일 목표 수정 | PATCH | `/api/dailygoals/{id}/` | |
| 일일 목표 삭제 | DELETE | `/api/dailygoals/{id}/` | |

### 8-3. GoalProgress

| 기능 | 메서드 | 엔드포인트 |
|------|--------|------------|
| 목표 진행률 목록 | GET | `/api/goalprogress/` |
| 특정 목표 진행률 | GET | `/api/goalprogress/{id}/` |

---

## 9. FEEDBACKS — 피드백 / 데일리 리포트 / 업적

사용자 경험과 하루 컨디션을 기록합니다.

### 9-1. Feedbacks

| 기능 | 메서드 | 엔드포인트 |
|------|--------|------------|
| 피드백 목록 | GET | `/api/feedbacks/` |
| 피드백 생성 | POST | `/api/feedbacks/` |
| 피드백 상세 | GET | `/api/feedbacks/{id}/` |
| 피드백 수정 | PATCH | `/api/feedbacks/{id}/` |
| 피드백 삭제 | DELETE | `/api/feedbacks/{id}/` |

### 9-2. DailyReports

| 기능 | 메서드 | 엔드포인트 |
|------|--------|------------|
| 데일리 리포트 목록 | GET | `/api/dailyreports/` |
| 데일리 리포트 생성 | POST | `/api/dailyreports/` |
| 리포트 상세/수정/삭제 | GET/PATCH/DELETE | `/api/dailyreports/{id}/` |

### 9-3. Achievements

| 기능 | 메서드 | 엔드포인트 |
|------|--------|------------|
| 업적 목록 | GET | `/api/achievements/` |
| 업적 생성 | POST | `/api/achievements/` |
| 업적 상세/수정/삭제 | GET/PATCH/DELETE | `/api/achievements/{id}/` |

---

## 10. SYSTEM — 헬스 체크 & 모니터링

k6 · Prometheus · Grafana · Alertmanager 구성과 연계되는 기본 헬스 엔드포인트입니다.

| 기능 | 메서드 | 엔드포인트 | 설명 |
|------|--------|------------|------|
| 서버 헬스 체크 | GET | `/healthz` | Nginx 프록시 레벨 헬스 |
| Prometheus 메트릭 | GET | `/metrics/` | Django / 앱 메트릭 노출 |

