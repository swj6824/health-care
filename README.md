Health Care — AI 기반 개인 맞춤형 운동·식단 관리 플랫폼

AI 식단 분석, 운동 관리, 식단 기록, 목표 관리를 한 플랫폼에서 제공하는 개인 맞춤형 헬스케어 서비스입니다. Django REST Framework, PostgreSQL, Redis, Docker Compose 기반으로 개발되었습니다.

목차

프로젝트 개요

주요 기능

서비스 시연 영상

아키텍처

AI 분석 흐름

Redis 성능 개선

ERD

실행 방법

배포 서버

기술 스택

API 문서

License

프로젝트 개요

Health Care는 운동(Task), 식단(Nutrition), AI 이미지 분석, Summary 대시보드, 목표 관리(Profile)를 통합 제공하는 헬스케어 서비스입니다. 모든 데이터는 하루 단위로 연결되며 Summary에 즉시 반영됩니다.

주요 기능
운동 관리

날짜 기반 Task 자동 생성

완료/스킵 처리

Summary 자동 업데이트

식단 관리

음식별 칼로리, 탄단지 누적 계산

삭제 시 Summary 자동 반영

식약처 14,584건 영양 DB 기반

AI 식단 분석

HuggingFace ViT 모델 기반 이미지 분석

음식명·칼로리·탄단지 추정

기존 식단 데이터와 자동 합산

(여기에 AI 분석 성공 이미지 삽입)

사용자 관리

JWT 인증

카카오 로그인

네이버 로그인

프로필 수정

모니터링

Prometheus + Grafana + Alertmanager + Slack

API 지연, 에러율, 프로브 상태, Redis 지표 모니터링

서비스 시연 영상
전체 시연 (1분 44초)

로그인 → 운동 Task 생성 → Summary 업데이트 →
AI 식단 분석 2회 → 영양소 누적 계산 → Nutrition 삭제 →
프로필 수정 → 로그아웃
전체 기능 흐름을 1분 44초로 확인할 수 있습니다.

(여기에 전체 시연 영상 삽입)

사용자 관리(로그인/소셜 로그인)

카카오 로그인, 네이버 로그인, 로그아웃 흐름을 보여주는 영상입니다.

(여기에 소셜 로그인 영상 삽입)

아키텍처

AWS EC2 단일 인스턴스에서 Docker Compose 기반으로 운영됩니다.

Nginx → Gunicorn → Django API → PostgreSQL / Redis
Prometheus / Grafana / Alertmanager / Blackbox 포함

(여기에 아키텍처 다이어그램 삽입)

AI 분석 흐름

사용자가 음식 사진 업로드

Django → HuggingFace 모델 호출

음식명·칼로리·영양소 추정

Nutrition에 누적 저장

Summary와 Dashboard에 즉시 반영

(여기에 AI 분석 흐름 이미지 삽입)

Redis 성능 개선

k6 + Grafana 실측 기준 Redis 전/후 성능 비교입니다.

항목	Redis 이전	Redis 이후
API p95	5~50초	1초 이하
OK 비율	93~95%	100%
총 처리량	1.6K	8K (+400%)
오류	잦은 422	0건
ai-chat 평균 응답	5.61초	0.63초

(여기에 Grafana Before 이미지)
(여기에 Grafana After 이미지)

ERD

운동, 식단, Summary, User 도메인이 하루 단위로 연결되는 구조입니다.

(여기에 ERD 이미지 삽입)

실행 방법

git clone 후 env 설정, Docker 실행만 하면 바로 동작합니다.

git clone https://github.com/원진레포지토리/healthcare.git

cd healthcare
cp .env.example .env
docker compose up --build

접속: http://localhost:8000

배포 서버

실 서버: http://3.231.64.141/

테스트 계정: swj6824 / dnjswls12@

기술 스택

Backend: Django 5, DRF, PostgreSQL, Redis, HuggingFace ViT
DevOps: Docker Compose, Nginx, Gunicorn, Prometheus, Grafana, Alertmanager
Frontend: Django Templates, Vanilla JS

API 문서

API 상세 명세는 PDF로 제공합니다.

(여기에 API 문서 PDF 링크 삽입)

License

© 2025 Health Care Project
식약처 영양성분 데이터(14,584건) 사용
출처: https://www.foodsafetykorea.go.kr/
