# Health Care — AI 기반 개인 맞춤형 운동·식단 관리 플랫폼
AI 식단 분석, 운동 관리, 식단 기록, Summary, 목표 관리 기능을 하나로 통합한 풀스택 헬스케어 서비스입니다. Django REST Framework + PostgreSQL + Redis + Docker Compose 기반으로 개발되었으며, AI 분석과 실시간 Summary 동기화가 핵심입니다.

---

## 📌 목차
- [프로젝트 개요](#프로젝트-개요)
- [주요 기능](#주요-기능)
- [서비스 시연 영상](#서비스-시연-영상)
- [아키텍처](#아키텍처)
- [AI 분석 흐름](#ai-분석-흐름)
- [Redis 성능 개선](#redis-성능-개선)
- [ERD](#erd)
- [실행 방법](#실행-방법)
- [배포 서버](#배포-서버)
- [기술 스택](#기술-스택)
- [API 문서](#api-문서)
- [License & Data Source](#license--data-source)

---

## 프로젝트 개요

- 프로젝트 명: Health Care
- 팀 구성:
| 이름 | 역할 | 담당 영역 |
|------|------|-----------|
| **신원진** | Backend Developer (Lead) | - 오늘 플랜 자동 생성(ensure_today) <br> - Task 완료 토글(멱등 처리) <br> - Nutrition 집계 및 Summary 계산 로직 설계 <br> - 핵심 API 대부분 구현(users, tasks, nutrition, goals) <br> - N+1 최적화(select_related/prefetch_related) <br> - Redis 캐싱 도입 및 성능 개선(p95 latency/RPS 개선) <br> - /healthz /readyz 헬스체크, 통합 예외 처리 <br> - Demo Seed 스크립트 구축 <br> - 전체 백엔드 구조 및 배포 운영 |
| **김수연** |  디자인 / Backend Support | - 프로젝트 일정 및 구조 논의 참여  <br> - Figma 디자인 초안 작성 <br> - Feedbacks / Goals 일부 모델 초안 설계 |
| **이종현** | PM / Model 초안 설계 | -  초기 ERD 초안 작업 일부 기여 <br> - User 모델 초안 설계 <br> - OAuth 적용 참고자료 조사 |
- 프로젝트 설명:
Health Care는 하루 단위의 운동(Task) · 식단(Nutrition) · AI 이미지 분석 · Summary 대시보드 · 목표 관리(Profile)
전 과정을 자동으로 연결해주는 AI 기반 개인 건강관리 플랫폼입니다.
- 프로젝트 시나리오:
사용자가 음식 사진을 업로드하면 AI가 음식명·칼로리·영양소를 분석해 Nutrition에 자동 반영하고,
그 결과가 Summary·Dashboard까지 실시간으로 업데이트되는 일원화된 건강 관리 흐름을 제공합니다.

핵심 기능은 다음과 같습니다:
- 일일 운동 계획 자동 생성 & 체계적 Task 관리
- AI 기반 식단 분석 (HuggingFace ViT)
- Daily Summary로 운동·식단 진행 상황 실시간 확인
- 프로필 기반 목표 설정 & 진행률 트래킹
- 카카오·네이버 소셜 로그인
- Docker 기반 단일 EC2 배포 / Redis·Prometheus·Grafana 모니터링 통합
Health Care는 “사용자가 하루의 운동·식단을 한 번에 관리하고
AI 분석 결과가 즉각 대시보드에 반영되는 것”을 목표로 제작된
엔드투엔드(End-to-End) 건강관리 플랫폼입니다.
---
✅ 빠른 시작 (Quick Start) — 한 번에 실행 가이드
``` bash
# 1) 저장소 클론
git clone https://github.com/swj6824/health-care.git
cd health-care

# 2) 환경 변수 설정 (.env 예시 제공)
cp .env.example .env
# HuggingFace 키(HF_TOKEN), Django SECRET_KEY, USE_SQLITE 여부 등을 입력하세요.

# 3) Docker 기반 실행 (권장)
docker compose up --build
# API, Nginx, Redis, Prometheus, Grafana가 자동으로 기동됩니다.

# 4) 브라우저 접속
http://localhost
# Nginx가 Django(8000) → 외부(80)로 reverse proxy합니다.

# 5) Admin 계정 생성(필요 시)
docker compose exec api python manage.py createsuperuser

# 6) (선택) 데모 데이터 생성
docker compose exec api python manage.py seed_demo --username <your_id> --days 60
```
 ### === 참고 사항 ===
- 로컬 개발 시: USE_SQLITE=True 로 SQLite 사용
- Redis 미사용 시 자동으로 Django 기본 메모리 캐시로 fallback
- /healthz (Nginx), /probe/api-healthz (Django 실제상태)로 헬스 체크 가능
- 정적 파일은 Docker 환경에서 자동 수집됨
---
---
## 서비스 시연 영상

### 🎥 전체 시연 (52초)
로그인 → 운동 Task 생성 → Summary 업데이트 →  
AI 식단 분석(2회) → 영양소 누적 계산 → Nutrition 삭제 →  
프로필 수정 → 로그아웃  
전체 기능 흐름을 52초에 담았습니다.
<video src="docs/시연영상.mp4" width="600" autoplay loop muted></video>

---
## 아키텍처
### Architecture Diagram
<img width="733" height="398" alt="시스템 아키텍쳐" src="https://github.com/user-attachments/assets/6e3713ab-5c3f-44e3-8a3a-a6f1373ffdb9" />

- 요청 경로: 클라이언트(웹 브라우저) → Nginx Reverse Proxy → Django REST API(DRF) → PostgreSQL(RDS) / Redis / HuggingFace AI → 단일 JSON 응답 반환.

- 도메인 구조: DDD 스타일로 `users`, `tasks`, `intakes`, `goals`, `feedbacks`, `ai`, `utils` 앱을 분리하여  
  인증/운동/식단/목표/피드백/AI 분석 기능을 모듈화했습니다.

- 데이터 저장소: 로컬 개발에서는 SQLite를 사용하고, 프로덕션 환경에서는 AWS RDS PostgreSQL을 사용합니다.  
  모든 운영 DB 트래픽은 `sslmode=require` 기반 TLS 암호화로 보호되며, `CONN_MAX_AGE=60`으로 커넥션 재사용을 최적화했습니다.

- 캐시 계층: Redis를 사용해 `/api/today/summary/`, `/api/recommendations/`, `/api/ai/meal-analyze/` 등  
  반복 조회/고비용 연산 결과를 캐싱하여 RPS를 높이고 p95 응답 시간을 줄였습니다.  
  Redis가 없을 경우 Django 기본 메모리 캐시로 자연스럽게 폴백되도록 구성했습니다.

- AI 처리: `POST /api/ai/meal-analyze/` 요청 시 HuggingFace Vision Transformer 모델을 호출해  
  음식 라벨과 confidence를 추출하고, 식약처 영양 CSV(14,584건)와 매칭해 per 100g 기준 영양 정보를 계산합니다.  
  이후 무게(g)를 반영한 실제 섭취량을 산출해 `macros_total` 형태로 반환하며, `meal-commit` 단계에서 DB에 반영됩니다.

- 운영 환경: AWS EC2(Ubuntu) 위에서 Docker Compose로 `api`, `nginx`, `redis`, `prometheus`, `grafana`,  
  `alertmanager`, `node-exporter`, `blackbox`, `cadvisor` 컨테이너를 한 번에 관리합니다.  
  Nginx는 `/api/*` 트래픽을 Django로 라우팅하고, `/healthz`와 `/probe/api-healthz`로 헬스 체크를 제공합니다.

- 스토리지: 사용자 업로드 이미지와 정적 파일은 `django-storages`를 통해 AWS S3 버킷에 저장하여  
  EC2 디스크 용량 이슈를 줄이고, 재배포 시에도 데이터가 유지되도록 했습니다.

- CI/CD: GitHub Actions를 사용해 `main` / `dev` 브랜치에 대한 push 및 PR 시  
  Python 3.12 환경 세팅 → `pip install -r requirements.txt` → `python manage.py test` 파이프라인을 자동 실행합니다.  
  `DJANGO_ENV=ci`를 별도로 분리하여 테스트 환경과 운영 환경 설정이 충돌하지 않도록 구성했습니다.

- 옵저버빌리티: Prometheus + Grafana + Alertmanager + Blackbox Exporter로  
  API 응답속도(p95), RPS, 에러율, EC2/컨테이너 리소스, Redis 메트릭을 모니터링합니다.  
  알람 조건(지연시간 증가, 에러율 상승, 타겟 다운 등)이 충족되면 Alertmanager가 Slack/Discord로 장애 알림을 전송합니다.

---
## 로그인 & 인증 플로우
세션 + JWT + 소셜 로그인이 함께 동작하는 로그인/로그아웃 플로우입니다.
- 프론트엔드: JWT + Session 상태 자동 동기화
- 백엔드: Django SessionAuth + JWT Dual Flow
- 소셜 로그인: Kakao/Naver OAuth → JWT 자동 발급

<video src="docs/로그인%20인증%20구.mov" width="640" controls muted></video>

---

## AI 분석 흐름 (AI Processing Pipeline)


https://github.com/user-attachments/assets/394cea09-cb04-48b7-b070-3efbdbeea0cd


<video src="docs/ai 시연.mp4" width="600" autoplay loop muted></video>
1. **이미지 업로드**  
   사용자가 음식 사진을 업로드하면 Django REST API에서 파일을 수신합니다.

2. **HuggingFace Vision Transformer 모델 호출**  
   Django 서버는 `HF_IMAGE_MODEL`로 지정된 ViT 기반 모델을 호출해  
   음식 라벨(top-1)과 confidence 값을 추출합니다.

3. **영양 성분 계산 (칼로리·탄단지)**  
   모델이 반환한 음식명(label)을 기준으로  
   식약처 영양 데이터(총 14,584건)에서 1:1 매칭을 수행합니다.  
   per 100g 기준 데이터를 weight(g)와 곱해 실제 섭취 영양소를 산출합니다.

4. **Nutrition 테이블에 저장 (meal-commit)**  
   AI 분석 결과는 프리뷰(json 응답)로 먼저 반환되며,  
   사용자가 확정하면 `POST /api/ai/meal-commit/` 요청을 통해  
   Nutrition 및 Daily Nutrition Summary 테이블에 누적 저장됩니다.

5. **Summary · Dashboard 자동 업데이트**  
   저장 직후 Summary 집계 로직이 실행되어  
   - 하루 총 칼로리  
   - 탄단지 비율  
   - 오늘 식사 리스트  
   - 목표 대비 Progress  
   - 추천 운동/식단 인사이트  
   등이 즉시 반영되며 Dashboard 화면에 실시간 업데이트됩니다.

---

## Redis 성능 개선
k6 부하 테스트 + Grafana 대시보드 기반으로, Redis 캐시 도입 전/후를 비교했습니다.

Redis 도입 전:
- 특정 구간에서 ai-meal-analyze view의 p95 지연 시간이 급등
- HTTP 상태 코드별 그래프에서 4xx/5xx 비율이 눈에 띄게 증가하는 구간 존재
- 누적 RPS 그래프가 일정 시점 이후 평탄해지면서 처리 한계에 근접하는 패턴
<img width="1138" height="580" alt="레디스 전" src="https://github.com/user-attachments/assets/0bbe6935-3983-4247-8c72-a7196455a6b3" />


Redis 도입 후:
- 동일 부하에서 ai-meal-analyze p95 지연 시간이 짧고 안정적으로 유지
- Error Rate 패널에서 비정상 응답이 거의 관측되지 않음
- RPS 게이지와 그래프가 더 높은 수준에서 안정적인 throughput 유지
<img width="1437" height="739" alt="레디스 후" src="https://github.com/user-attachments/assets/0e0ee7e9-d02d-456a-822b-0ed7f7e7dba7" />

| 항목 | Redis 이전 | Redis 이후 |
|------|------------|------------|
| API p95 | 5~50초 | **1초 이하** |
| OK 비율 | 93~95% | **100%** |
| 총 처리량 | 1.6K | **8K (+400%)** |
| 오류 | 잦은 422 | **0건** |
| ai-chat 평균 | 5.61초 | **0.63초** |

---

배포 서버
실 서버 주소: http://3.231.64.141/
인프라: AWS EC2(Ubuntu) + Docker Compose + Nginx + Gunicorn + PostgreSQL(RDS) + Redis

기술 스택
Backend: Django 5, Django REST Framework, PostgreSQL, Redis, HuggingFace ViT
DevOps: Docker Compose, Nginx, Gunicorn, Prometheus, Grafana, Alertmanager, Blackbox Exporter
Frontend: Django Templates, Vanilla JS

API 문서
API 상세 명세는 docs/api.md로 제공합니다. (GitHub에서 docs/api.md 파일을 생성 후, 구글 시트/문서에서 정리한 내용을 붙여넣으면 됩니다.)
- [API 상세 문서](docs/api.md)

추가 문서
docs/시연영상.mp4, docs/ai 시연.mp4, docs/로그인 인증 구.mov — 시연용 영상 파일
docs/레디스 전.png, docs/레디스 후.png — Redis 도입 전/후 모니터링 캡처
docs/시스템 아키텍쳐.png — 전체 시스템 아키텍처 다이어그램
docs/api.md — REST API 상세 문서
k6/k6/full_service_2_8_2.js — 부하 테스트 스크립트

License & Data Source

본 프로젝트는 교육·연구용으로 제작된 비상업적 프로젝트입니다.

📌 영양성분 데이터 출처

식품의약품안전처 (MFDS) — 식품영양성분 데이터베이스
https://www.foodsafetykorea.go.kr/
데이터는 식약처 공개 API 및 CSV 자료를 가공하여 사용하였습니다.
모든 데이터는 비상업적 용도로만 활용됩니다.

📌 소스 코드 라이선스 (MIT)
본 프로젝트의 코드는 MIT License를 따릅니다.
