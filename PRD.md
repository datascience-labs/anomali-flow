# AnomaliFlow — Product Requirements Document (PRD)

> 버전: 1.0.0  
> 작성일: 2026-04-12  
> 상태: 초안 (Draft)

---

## 1. 제품 개요 (Product Overview)

**AnomaliFlow**는 데이터 과학자 및 ML 엔지니어가 다양한 유형의 데이터셋에서 이상치(Anomaly)를 탐지하고, 근본 원인(Root Cause)을 분석할 수 있도록 지원하는 **비동기 ML 파이프라인 플랫폼**입니다.

CSV 파일 업로드부터 알고리즘 선택, 비동기 실행, 결과 시각화까지 end-to-end 워크플로우를 단일 플랫폼에서 처리합니다.

---

## 2. 문제 정의 (Problem Statement)

| 문제 | 현황 |
|------|------|
| 이상치 탐지 알고리즘 적용의 복잡성 | 각 알고리즘마다 별도 코드 작성 필요 |
| 대용량 데이터 처리 시 UI 블로킹 | 동기식 처리로 인한 사용자 경험 저하 |
| 결과 추적 및 재현성 부족 | 실험 이력 관리 미흡 |
| 근본 원인 분석의 자동화 필요 | 수동 feature 분석에 의존 |

---

## 3. 목표 사용자 (Target Users)

- **데이터 과학자**: 빠른 프로토타이핑 및 알고리즘 비교 실험
- **ML 엔지니어**: 이상치 탐지 파이프라인의 운영 환경 배포
- **데이터 분석가**: 코드 없이 CSV 업로드 후 시각적 결과 확인
- **시스템 운영자**: 실시간 시스템 메트릭 모니터링

---

## 4. 핵심 기능 요구사항 (Core Functional Requirements)

### 4.1 데이터 입력 (Data Ingestion)

| ID | 요구사항 | 우선순위 |
|----|---------|---------|
| F-01 | CSV 파일 업로드 (drag & drop 지원) | P0 |
| F-02 | 데이터 미리보기 (첫 10행 표시) | P0 |
| F-03 | 컬럼 타입 자동 감지 (숫자형 / 범주형 / 시계열) | P1 |
| F-04 | 클라우드 스토리지 마운트 (S3, GCS) | P2 |
| F-05 | 실시간 스트리밍 데이터 입력 (Kafka 연동) | P3 |

### 4.2 피처 엔지니어링 (Feature Engineering)

| ID | 요구사항 | 우선순위 |
|----|---------|---------|
| F-10 | X 컬럼 다중 선택 | P0 |
| F-11 | 결측값 처리 전략 선택 (median / mean / drop) | P1 |
| F-12 | 스케일링 방법 선택 (MinMaxScaler, StandardScaler) | P1 |
| F-13 | 시계열 인덱스 컬럼 지정 | P1 |

### 4.3 알고리즘 선택 및 실행

| ID | 요구사항 | 데이터 유형 | 우선순위 |
|----|---------|-----------|---------|
| F-20 | IsolationForest | 수치형 / 시계열 | P0 |
| F-21 | GMM (Gaussian Mixture Model) | 수치형 / 시계열 | P0 |
| F-22 | LOF (Local Outlier Factor) | 범주형 / 수치형 | P0 |
| F-23 | DBSCAN | 범주형 / 수치형 | P0 |
| F-24 | KMeans | 수치형 | P0 |
| F-25 | 하이퍼파라미터 커스터마이징 UI | 전체 | P1 |

### 4.4 비동기 작업 관리 (Async Task Management)

| ID | 요구사항 | 우선순위 |
|----|---------|---------|
| F-30 | Celery 기반 비동기 태스크 실행 | P0 |
| F-31 | 실시간 작업 상태 폴링 (PENDING / STARTED / SUCCESS / FAILURE) | P0 |
| F-32 | 작업 취소 기능 | P1 |
| F-33 | 작업 이력 조회 (task_id 기반) | P1 |
| F-34 | Prefect 워크플로우 오케스트레이션 연동 | P2 |

### 4.5 결과 시각화 (Result Visualization)

| ID | 요구사항 | 우선순위 |
|----|---------|---------|
| F-40 | 이상치 인덱스 목록 표시 | P0 |
| F-41 | 이상치 확률 점수 시각화 (bar chart) | P0 |
| F-42 | Root Cause Score 히트맵 (feature × time) | P1 |
| F-43 | 시계열 데이터 이상치 오버레이 차트 | P1 |
| F-44 | 결과 CSV/JSON 다운로드 | P1 |

### 4.6 결과 저장 및 관리 (Result Persistence)

| ID | 요구사항 | 우선순위 |
|----|---------|---------|
| F-50 | MongoDB에 실행 결과 자동 저장 | P0 |
| F-51 | task_id 기반 결과 재조회 | P0 |
| F-52 | 실험 메타데이터 저장 (알고리즘, 파라미터, 타임스탬프) | P1 |

---

## 5. 비기능 요구사항 (Non-Functional Requirements)

