# AnomaliFlow — Enhancement Plan v2.0

> 현재 Python(Streamlit/Dash) 기반 MVP에서  
> **TypeScript + React + Ant Design + PostgreSQL + ORM** 풀스택 웹앱으로 확장하는 로드맵

---

## 1. 확장 목표 (Enhancement Goals)

| 현재 (v1.x) | 목표 (v2.x) |
|------------|------------|
| Streamlit / Dash (Python) UI | React 18 + TypeScript SPA |
| 컴포넌트 스타일 없음 | Ant Design 5.x 디자인 시스템 |
| MongoDB only | PostgreSQL (메인) + MongoDB (결과 캐시) |
| ORM 없음 | SQLAlchemy (Python) + Prisma (TypeScript) |
| 인증 없음 | JWT 기반 사용자 인증 / 권한 관리 |
| 단일 사용자 | 멀티 워크스페이스 / 팀 협업 |

---

## 2. 기술 스택 (Technology Stack)

### 2.1 Frontend

```
React 18                  # UI 프레임워크
TypeScript 5.x            # 타입 안전성
Ant Design 5.x            # 엔터프라이즈 UI 컴포넌트
Ant Design Charts (G2Plot)# 시각화 (시계열, 히트맵, 바 차트)
Ant Design ProComponents  # ProTable, ProForm, ProLayout
Zustand                   # 경량 전역 상태 관리
React Query (TanStack)    # 서버 상태 / API 캐싱
Axios                     # HTTP 클라이언트
React Router v6           # SPA 라우팅
Vite                      # 빌드 도구
```

### 2.2 Backend

```
FastAPI                   # REST API (기존 유지 + 확장)
SQLAlchemy 2.x (async)    # PostgreSQL ORM (Python)
Alembic                   # DB 마이그레이션
asyncpg                   # 비동기 PostgreSQL 드라이버
Pydantic v2               # 요청/응답 스키마 검증
python-jose               # JWT 토큰
passlib[bcrypt]           # 비밀번호 해싱
```

### 2.3 데이터베이스

```
PostgreSQL 16             # 주요 관계형 데이터 (사용자, 실험, 메타데이터)
MongoDB 7.x               # ML 결과 저장 (대용량 비정형 데이터)
Redis 7.x                 # Celery 브로커 + API 결과 캐시
```

### 2.4 인프라 & 오케스트레이션

```
Docker Compose v3.8       # 로컬 개발
Celery 5.x                # 비동기 작업 큐
Prefect 2.x               # 워크플로우 오케스트레이션
Nginx                     # 리버스 프록시 + 정적 파일 서빙
```

---

## 3. 프로젝트 디렉토리 구조 (Project Structure)

