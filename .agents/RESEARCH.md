# Personal Memory Agent MVP 연구 노트

본 연구 노트는 Personal Memory Agent MVP 개발을 위한 기술 스택 선정, 비교 및 구현 순서를 정리합니다. 사용자의 목표, 일기, 관심사를 통합 관리하고 추후 로컬 LLM 및 RAG(Retrieval-Augmented Generation) 시스템으로 확장하기 위한 최적의 경로를 제안합니다.

---

## 1. 프레임워크: FastAPI
FastAPI는 현대적인 Python 웹 프레임워크로, 비동기 지원과 빠른 개발 속도 덕분에 에이전트 백엔드 구축에 최적입니다.

*   **선택 이유:**
    *   **비동기 처리(Async):** LLM 요청이나 DB I/O 대기 중 다른 요청 처리 가능.
    *   **Pydantic 기반 검증:** 데이터 모델 정의가 명확하며 자동 API 문서(Swagger) 생성.
    *   **확장성:** LangChain, LlamaIndex 등 AI 라이브러리와의 결합이 용이함.
*   **비교:** Flask(단순하나 비동기 미흡), Django(강력하나 초기 설정이 무겁고 유연성 낮음).

## 2. 데이터베이스: MongoDB
비정형/반정형 데이터를 다루는 개인 메모리 에이전트 특성상 유연한 스키마를 가진 NoSQL이 적합합니다.

*   **선택 이유:**
    *   **유연한 스키마:** 목표, 할 일, 관심사 자료 등 각기 다른 구조의 데이터를 쉽게 저장.
    *   **JSON 친화적:** FastAPI(Pydantic)와 호환성이 뛰어남.
    *   **Fallback 전략:** MongoDB 연결 실패 시 로컬 개발을 위한 `In-memory` (Dictionary/List) 저장소 구현 용이.
*   **운영 방식:** 개발 초기에는 Docker를 통한 로컬 설치 또는 MongoDB Atlas(무료 티어) 추천.

## 3. Vector DB 및 임베딩 (추후 확장)
MVP 이후 검색 품질을 높이기 위해 벡터 검색(Semantic Search) 도입이 필요합니다.

| 구분 | ChromaDB | Qdrant | Pinecone |
| :--- | :--- | :--- | :--- |
| **특징** | 로컬 설치형, 가볍고 빠름 | 고성능, Rust 기반, 필터링 강점 | Managed 서비스, 운영 부담 적음 |
| **추천** | **MVP 이후 첫 단계** | 대규모 데이터/복잡한 필터링 시 | 클라우드 인프라 선호 시 |

*   **Embeddings:**
    *   **로컬:** `Sentence-Transformers` (Hugging Face) - 비용 무료, 개인 정보 보호 우수.
    *   **API:** OpenAI `text-embedding-3-small` - 높은 품질, 유료.
    *   **결정:** 개인 에이전트이므로 **로컬 임베딩 모델** 사용 권장.

## 4. 로컬 LLM (Qwen / Gemma)
프라이버시와 커스터마이징을 위해 로컬 모델 사용을 지향합니다.

*   **모델 추천:**
    *   **Qwen2 (Alibaba):** 코딩 및 한국어 성능이 준수함.
    *   **Gemma 2 (Google):** 가볍고 한국어 지원 및 논리 추론 성능 우수.
*   **실행 환경:**
    *   **Ollama:** 가장 간편한 로컬 실행 도구 (REST API 제공).
    *   **vLLM:** 고성능 추론 서버 (GPU 필요).
    *   **Hugging Face Transformers:** 세밀한 제어 필요 시.

## 5. 배포 전략
개인 에이전트이므로 보안과 접근성을 고려해야 합니다.

1.  **Containerization:** `Docker`를 사용하여 FastAPI + MongoDB + (Vector DB)를 패키징.
2.  **Hosting:**
    *   **로컬 호스팅:** 개인 PC/NAS에서 실행 (Tailscale 등으로 외부 접근).
    *   **Cloud:** Railway, Render (간편), AWS Lightsail (안정적).
    *   **GPU Hosting:** 로컬 모델 구동 시 RunPod, Lambda Labs 등 고려.

---

## 6. 구현 우선순위 (Implementation Order)

### Phase 1: MVP 기초 (현재 목표)
1.  **Project Scaffold:** FastAPI 기본 구조 및 환경 변수 설정.
2.  **Data Layer:** MongoDB 연결부 및 Repository 패턴(In-memory fallback 포함) 구현.
3.  **Core API (CRUD):** 목표, 할 일, 일기, 관심사 자료 API 개발.
4.  **Mock Service:** 키워드 기반 검색 및 Mock LLM 응답 로직 구현.
5.  **State Logic:** 규칙 기반(Rule-based) 상태 점수 계산기 구현.

### Phase 2: 지능화 (RAG 도입)
1.  **Embedding Integration:** 로컬 임베딩 모델 연결.
2.  **Vector DB Setup:** ChromaDB 연동 및 데이터 동기화 로직.
3.  **RAG Pipeline:** 검색 결과와 프롬프트를 조합하여 LLM에 전달하는 구조.

### Phase 3: 로컬 모델 및 최적화
1.  **Local LLM Connection:** Ollama 등을 통한 실제 Qwen/Gemma 모델 연결.
2.  **Performance Tuning:** 비동기 처리 최적화 및 검색 정확도 개선.
3.  **Personalization:** 사용자 패턴 분석을 통한 맞춤형 제안 기능 강화.

---
**보고자:** Research Agent (Gemini CLI)
**작성일:** 2026-05-20
