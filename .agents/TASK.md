Personal Memory Agent MVP 백엔드 구조를 만들고 싶다.

이 프로젝트는 단순 To-do List나 일기장이 아니라,
사용자의 목표/할 일, 일기/회고, 관심사 자료를 함께 저장하고,
새로운 질문이 들어오면 과거 기록을 검색하여 개인 맥락에 맞는 답변을 생성하는 Memory Agent이다.

MVP에서 기억해야 하는 핵심 데이터는 3가지이다.

1. 목표 / 할 일
- 사용자가 무엇을 해야 하는지
- 어떤 장기 목표와 연결되어 있는지
- 완료했는지, 미뤘는지, 마감이 가까운지

2. 일기 / 회고
- 오늘 무엇을 했는지
- 무엇을 못 했는지
- 어떤 감정/에너지 상태였는지
- 반복되는 회피 패턴이 있는지

3. 관심사 자료
- 사용자가 저장한 링크
- 사용자가 올린 사진/캡처/이미지 자료
- 사용자가 취미로 자주 보는 주제
- 반복적으로 수집하는 관심 분야
- 예: AI 논문 링크, 모델 비교 글, 운동 자료, 디자인 참고 이미지, 프로젝트 아이디어 캡처

MVP의 핵심 목표:
- 목표와 할 일을 저장한다.
- 일기/회고를 저장한다.
- 링크나 사진 같은 관심사 자료를 저장한다.
- 저장된 목표, 할 일, 일기, 관심사 자료를 기반으로 간단한 memory search를 수행한다.
- LLM은 아직 실제 모델을 붙이지 않고 mock response로 전체 흐름을 검증한다.
- 추후 Qwen/Gemma 로컬 모델을 붙일 수 있도록 LLM 호출부는 분리 가능한 구조로 만든다.

필수 기능:
1. FastAPI 프로젝트 구조 생성

2. 목표 API
- 목표 등록
- 목표 조회
- 목표 수정
- 목표 삭제

3. 할 일 API
- 할 일 등록
- 할 일 조회
- 할 일 완료/미완료 상태 변경
- 할 일을 특정 목표와 연결

4. 일기/회고 API
- 일기/회고 저장
- 일기/회고 조회
- 감정 상태, 에너지 상태, 태그 저장

5. 관심사 자료 API
- 링크 저장 API
- 사진/이미지 자료 메타데이터 저장 API
- 관심사 자료 조회 API
- 관심사 자료에 태그 저장
- 관심사 자료를 목표나 일기와 연결할 수 있게 설계

6. mock memory search API
- 목표, 할 일, 일기/회고, 관심사 자료에서 keyword 기반으로 관련 기록을 찾는다.
- 실제 embedding/vector DB는 아직 사용하지 않는다.
- 검색 결과에는 source_type을 포함한다.
  예: goal, todo, daily_log, interest_item

7. mock LLM response API
- 현재 질문
- 등록된 목표
- 최근 할 일
- 검색된 과거 일기/회고
- 검색된 관심사 자료
를 조합하여 임시 답변을 반환한다.

8. 상태 점수 mock 계산
- goal_alignment
- task_completion
- avoidance_pattern
- overload_risk
- interest_consistency
- net_state_score
를 임시 rule-based 방식으로 계산한다.

9. MongoDB 연결 구조 생성
- 실제 MongoDB 연결이 가능하면 MongoDB 사용
- 연결 실패 시 in-memory 저장소로 fallback하여 개발 가능하게 처리

10. README에 실행 방법 작성

데이터 구조 초안:

- goals
  - id
  - title
  - category
  - priority
  - deadline
  - success_criteria
  - risk
  - created_at
  - updated_at

- todos
  - id
  - title
  - related_goal_id
  - priority
  - due_date
  - status
  - created_at
  - completed_at

- daily_logs
  - id
  - date
  - content
  - mood
  - energy
  - tags
  - related_goal_ids
  - related_todo_ids
  - created_at

- interest_items
  - id
  - item_type
    - link
    - image
    - note
  - title
  - description
  - url
  - file_path
  - source
  - tags
  - related_goal_ids
  - related_log_ids
  - created_at

- memory_chunks
  - id
  - source_type
    - goal
    - todo
    - daily_log
    - interest_item
    - chat_message
  - source_id
  - content
  - summary
  - tags
  - embedding_status
  - created_at

- state_evaluations
  - id
  - date
  - goal_alignment
  - task_completion
  - avoidance_pattern
  - overload_risk
  - interest_consistency
  - net_state_score
  - reason
  - created_at

예시 시나리오:
사용자가 다음과 같은 데이터를 저장한다.

1. 목표
"LLM/RAG 기반 개인 AI 에이전트 포트폴리오 만들기"

2. 할 일
"FastAPI로 memory search API 만들기"

3. 일기
"오늘 구현은 많이 못 했는데, Qwen/Gemma 로컬 모델 자료를 계속 찾아봤다."

4. 관심사 링크
"https://example.com/qwen-local-llm-guide"
태그: llm, local-model, qwen, rag

이후 사용자가 질문한다.

"나 요즘 어떤 주제에 계속 관심 있어?"

시스템은 목표, 일기, 관심사 링크를 함께 검색하고 다음과 같이 mock 답변한다.

"최근 기록을 보면 단순히 To-do 관리보다, 로컬 LLM과 RAG 기반 개인 에이전트 구현에 관심이 집중되어 있습니다. 특히 Qwen/Gemma 같은 로컬 모델, memory search, FastAPI 백엔드 구조 관련 자료를 반복적으로 저장하고 있습니다."

아직 하지 말 것:
- 실제 로컬 LLM 연결
- 실제 embedding 모델 연결
- 실제 ChromaDB/Qdrant 연결
- 실제 이미지 분석/OCR
- 실제 링크 크롤링/본문 추출
- 복잡한 프론트엔드
- 인증 시스템
- 캘린더 연동
- 외부 웹 검색 연동

중요한 설계 원칙:
- MVP 우선
- 모델보다 저장-검색-응답 흐름을 먼저 검증
- 핵심 데이터는 목표/할 일, 일기/회고, 관심사 자료 3축으로 관리
- 관심사 자료는 처음에는 링크 URL, 이미지 파일 경로, 태그, 설명만 저장
- LLM Provider 구조는 나중에 Qwen/Gemma with Hugging Face Transformers or llama.cpp/GGUF로 교체 가능하게 설계
- MongoDB가 없어도 로컬 개발이 가능해야 함
- API 응답은 프론트엔드가 붙기 쉽게 JSON 형태로 반환