```
anomali-flow/
├── backend/                        # FastAPI 백엔드
│   ├── app/
│   │   ├── api/
│   │   │   ├── v1/
│   │   │   │   ├── endpoints/
│   │   │   │   │   ├── auth.py         # 로그인 / 회원가입
│   │   │   │   │   ├── datasets.py     # 데이터셋 CRUD
│   │   │   │   │   ├── experiments.py  # 실험 생성 / 조회
│   │   │   │   │   └── tasks.py        # Celery 태스크 제출 / 조회
│   │   │   │   └── router.py
│   │   │   └── deps.py                 # 의존성 주입 (DB session, 인증)
│   │   ├── core/
│   │   │   ├── config.py               # 환경 변수 (pydantic-settings)
│   │   │   └── security.py             # JWT 유틸리티
│   │   ├── db/
│   │   │   ├── base.py                 # SQLAlchemy Base
│   │   │   ├── session.py              # async session factory
│   │   │   └── init_db.py              # DB 초기화
│   │   ├── models/                     # SQLAlchemy ORM 모델
│   │   │   ├── user.py
│   │   │   ├── workspace.py
│   │   │   ├── dataset.py
│   │   │   └── experiment.py
│   │   ├── schemas/                    # Pydantic 스키마
│   │   │   ├── user.py
│   │   │   ├── dataset.py
│   │   │   ├── experiment.py
│   │   │   └── task.py
│   │   ├── services/
│   │   │   ├── ml/
│   │   │   │   ├── timeseries.py
│   │   │   │   ├── categorical.py
│   │   │   │   └── numerical.py
│   │   │   └── storage.py              # 파일 업로드 서비스
│   │   └── main.py
│   ├── worker/
│   │   ├── celery_app.py
│   │   └── tasks.py
│   ├── migrations/                     # Alembic 마이그레이션
│   │   ├── env.py
│   │   └── versions/
│   ├── alembic.ini
│   └── requirements.txt
│
├── frontend/                           # React + TypeScript 프론트엔드
│   ├── public/
│   ├── src/
│   │   ├── api/                        # Axios API 클라이언트
│   │   │   ├── client.ts
│   │   │   ├── auth.ts
│   │   │   ├── datasets.ts
│   │   │   ├── experiments.ts
│   │   │   └── tasks.ts
│   │   ├── components/
│   │   │   ├── layout/
│   │   │   │   ├── AppLayout.tsx       # ProLayout 기반 전체 레이아웃
│   │   │   │   ├── Sidebar.tsx
│   │   │   │   └── Header.tsx
│   │   │   ├── dataset/
│   │   │   │   ├── DataUploader.tsx    # Ant Design Upload + Dragger
│   │   │   │   ├── DataPreview.tsx     # ProTable 데이터 미리보기
│   │   │   │   └── ColumnSelector.tsx  # 컬럼 선택 Transfer 컴포넌트
│   │   │   ├── experiment/
│   │   │   │   ├── AlgorithmSelector.tsx
│   │   │   │   ├── ParamForm.tsx       # ProForm 동적 파라미터 폼
│   │   │   │   └── ExperimentList.tsx  # ProTable 실험 목록
│   │   │   ├── result/
│   │   │   │   ├── AnomalyChart.tsx    # G2Plot 시계열 이상치 차트
│   │   │   │   ├── RootCauseHeatmap.tsx# G2Plot 히트맵
│   │   │   │   ├── ScoreBarChart.tsx   # 이상치 점수 바 차트
│   │   │   │   └── ResultTable.tsx     # ProTable 결과 테이블
│   │   │   └── common/
│   │   │       ├── TaskStatusBadge.tsx # 작업 상태 Badge
│   │   │       └── LoadingSpinner.tsx
│   │   ├── hooks/
│   │   │   ├── useTaskPolling.ts       # 태스크 상태 폴링 훅
│   │   │   ├── useDataset.ts
│   │   │   └── useExperiment.ts
│   │   ├── pages/
│   │   │   ├── LoginPage.tsx
│   │   │   ├── DashboardPage.tsx
│   │   │   ├── DatasetPage.tsx
│   │   │   ├── ExperimentPage.tsx
│   │   │   └── ResultPage.tsx
│   │   ├── store/
│   │   │   ├── authStore.ts            # Zustand 인증 상태
│   │   │   └── experimentStore.ts
│   │   ├── types/
│   │   │   ├── api.ts                  # API 응답 타입
│   │   │   ├── dataset.ts
│   │   │   ├── experiment.ts
│   │   │   └── algorithm.ts
│   │   ├── utils/
│   │   │   ├── formatters.ts
│   │   │   └── validators.ts
│   │   ├── App.tsx
│   │   ├── main.tsx
│   │   └── vite-env.d.ts
│   ├── package.json
│   ├── tsconfig.json
│   └── vite.config.ts
│
├── docker-compose.yml                  # 전체 스택 구성
├── docker-compose.dev.yml              # 개발 환경 오버라이드
└── nginx/
    └── nginx.conf
```

---

## 4. PostgreSQL 데이터베이스 스키마

### 4.1 SQLAlchemy ORM 모델 (Python)

