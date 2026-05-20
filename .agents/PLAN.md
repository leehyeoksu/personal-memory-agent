# Personal Memory Agent MVP — Codex 구현 계획

**작성일:** 2026-05-20 | **작성자:** Planner Agent (Claude) | **대상:** Codex

---

## 개요

FastAPI 기반 Personal Memory Agent MVP 백엔드를 구현한다. 목표/할 일, 일기/회고, 관심사 자료를 저장하고, keyword 기반 memory search + mock LLM 응답까지 동작하는 완전한 흐름을 검증한다.

**MVP에서 하지 말 것:** 실제 LLM, 실제 embedding/벡터 DB, 이미지 분석, 링크 크롤링, 인증.

---

## 최종 디렉토리 구조

```
personal-memory-agent/
├── app/
│   ├── main.py                  # FastAPI 앱 진입점
│   ├── config.py                # 환경 변수 (pydantic-settings)
│   ├── database/
│   │   ├── connection.py        # MongoDB 연결 + InMemory fallback
│   │   └── repositories/
│   │       ├── base.py          # 공통 CRUD 인터페이스
│   │       ├── goal_repo.py
│   │       ├── todo_repo.py
│   │       ├── daily_log_repo.py
│   │       ├── interest_item_repo.py
│   │       ├── memory_chunk_repo.py
│   │       └── state_evaluation_repo.py
│   ├── models/
│   │   ├── goal.py
│   │   ├── todo.py
│   │   ├── daily_log.py
│   │   ├── interest_item.py
│   │   ├── memory_chunk.py
│   │   └── state_evaluation.py
│   ├── routers/
│   │   ├── goals.py
│   │   ├── todos.py
│   │   ├── daily_logs.py
│   │   ├── interest_items.py
│   │   ├── memory.py            # mock memory search
│   │   └── agent.py             # mock LLM + state score
│   └── services/
│       ├── memory_search.py
│       ├── llm_provider.py      # ABC + MockLLMProvider
│       └── state_evaluator.py
├── tests/
│   ├── test_goals.py
│   ├── test_todos.py
│   ├── test_daily_logs.py
│   ├── test_interest_items.py
│   └── test_memory_search.py
├── .env.example
├── requirements.txt
└── README.md
```

---

## Step 1: 스캐폴딩

**`requirements.txt`**
```
fastapi>=0.111.0
uvicorn[standard]>=0.29.0
pydantic>=2.7.0
pydantic-settings>=2.2.0
motor>=3.4.0
pymongo>=4.7.0
python-dotenv>=1.0.0
python-multipart>=0.0.9
pytest>=8.2.0
pytest-asyncio>=0.23.0
httpx>=0.27.0
```

**`.env.example`**
```
MONGODB_URI=mongodb://localhost:27017
MONGODB_DB_NAME=personal_memory_agent
USE_IN_MEMORY=false
```

**`app/config.py`** — `pydantic_settings.BaseSettings`로 3개 필드 로드.

**`app/main.py`** — lifespan으로 DB 연결/해제, `/api/v1` prefix로 전체 router 등록, CORS 미들웨어.

---

## Step 2: 데이터 모델 (`app/models/`)

각 파일에 `XxxCreate`, `XxxUpdate` (모든 필드 Optional), `XxxResponse` (id + timestamp 포함) 3개 모델.

| 모델 | 핵심 필드 |
|------|-----------|
| `Goal` | title, category, priority(1~5), deadline, success_criteria, risk |
| `Todo` | title, related_goal_id, priority, due_date, **status**(pending/in_progress/done/deferred), completed_at |
| `DailyLog` | date, content, **mood**(1~5 Enum), **energy**(1~5 Enum), tags[], related_goal_ids[], related_todo_ids[] |
| `InterestItem` | **item_type**(link/image/note), title, description, url, file_path, tags[], related_goal_ids[], related_log_ids[] |
| `MemoryChunk` | source_type(goal/todo/daily_log/interest_item/chat_message), source_id, content, summary, tags[], embedding_status |
| `StateEvaluation` | date, goal_alignment, task_completion, avoidance_pattern, overload_risk, interest_consistency, net_state_score, reason |

---

## Step 3: Database Layer (`app/database/`)

### `connection.py` — 핵심 설계

```
USE_IN_MEMORY=true 또는 MONGODB_URI 미설정  →  InMemoryClient 사용
MongoDB 연결 시도 → 실패 시  →  InMemoryClient로 자동 fallback (경고 로그)
```

`InMemoryClient`: 컬렉션별 `dict[str, dict]` 저장소. Motor와 동일한 비동기 시그니처(`insert_one`, `find_one`, `find`, `update_one`, `delete_one`).

### `repositories/base.py`

```python
class BaseRepository(Generic[T]):
    async def create(self, data: dict) -> dict
    async def get_by_id(self, id: str) -> Optional[dict]
    async def get_all(self, limit: int = 100, skip: int = 0) -> list[dict]
    async def update(self, id: str, data: dict) -> Optional[dict]
    async def delete(self, id: str) -> bool
    async def search_by_keywords(self, keywords: list[str], fields: list[str]) -> list[dict]
```