| 카테고리 | 요구사항 |
|---------|---------|
| **성능** | CSV 10만 행 기준 알고리즘 실행 완료 시간 ≤ 60초 |
| **확장성** | Celery worker 수평 확장(scale-out) 지원 |
| **가용성** | API 서버 99.9% uptime (Docker health check) |
| **보안** | API 엔드포인트 인증 (향후 JWT) |
| **관측성** | 작업 로그 중앙화 (향후 ELK 스택) |
| **이식성** | Docker Compose 기반 단일 명령 실행 |

---

## 6. 시스템 아키텍처 (System Architecture)

```
┌──────────────┐     HTTP      ┌─────────────────┐
│  Streamlit   │ ─────────────▶│   FastAPI        │
│  / Dash UI   │               │  (REST API)      │
└──────────────┘               └────────┬─────────┘
                                        │ Celery Task
                                        ▼
                               ┌─────────────────┐
                               │   Redis          │
                               │  (Broker)        │
                               └────────┬─────────┘
                                        │
                               ┌────────▼─────────┐
                               │  Celery Worker   │
                               │  (scikit-learn)  │
                               └────────┬─────────┘
                                        │ Store Results
                                        ▼
                               ┌─────────────────┐
                               │   MongoDB        │
                               │  (Results DB)    │
                               └─────────────────┘

                               ┌─────────────────┐
                               │   Prefect        │
                               │  (Orchestration) │
                               └─────────────────┘
```

---

## 7. API 명세 (API Specification)

### 7.1 태스크 생성

```
POST /tasks
Content-Type: application/json

{
  "df": { "col1": [...], "col2": [...] },
  "algorithm": "IsolationForest" | "GMM" | "LOF" | "DBSCAN" | "KMeans",
  "params": {
    // IsolationForest
    "max_samples": "auto",
    "n_jobs": -1,
    "contamination": 0.05

    // GMM
    // "n_init": 10, "n_components": 3, "random_state": 42, "init_params": "kmeans"

    // LOF / DBSCAN
    // "n_neighbors": 20, "eps": 0.5, "min_samples": 5
  }
}

Response 200:
{ "task_id": "abc123" }
```

### 7.2 태스크 결과 조회

```
GET /tasks/{task_id}

Response 200 (SUCCESS):
{
  "status": "SUCCESS",
  "result": {
    "outlier_indices": [3, 7, 42],
    "outlier_probabilities": [0.92, 0.87, 0.71],
    "root_cause_scores": {
      "2023-01-01": { "col1": 0.8, "col2": 0.2 }
    },
    "index": ["2023-01-01", ...],
    "_id": "mongo_object_id"
  }
}
```

---

## 8. 데이터 모델 (Data Model)

### MongoDB: `celery_results.results`

```json
{
  "_id": "ObjectId",
  "task_id": "string",
  "status": "success | failure",
  "algorithm": "string",
  "params": { ... },
  "created_at": "ISODate"
}
```

---

## 9. 지원 알고리즘 파라미터 (Algorithm Parameters)

| 알고리즘 | 파라미터 | 기본값 |
|---------|---------|-------|
| IsolationForest | max_samples, n_jobs, contamination | auto, -1, 0.05 |
| GMM | n_components, n_init, random_state, init_params | 3, 10, 42, "kmeans" |
| LOF | n_neighbors, n_jobs, contamination | 20, -1, 0.05 |
| DBSCAN | eps, min_samples, n_jobs | 0.5, 5, -1 |
| KMeans | n_clusters, n_init | 8, 10 |

---

## 10. 마일스톤 (Milestones)

| 버전 | 범위 | 상태 |
|-----|------|------|
| v0.1 | FastAPI + Celery + MongoDB + Redis 기본 파이프라인 | ✅ 완료 |
| v0.2 | Streamlit / Dash 기반 UI | ✅ 완료 |
| v0.3 | Prefect 워크플로우 오케스트레이션 연동 | ✅ 완료 |
| v0.4 | Root Cause Score 계산 | ✅ 완료 |
| v1.0 | Docker Compose 통합 배포 | ✅ 완료 |
| v2.0 | React + TypeScript + Ant Design 웹앱 | 🔲 계획 |
| v2.1 | PostgreSQL + ORM (SQLAlchemy/Prisma) | 🔲 계획 |
| v3.0 | 멀티테넌트 / 인증 / 대시보드 | 🔲 계획 |

---

## 11. 성공 지표 (Success Metrics)

- 알고리즘 실행 성공률 ≥ 99%
- 평균 API 응답 시간 ≤ 200ms (태스크 제출 기준)
- 사용자 CSV 업로드 → 결과 확인 전체 시간 ≤ 2분 (10만 행 기준)
- Docker Compose 단일 명령으로 전체 스택 5분 이내 기동

---

## 12. 알려진 제약 및 위험 (Constraints & Risks)

| 항목 | 내용 |
|------|------|
| MongoDB 결과 저장 | 대용량 root_cause_scores 저장 시 문서 크기 제한(16MB) 주의 |
| Celery 브로커 | Redis 단일 장애점 — 운영 환경에서 Redis Sentinel/Cluster 필요 |
| 인증 없음 | 현재 API 무인증 상태 — 내부망 전용 또는 VPN 환경 필수 |
| Python GIL | CPU 집약적 ML 연산은 multiprocessing prefork pool 권장 |