```python
# backend/app/models/user.py
from sqlalchemy import Column, Integer, String, DateTime, Boolean
from sqlalchemy.orm import relationship
from app.db.base import Base
import datetime

class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True, nullable=False)
    username = Column(String, unique=True, index=True, nullable=False)
    hashed_password = Column(String, nullable=False)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime, default=datetime.datetime.utcnow)

    workspaces = relationship("Workspace", back_populates="owner")
```

```python
# backend/app/models/workspace.py
class Workspace(Base):
    __tablename__ = "workspaces"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, nullable=False)
    owner_id = Column(Integer, ForeignKey("users.id"), nullable=False)
    created_at = Column(DateTime, default=datetime.datetime.utcnow)

    owner = relationship("User", back_populates="workspaces")
    datasets = relationship("Dataset", back_populates="workspace")
    experiments = relationship("Experiment", back_populates="workspace")
```

```python
# backend/app/models/dataset.py
class Dataset(Base):
    __tablename__ = "datasets"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, nullable=False)
    workspace_id = Column(Integer, ForeignKey("workspaces.id"), nullable=False)
    file_path = Column(String, nullable=False)        # 로컬 또는 S3 경로
    row_count = Column(Integer)
    column_count = Column(Integer)
    data_type = Column(String)                        # timeseries / numerical / categorical
    created_at = Column(DateTime, default=datetime.datetime.utcnow)

    workspace = relationship("Workspace", back_populates="datasets")
    experiments = relationship("Experiment", back_populates="dataset")
```

```python
# backend/app/models/experiment.py
class Experiment(Base):
    __tablename__ = "experiments"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, nullable=False)
    workspace_id = Column(Integer, ForeignKey("workspaces.id"), nullable=False)
    dataset_id = Column(Integer, ForeignKey("datasets.id"), nullable=False)
    algorithm = Column(String, nullable=False)
    params = Column(JSON, nullable=False)             # 하이퍼파라미터 JSON
    celery_task_id = Column(String, index=True)       # Celery task_id
    mongo_result_id = Column(String)                  # MongoDB ObjectId
    status = Column(String, default="PENDING")        # PENDING/STARTED/SUCCESS/FAILURE
    outlier_count = Column(Integer)
    created_at = Column(DateTime, default=datetime.datetime.utcnow)
    completed_at = Column(DateTime, nullable=True)

    workspace = relationship("Workspace", back_populates="experiments")
    dataset = relationship("Dataset", back_populates="experiments")
```

### 4.2 ERD (Entity Relationship Diagram)

```
┌──────────┐     1:N     ┌─────────────┐     1:N     ┌──────────────┐
│  User    │────────────▶│  Workspace  │────────────▶│   Dataset    │
│  id (PK) │             │  id (PK)    │             │  id (PK)     │
│  email   │             │  name       │             │  name        │
│  username│             │  owner_id   │             │  workspace_id│
│  password│             │  created_at │             │  file_path   │
└──────────┘             └──────┬──────┘             │  data_type   │
                                │ 1:N                └──────┬───────┘
                                ▼                           │ 1:N
                         ┌──────────────┐                  ▼
                         │  Experiment  │◀─────────────────┘
                         │  id (PK)     │
                         │  name        │
                         │  algorithm   │
                         │  params(JSON)│
                         │  task_id     │
                         │  status      │
                         └──────────────┘
                                │
                                │ mongo_result_id
                                ▼
                         ┌──────────────┐
                         │  MongoDB     │
                         │  (results)   │
                         │  outlier_idx │
                         │  root_cause  │
                         └──────────────┘
```

### 4.3 Alembic 마이그레이션

```bash
# 초기 마이그레이션 생성
alembic init migrations
alembic revision --autogenerate -m "initial_schema"
alembic upgrade head

# 스키마 변경 시
alembic revision --autogenerate -m "add_experiment_tags"
alembic upgrade head
```

### 4.4 비동기 세션 설정