`search_by_keywords`: OR 방식, 대소문자 무시. MongoDB → `$regex`, InMemory → `str.lower() in`.

각 `XxxRepository`는 `BaseRepository` 상속, 컬렉션 이름과 검색 필드만 오버라이드.

---

## Step 4: API 라우터 (`app/routers/`)

모든 라우터: `Annotated[DatabaseClient, Depends(get_db)]` 패턴.

| 라우터 | prefix | 주요 엔드포인트 |
|--------|--------|----------------|
| `goals` | `/api/v1/goals` | CRUD 5개 |
| `todos` | `/api/v1/todos` | CRUD 5개 + `PATCH /{id}/status` |
| `daily_logs` | `/api/v1/daily-logs` | POST, GET(date_from/date_to 필터), GET by id, DELETE |
| `interest_items` | `/api/v1/interest-items` | CRUD + GET(item_type/tags 필터) |
| `memory` | `/api/v1/memory` | `GET /search?q=<query>&limit=10` |
| `agent` | `/api/v1/agent` | `POST /ask`, `GET /state-score?date=YYYY-MM-DD` |

### `GET /memory/search` 응답 형식
```json
{
  "query": "로컬 LLM",
  "total": 3,
  "results": [
    { "source_type": "interest_item", "id": "...", "title": "...", "tags": ["llm"] }
  ]
}
```

### `POST /agent/ask` 처리 흐름
1. 질문 키워드로 memory search
2. 최근 goals(3개) + todos(5개) 조회
3. `LLMProvider.generate(question, context)` 호출
4. MockLLMProvider가 source_type별 집계 후 템플릿 응답 반환

---

## Step 5: 서비스 레이어 (`app/services/`)

### `memory_search.py`
4개 컬렉션 `asyncio.gather` 병렬 검색 → `source_type` 필드 추가 → `created_at` 내림차순 정렬.

검색 필드:
- goals: `title, category, success_criteria, risk`
- todos: `title`
- daily_logs: `content, tags`
- interest_items: `title, description, tags, source`

### `llm_provider.py`
```python
class LLMProvider(ABC):
    async def generate(self, question: str, context: list[dict]) -> str: ...

class MockLLMProvider(LLMProvider):
    # context 없음 → "아직 저장된 기록이 없어 답변하기 어렵습니다."
    # context 있음 → source_type 집계 후 템플릿 문자열
```

향후 교체: `LLM_PROVIDER=ollama` 환경 변수로 `OllamaProvider` 주입.

### `state_evaluator.py` — Rule-based 점수

| 지표 | 계산 방법 | 가중치 |
|------|-----------|--------|
| `task_completion` | 최근 7일 완료 todo / 전체 todo | 0.30 |
| `goal_alignment` | related_goal_id 있는 todo 비율 | 0.30 |
| `avoidance_pattern` | 최근 3일 deferred todo 존재 시 패널티 | 0.15 |
| `overload_risk` | 오늘 마감 todo ≥ 3개 시 상승 | 0.10 |
| `interest_consistency` | tags 다양성 역수 (집중 = 높음) | 0.15 |
| `net_state_score` | 위 가중평균 | — |

---

## Step 6: Docker



---

## Step 7: 테스트 (`tests/`)

항상 `USE_IN_MEMORY=true`로 실행. pytest + httpx `AsyncClient`.

| 파일 | 핵심 검증 |
|------|-----------|
| `test_goals.py` | 생성→id 반환, 없는 id→404, 수정 확인 |
| `test_todos.py` | 기본 status=pending, PATCH done→completed_at 자동 설정 |
| `test_daily_logs.py` | mood/energy 범위 외 → 422 |
| `test_interest_items.py` | url 없이 link 타입 저장 허용, tags 필터 동작 |
| `test_memory_search.py` | 저장 후 검색→source_type 포함, 없는 키워드→빈 배열 |

---

## 구현 순서

```
1. requirements.txt, .env.example
2. app/config.py
3. app/database/connection.py  (InMemoryClient → MongoDB 순)
4. app/models/ (6개)
5. app/database/repositories/base.py
6. app/database/repositories/ (나머지 5개)
7. app/services/memory_search.py
8. app/services/llm_provider.py
9. app/services/state_evaluator.py
10. app/routers/ (6개)
11. app/main.py
12. tests/ (5개)
13. README.md
```

---

## 주의사항

- **id**: `str(uuid.uuid4())` 기본 사용. MongoDB `_id` → `id` 매핑 통일.
- **날짜 직렬화**: Pydantic v2 `model_config = ConfigDict(...)`.
- **비동기**: 모든 DB 호출 `async/await`. InMemoryClient도 `async def`.
- **에러**: 없는 리소스 → `HTTPException(404)`. 잘못된 입력 → Pydantic 422 자동 처리.
- **LLMProvider 주입**: `app.state`에 저장, router에서 `Depends`로 주입.