```python
# backend/app/db/session.py
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from sqlalchemy.orm import sessionmaker
from app.core.config import settings

engine = create_async_engine(
    settings.DATABASE_URL,  # postgresql+asyncpg://user:pass@postgres:5432/anomaliflow
    echo=True,
    pool_size=20,
    max_overflow=40,
)

AsyncSessionLocal = sessionmaker(
    engine, class_=AsyncSession, expire_on_commit=False
)

async def get_db() -> AsyncSession:
    async with AsyncSessionLocal() as session:
        yield session
```

---

## 5. React + TypeScript 핵심 컴포넌트

### 5.1 API 타입 정의

```typescript
// frontend/src/types/experiment.ts
export type AlgorithmType =
  | "IsolationForest"
  | "GMM"
  | "LOF"
  | "DBSCAN"
  | "KMeans";

export type DataType = "timeseries" | "numerical" | "categorical";

export type TaskStatus = "PENDING" | "STARTED" | "SUCCESS" | "FAILURE";

export interface ExperimentParams {
  // IsolationForest / GMM
  max_samples?: string | number;
  n_jobs?: number;
  contamination?: number;
  n_components?: number;
  n_init?: number;
  random_state?: number;
  init_params?: string;
  // LOF / DBSCAN
  n_neighbors?: number;
  eps?: number;
  min_samples?: number;
  // KMeans
  n_clusters?: number;
}

export interface Experiment {
  id: number;
  name: string;
  algorithm: AlgorithmType;
  params: ExperimentParams;
  status: TaskStatus;
  outlier_count: number | null;
  created_at: string;
  completed_at: string | null;
}

export interface ExperimentResult {
  outlier_indices: number[];
  outlier_probabilities: number[];
  root_cause_scores: Record<string, Record<string, number>>;
  index: string[];
}
```

### 5.2 태스크 상태 폴링 훅

```typescript
// frontend/src/hooks/useTaskPolling.ts
import { useQuery } from "@tanstack/react-query";
import { getTaskResult } from "../api/tasks";
import type { TaskStatus } from "../types/experiment";

export function useTaskPolling(taskId: string | null) {
  return useQuery({
    queryKey: ["task", taskId],
    queryFn: () => getTaskResult(taskId!),
    enabled: !!taskId,
    refetchInterval: (data) => {
      const status: TaskStatus = data?.status;
      return status === "SUCCESS" || status === "FAILURE" ? false : 2000;
    },
    staleTime: 0,
  });
}
```

### 5.3 알고리즘 파라미터 동적 폼

```typescript
// frontend/src/components/experiment/ParamForm.tsx
import { ProForm, ProFormDigit, ProFormSelect } from "@ant-design/pro-components";
import type { AlgorithmType, ExperimentParams } from "../../types/experiment";

interface ParamFormProps {
  algorithm: AlgorithmType;
  onFinish: (params: ExperimentParams) => Promise<void>;
}

const algorithmParamConfig: Record<AlgorithmType, React.ReactNode> = {
  IsolationForest: (
    <>
      <ProFormDigit name="contamination" label="Contamination" initialValue={0.05} min={0.01} max={0.5} />
      <ProFormDigit name="n_jobs" label="N Jobs" initialValue={-1} />
    </>
  ),
  GMM: (
    <>
      <ProFormDigit name="n_components" label="N Components" initialValue={3} min={1} />
      <ProFormDigit name="n_init" label="N Init" initialValue={10} min={1} />
      <ProFormSelect name="init_params" label="Init Params"
        options={[{ label: "kmeans", value: "kmeans" }, { label: "random", value: "random" }]}
        initialValue="kmeans" />
    </>
  ),
  LOF: (
    <>
      <ProFormDigit name="n_neighbors" label="N Neighbors" initialValue={20} min={2} />
      <ProFormDigit name="contamination" label="Contamination" initialValue={0.05} />
    </>
  ),
  DBSCAN: (
    <>
      <ProFormDigit name="eps" label="Epsilon" initialValue={0.5} min={0.01} />
      <ProFormDigit name="min_samples" label="Min Samples" initialValue={5} min={1} />
    </>
  ),
  KMeans: (
    <>
      <ProFormDigit name="n_clusters" label="N Clusters" initialValue={8} min={2} />
      <ProFormDigit name="n_init" label="N Init" initialValue={10} min={1} />
    </>
  ),
};

export const ParamForm: React.FC<ParamFormProps> = ({ algorithm, onFinish }) => (
  <ProForm onFinish={onFinish}>
    {algorithmParamConfig[algorithm]}
  </ProForm>
);
```

### 5.4 Root Cause 히트맵 컴포넌트

```typescript
// frontend/src/components/result/RootCauseHeatmap.tsx
import { Heatmap } from "@ant-design/charts";
import type { ExperimentResult } from "../../types/experiment";

interface Props {
  result: ExperimentResult;
}

export const RootCauseHeatmap: React.FC<Props> = ({ result }) => {
  const data = Object.entries(result.root_cause_scores).flatMap(
    ([timestamp, scores]) =>
      Object.entries(scores).map(([feature, score]) => ({
        timestamp,
        feature,
        score: parseFloat(score.toFixed(4)),
      }))
  );

  const config = {
    data,
    xField: "timestamp",
    yField: "feature",
    colorField: "score",
    color: ["#BAE7FF", "#1890FF", "#0050B3"],
    label: { style: { fill: "#fff", fontSize: 10 } },
    tooltip: { showMarkers: false },
  };

  return <Heatmap {...config} />;
};
```

---

## 6. Ant Design 레이아웃 및 UI 가이드라인

### 6.1 ProLayout 기반 앱 레이아웃

```typescript
// frontend/src/components/layout/AppLayout.tsx
import { ProLayout } from "@ant-design/pro-components";
import { routes } from "../../router/routes";

export const AppLayout: React.FC = () => (
  <ProLayout
    title="AnomaliFlow"
    logo="/logo.svg"
    route={{ routes }}
    layout="side"
    colorPrimary="#1890FF"
    token={{
      header: { colorBgHeader: "#001529" },
    }}
    menuItemRender={(item, dom) => (
      <Link to={item.path ?? "/"}>{dom}</Link>
    )}
  />
);
```

### 6.2 Ant Design 컴포넌트 활용 매핑

| 기능 | Ant Design 컴포넌트 |
|------|-------------------|
| 파일 업로드 | `Upload.Dragger` |
| 데이터 미리보기 | `ProTable` (가상 스크롤) |
| 컬럼 선택 | `Transfer` 또는 `Select (mode=multiple)` |
| 알고리즘 선택 | `Radio.Group` + `Card` |
| 파라미터 입력 | `ProForm` + `ProFormDigit / ProFormSelect` |
| 작업 상태 표시 | `Steps` + `Badge` |
| 실험 목록 | `ProTable` (검색 / 필터 / 정렬 내장) |
| 결과 차트 | `@ant-design/charts` (Line, Bar, Heatmap) |
| 알림 | `notification` (태스크 완료 시) |
| 로그인 폼 | `ProForm` + `LoginForm` |

---

## 7. 환경 변수 설정 (Environment Variables)

### Backend (.env)

```env
# PostgreSQL
DATABASE_URL=postgresql+asyncpg://anomali:password@postgres:5432/anomaliflow

# MongoDB
MONGO_URL=mongodb://mongo:27017
MONGO_DB=celery_results

# Redis
REDIS_URL=redis://redis:6379/0

# JWT
SECRET_KEY=your-super-secret-key-here
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=1440

# App
DEBUG=false
API_V1_STR=/api/v1
```

### Frontend (.env)

```env
VITE_API_BASE_URL=http://localhost:8000/api/v1
VITE_APP_TITLE=AnomaliFlow
```

---

## 8. Docker Compose 확장 구성

```yaml
version: "3.8"

services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: anomali
      POSTGRES_PASSWORD: password
      POSTGRES_DB: anomaliflow
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - mynetwork

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    networks:
      - mynetwork

  mongo:
    image: mongo:7
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db
    networks:
      - mynetwork

  backend:
    build: ./backend
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
    ports:
      - "8000:8000"
    env_file: ./backend/.env
    depends_on:
      - postgres
      - redis
      - mongo
    networks:
      - mynetwork

  celery_worker:
    build: ./backend
    command: celery -A worker.celery_app worker --loglevel=info --concurrency=4
    env_file: ./backend/.env
    depends_on:
      - redis
      - mongo
      - postgres
    networks:
      - mynetwork

  frontend:
    build: ./frontend
    ports:
      - "3000:80"
    depends_on:
      - backend
    networks:
      - mynetwork

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - frontend
      - backend
    networks:
      - mynetwork

volumes:
  postgres-data:
  mongo-data:

networks:
  mynetwork:
    driver: bridge
```

---

## 9. 마이그레이션 전략 (Migration Strategy)

### Phase 1: 백엔드 확장 (4주)
- [ ] FastAPI 앱 구조 모듈화 (`app/` 패키지 분리)
- [ ] PostgreSQL 연결 및 SQLAlchemy ORM 모델 구현
- [ ] Alembic 마이그레이션 초기 설정
- [ ] JWT 인증 엔드포인트 (`/auth/login`, `/auth/register`)
- [ ] Workspace / Dataset / Experiment API CRUD
- [ ] 기존 MongoDB 결과 저장 로직 유지 (Experiment.mongo_result_id 연동)

### Phase 2: 프론트엔드 구축 (4주)
- [ ] Vite + React + TypeScript 프로젝트 초기화
- [ ] Ant Design ProLayout 기반 레이아웃 구성
- [ ] 로그인 / 회원가입 페이지
- [ ] 데이터셋 업로드 및 미리보기 페이지
- [ ] 실험 생성 (알고리즘 선택 + 파라미터 설정)
- [ ] 태스크 상태 폴링 및 실시간 진행 표시
- [ ] 결과 시각화 (이상치 차트, Root Cause 히트맵)

### Phase 3: 통합 및 운영 (2주)
- [ ] Nginx 리버스 프록시 설정
- [ ] Docker Compose 전체 스택 통합 테스트
- [ ] Prefect 워크플로우 연동 유지
- [ ] 기존 Streamlit/Dash 레거시 병행 운영 → 단계적 교체

---

## 10. API 확장 명세

### 인증 (Authentication)

```
POST /api/v1/auth/register    # 회원가입
POST /api/v1/auth/login       # 로그인 (JWT 발급)
GET  /api/v1/auth/me          # 현재 사용자 정보
```

### 워크스페이스 (Workspace)

```
GET    /api/v1/workspaces          # 내 워크스페이스 목록
POST   /api/v1/workspaces          # 워크스페이스 생성
DELETE /api/v1/workspaces/{id}     # 삭제
```

### 데이터셋 (Dataset)

```
GET    /api/v1/workspaces/{wid}/datasets          # 목록
POST   /api/v1/workspaces/{wid}/datasets/upload   # CSV 업로드
GET    /api/v1/workspaces/{wid}/datasets/{id}     # 상세 + 미리보기
DELETE /api/v1/workspaces/{wid}/datasets/{id}     # 삭제
```

### 실험 (Experiment)

```
GET    /api/v1/workspaces/{wid}/experiments        # 목록
POST   /api/v1/workspaces/{wid}/experiments        # 실험 생성 + 태스크 제출
GET    /api/v1/workspaces/{wid}/experiments/{id}   # 상세 + 결과
DELETE /api/v1/workspaces/{wid}/experiments/{id}   # 삭제
```

### 태스크 (Task)

```
GET /api/v1/tasks/{task_id}    # Celery 태스크 상태 조회 (기존 호환)
```
