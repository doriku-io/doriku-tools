# doriku-tools Plan

> doriku 서비스가 제공할 독립 tool 후보 분석 및 우선순위 계획
> 작성일: 2026-02-15

---

## 1. 배경 및 목적

### 1.1 doriku 플랫폼 개요

doriku는 **다양한 AI 에이전트, 서버, 사람, 조직 간 프로젝트 동기화 플랫폼**이다.
Claude Code, Cursor, Windsurf 등 복수의 AI 코딩 도구와 인간 팀원을 하나의 실시간 워크스페이스로 연결하여 개발을 원활하게 하는 데 목적을 둔다.

### 1.2 제공 프로토콜

| Layer | Protocol | Endpoint |
|-------|----------|----------|
| REST API | HTTPS | `https://api.doriku.io/api/v1` |
| WebSocket | WSS | `wss://api.doriku.io/ws` |
| MCP | Streamable HTTP | `https://api.doriku.io/mcp` |
| Webhooks | HTTPS (HMAC-SHA256) | 사용자 설정 |

### 1.3 현재 기능

- 태스크 CRUD, 분해(decompose), 할당(assign)
- 에이전트 등록, 하트비트, 온/오프라인 추적
- 채널 메시지 송수신
- Claude Code Hook 자동 동기화 (`sync-doriku.sh`)
- WebSocket 실시간 이벤트 전파 (~200ms)

### 1.4 이 문서의 목적

doriku의 핵심 차별점(크로스 에이전트, 크로스 조직 동기화)을 기반으로, 시장 gap 분석과 기술 트렌드를 반영하여 **doriku-tools 저장소에서 개발·관리할 tool 후보를 도출**하고 우선순위를 제안한다.

---

## 2. 시장 분석

### 2.1 멀티 AI 에이전트 협업의 핵심 페인 포인트

| # | 페인 포인트 | 상세 | 출처 |
|---|------------|------|------|
| P1 | **Git 충돌 및 파일 경합** | 병렬 에이전트가 같은 파일을 동시 편집 → 통합 문제 보장. Git Worktree가 사실상 표준 우회책이나 수동·인프라 수준의 패턴 | Clash, CCManager |
| P2 | **컨텍스트 단절 및 메모리 상실** | 세션 간 프로젝트 히스토리, 다른 에이전트의 결정, 과거 아키텍처 선택에 대한 인식 제로 | RedMonk 2025 |
| P3 | **생산성 역설** | METR 연구: 숙련 개발자가 AI 도구 사용 시 오히려 **19% 더 오래 걸림** (본인은 20% 빨라졌다고 인식). 관리·리뷰·수정 오버헤드가 속도 이득 상쇄 | METR 2025 |
| P4 | **신뢰도 하락 및 리뷰 부담** | 52% 개발자가 에이전트 영향 체감, 69% 생산성 향상 보고 — but **AI 출력 신뢰도는 동시에 하락 중**. 멀티 에이전트 시 리뷰 부담 폭증 | Stack Overflow 2025 |
| P5 | **도구 파편화** | Claude Code, Cursor, Windsurf, Copilot 간 컨텍스트 모델·파일 처리·프롬프트 아키텍처 상이. 크로스 도구 인식 방법 없음 | 업계 공통 |

### 2.2 기존 플랫폼 비교

| 플랫폼 | 접근 방식 | 강점 | 한계 |
|--------|----------|------|------|
| **Devin** | 완전 자율 단일 에이전트 | 샌드박스 환경, Slack 통신, SWE-bench 3x 성능 | 단일 에이전트. 병렬 협업 불가 |
| **Factory AI** | Agent-Native Development | LLM/인터페이스 불문, 조직 컨텍스트 수집, CI/CD 연동. 200% QoQ 성장 | 폐쇄형. 크로스 도구 조율 제한적 |
| **OpenHands** | 오픈소스 코딩 에이전트 SDK | 67K GitHub stars, SWE-Bench 72% 해결률, 셀프호스팅 | 오케스트레이션보다 플랫폼 빌딩 초점 |
| **Agent-MCP** | MCP 기반 멀티 에이전트 조율 | 지식 그래프, 태스크 관리, 충돌 해결 | 초기 단계, 프로덕션 검증 부족 |
| **AgentBase** | 멀티 도구 비주얼 오케스트레이터 | 비주얼 캔버스, 편집 격리 토글, 진행 추적 | **로컬 전용**. 크로스 서버/조직 불가 |
| **Claude Code Agent Teams** | 네이티브 멀티 에이전트 | 리드 에이전트 → 의존성 그래프 → 백그라운드 에이전트 스폰 | Claude Code에 한정 |
| **doriku** | 크로스 도구/조직 동기화 플랫폼 | MCP+API+WS 3채널, 크로스 서버, 크로스 조직 | tool 생태계 확장 필요 (이 문서의 주제) |

### 2.3 시장 Gap 분석

| Gap | 설명 | 현재 해결책 | doriku 기회 |
|-----|------|------------|------------|
| **크로스 에이전트 인식 레이어** | Agent A가 Agent B의 작업을 모름 | 없음 | **WS 기반 실시간 브로드캐스트로 해결 가능** |
| **사전적 충돌 방지** | 충돌 발생 후 해결만 존재 | Clash (Git 수준만) | **태스크 → 파일 매핑 + AST 분석으로 사전 예측** |
| **통합 리뷰 파이프라인** | 멀티 에이전트 PR 통합 뷰 없음 | 없음 | **PR 집계 + 의존성 + 품질 메트릭 통합** |
| **영속적 프로젝트 메모리** | CLAUDE.md, .cursorrules 등 도구별 분산 | 도구별 개별 파일 | **중앙 컨텍스트 저장소로 통합 동기화** |
| **크로스 조직 거버넌스** | 에이전트 행위 감사, 사람 승인 이력 추적 | 없음 | **불변 감사 로그 + 정책 가드** |
| **비용/효율 분석** | 에이전트별 토큰 소비·API 비용 추적 | 없음 | **중앙 메트릭 수집 + 대시보드** |

### 2.4 MCP 생태계 현황 및 gap

**생태계 규모**: MCP.so 기준 17,647+ 서버 등록

**주요 보안 이슈** (2,614개 MCP 구현 분석):
- 82% — Path Traversal 취약 파일시스템 작업 사용
- 67% — Code Injection 관련 민감 API 사용
- 53% — 장기 정적 비밀키(API key, PAT) 의존
- 8.5%만 OAuth 인증 사용

**프로토콜 수준 gap**:
- 멀티테넌시 미지원
- 관리자 제어(MCP 서버 접근 거버넌스) 없음
- 컨텍스트 인식 동적 디스커버리 없음
- 실시간 푸시 알림 패턴 제한적
- 비인간 클라이언트 ID 표준 없음
- 컴플라이언스 테스트 스위트 없음

### 2.5 주요 트렌드 (Anthropic 2026 Agentic Coding Report)

- **57%** 조직이 멀티 스텝 에이전트 워크플로우 배포 중
- **16%**가 크로스 펑셔널 또는 엔드투엔드 프로세스까지 진행
- Gartner: 멀티 에이전트 시스템 문의 **1,445% 급증**
- 전략 우선순위 4가지:
  1. 멀티 에이전트 조율 마스터
  2. 인간-에이전트 감독 스케일링
  3. 엔지니어링 팀 너머로 에이전틱 코딩 확장
  4. 보안 아키텍처를 설계 핵심 원칙으로 내재화

---

## 3. Tool 후보 상세

### 카테고리 A: Coordination (실시간 충돌 방지)

해결 대상: P1(Git 충돌), P5(도구 파편화)

#### A-1. file-lock

| 항목 | 내용 |
|------|------|
| **목적** | 에이전트가 파일 편집 시 선점 잠금 요청/해제. 동일 파일 접근 시 경고 |
| **프로토콜** | MCP (잠금 요청/해제) + WS (실시간 잠금 상태 브로드캐스트) |
| **MCP Tools** | `acquire_file_lock(file_path, agent_id, ttl)` → 잠금 획득 |
| | `release_file_lock(file_path, agent_id)` → 잠금 해제 |
| | `list_file_locks(workspace_id)` → 현재 잠금 목록 조회 |
| **WS Events** | `file.locked` / `file.unlocked` / `file.lock_conflict` |
| **핵심 로직** | TTL 기반 자동 해제 (에이전트 비정상 종료 대비), 우선순위 큐 (사람 > 에이전트), 디렉터리 수준 잠금 지원 |
| **차별점** | 기존 솔루션은 로컬 전용. doriku는 크로스 서버/조직 간 파일 잠금 제공 |

#### A-2. work-broadcast

| 항목 | 내용 |
|------|------|
| **목적** | 에이전트가 현재 작업 중인 태스크·파일·브랜치를 실시간 브로드캐스트 |
| **프로토콜** | WS (주) + API (이력 조회) |
| **WS Events** | `agent.work_started` / `agent.work_updated` / `agent.work_stopped` |
| **API Endpoints** | `GET /api/v1/agents/{id}/current-work` → 특정 에이전트 현재 작업 |
| | `GET /api/v1/workspaces/{id}/active-work` → 워크스페이스 전체 활성 작업 |
| **데이터 모델** | `{ agent_id, task_id, files: [path], branch, started_at, description }` |
| **핵심 로직** | 하트비트와 통합 — 하트비트 실패 시 자동 work_stopped, 파일 목록 기반 충돌 경고 자동 발생 |
| **차별점** | Claude Code, Cursor, Windsurf 등 이종 에이전트 간 작업 가시성 제공 |

#### A-3. conflict-predict

| 항목 | 내용 |
|------|------|
| **목적** | 두 에이전트의 태스크가 같은 파일/모듈에 영향을 줄 가능성을 사전 분석 |
| **프로토콜** | API (분석 요청) + WS (실시간 경고) |
| **API Endpoints** | `POST /api/v1/predict-conflicts` → 태스크 목록 기반 충돌 예측 |
| | `GET /api/v1/conflict-map/{workspace_id}` → 현재 충돌 위험 맵 |
| **분석 수준** | Level 1: 파일 경로 매칭 (빠름), Level 2: Git diff 기반 변경 영역 분석 (중간), Level 3: AST 수준 의존성 그래프 분석 (정밀) |
| **WS Events** | `conflict.warning` — 충돌 위험 감지 시 관련 에이전트에 푸시 |
| **차별점** | Clash는 Git 수준만 분석. doriku는 태스크 메타데이터 + 파일 의존성 + AST 복합 분석 |

---

### 카테고리 B: Shared Memory (프로젝트 컨텍스트 동기화)

해결 대상: P2(컨텍스트 단절), P5(도구 파편화)

#### B-1. context-sync

| 항목 | 내용 |
|------|------|
| **목적** | CLAUDE.md, .cursorrules, AGENTS.md 등 에이전트별 설정 파일을 통합 관리·동기화 |
| **프로토콜** | MCP (읽기/쓰기) + API (관리) |
| **MCP Tools** | `get_project_context(workspace_id)` → 통합 프로젝트 컨텍스트 반환 |
| | `update_project_context(workspace_id, section, content)` → 컨텍스트 업데이트 |
| | `get_context_for_agent(workspace_id, agent_type)` → 에이전트 유형별 맞춤 컨텍스트 |
| **데이터 모델** | 섹션별 관리: `project_rules`, `code_conventions`, `architecture`, `stack`, `env_setup` |
| **핵심 로직** | 에이전트 유형별 변환 — 동일 규칙을 CLAUDE.md 형식 / .cursorrules 형식 / AGENTS.md 형식으로 자동 변환 제공 |
| **차별점** | 현재 시장에 크로스 도구 컨텍스트 통합 솔루션 전무. doriku가 최초 제공 가능 |

#### B-2. decision-log

| 항목 | 내용 |
|------|------|
| **목적** | ADR(Architecture Decision Record)을 중앙 저장. 모든 에이전트가 결정 이력 조회 가능 |
| **프로토콜** | MCP (CRUD) |
| **MCP Tools** | `log_decision(workspace_id, title, context, decision, consequences)` |
| | `search_decisions(workspace_id, query)` → 키워드/태그 기반 검색 |
| | `get_decision(decision_id)` → 특정 결정 상세 조회 |
| **데이터 모델** | `{ id, title, status(proposed/accepted/deprecated), context, decision, consequences, tags, created_by, created_at }` |
| **핵심 로직** | 불변 로그 (수정 시 새 버전 생성), 태그 기반 관련 결정 자동 연결, 에이전트가 관련 코드 작업 시 관련 ADR 자동 제안 |
| **차별점** | "왜 이렇게 결정했는가"에 대한 크로스 세션, 크로스 에이전트, 크로스 조직 접근 |

#### B-3. project-summary

| 항목 | 내용 |
|------|------|
| **목적** | 코드베이스 구조·스택·주요 패턴을 자동 인덱싱하여 신규 에이전트 온보딩 시 제공 |
| **프로토콜** | API (생성/갱신) + MCP (조회) |
| **API Endpoints** | `POST /api/v1/projects/{id}/index` → 인덱싱 트리거 |
| | `GET /api/v1/projects/{id}/summary` → 프로젝트 요약 조회 |
| **MCP Tools** | `get_project_summary(workspace_id)` → 구조화된 프로젝트 요약 반환 |
| | `get_module_summary(workspace_id, module_path)` → 특정 모듈 상세 |
| **생성 항목** | 디렉터리 트리, 기술 스택 감지, 주요 진입점, API 엔드포인트 목록, DB 스키마 개요, 테스트 커버리지, 최근 변경 히스토리 |
| **차별점** | 에이전트가 새 세션 시작 시 "프로젝트를 처음부터 이해"하는 시간 대폭 단축 |

---

### 카테고리 C: Review & Quality (멀티 에이전트 품질 관리)

해결 대상: P3(생산성 역설), P4(리뷰 부담)

#### C-1. pr-aggregator

| 항목 | 내용 |
|------|------|
| **목적** | 여러 에이전트가 생성한 PR을 통합 뷰로 제공. 상호 의존성, 충돌 가능성 표시 |
| **프로토콜** | API (집계) + WS (실시간 업데이트) |
| **API Endpoints** | `GET /api/v1/workspaces/{id}/prs` → 워크스페이스 전체 PR 목록 |
| | `GET /api/v1/workspaces/{id}/pr-dependencies` → PR 간 의존성 그래프 |
| | `GET /api/v1/workspaces/{id}/pr-conflicts` → PR 간 충돌 분석 |
| **WS Events** | `pr.created` / `pr.updated` / `pr.conflict_detected` / `pr.merged` |
| **핵심 로직** | GitHub/GitLab PR 자동 수집, 변경 파일 교차 분석, 머지 순서 추천, 리뷰 우선순위 산출 |
| **차별점** | 현재 멀티 에이전트 PR 통합 리뷰 도구 전무 |

#### C-2. code-review-request

| 항목 | 내용 |
|------|------|
| **목적** | 에이전트 A가 에이전트 B 또는 사람에게 코드 리뷰 요청 |
| **프로토콜** | MCP (요청 생성) + WS (알림) |
| **MCP Tools** | `request_review(workspace_id, pr_url, reviewer_type, description)` |
| | `list_review_requests(workspace_id, status)` |
| | `submit_review(review_id, result, comments)` |
| **라우팅 로직** | `reviewer_type`: `auto` (시스템이 적합한 리뷰어 선택), `agent` (특정 에이전트), `human` (특정 사람), `team` (팀 채널) |
| **차별점** | 에이전트-에이전트, 에이전트-인간 간 구조화된 리뷰 워크플로우 |

#### C-3. test-coverage-sync

| 항목 | 내용 |
|------|------|
| **목적** | 각 에이전트가 실행한 테스트 결과를 중앙 수집. 전체 커버리지 맵 제공 |
| **프로토콜** | API (결과 업로드/조회) |
| **API Endpoints** | `POST /api/v1/workspaces/{id}/test-results` → 테스트 결과 업로드 |
| | `GET /api/v1/workspaces/{id}/coverage-map` → 통합 커버리지 맵 |
| | `GET /api/v1/workspaces/{id}/untested-changes` → 테스트 안 된 변경사항 |
| **핵심 로직** | 에이전트별 테스트 실행 이력 추적, 중복 테스트 감지, 미테스트 영역 하이라이트, 커버리지 트렌드 |
| **차별점** | 분산 에이전트 환경에서의 테스트 커버리지 통합은 미개척 영역 |

---

### 카테고리 D: DevOps Bridge (외부 서비스 연동)

해결 대상: 엔터프라이즈 채택 가속

#### D-1. ci-status

| 항목 | 내용 |
|------|------|
| **목적** | GitHub Actions, GitLab CI 등의 파이프라인 상태를 워크스페이스로 실시간 전파 |
| **프로토콜** | WS (실시간) + API (이력) |
| **지원 CI** | GitHub Actions, GitLab CI, CircleCI, Jenkins (Webhook 수신) |
| **WS Events** | `ci.run_started` / `ci.run_completed` / `ci.run_failed` |
| **핵심 로직** | CI 실패 시 관련 태스크/에이전트에 자동 알림, 실패 로그 요약 제공, 에이전트가 자동 수정 시도 트리거 가능 |

#### D-2. issue-bridge

| 항목 | 내용 |
|------|------|
| **목적** | Jira, Linear, GitHub Issues 양방향 동기화 |
| **프로토콜** | API (CRUD) + Webhook (외부 → doriku) |
| **지원 서비스** | Jira, Linear, GitHub Issues, GitLab Issues |
| **동기화 방향** | 양방향: doriku 태스크 ↔ 외부 이슈. 상태 변경 자동 전파 |
| **매핑** | doriku `pending` → Jira `To Do`, doriku `running` → Jira `In Progress`, doriku `completed` → Jira `Done` |
| **핵심 로직** | 태스크 생성 시 이슈 자동 생성, PR 링크 자동 연결, 커스텀 필드 매핑 지원 |

#### D-3. deploy-coordinator

| 항목 | 내용 |
|------|------|
| **목적** | 배포 전 모든 에이전트의 미완료 작업 확인, 배포 잠금, 롤백 조율 |
| **프로토콜** | MCP (잠금/해제) + WS (상태 브로드캐스트) |
| **MCP Tools** | `request_deploy_lock(workspace_id, environment, reason)` |
| | `check_deploy_readiness(workspace_id, environment)` → 미완료 작업, 미머지 PR, 실패 CI 체크 |
| | `release_deploy_lock(workspace_id, environment)` |
| **핵심 로직** | 배포 잠금 중 에이전트 작업 일시 중지 알림, 체크리스트 자동 검증, 롤백 시 관련 에이전트에 알림 |

---

### 카테고리 E: Analytics & Governance (감사·비용 추적)

해결 대상: 엔터프라이즈 거버넌스 요구

#### E-1. audit-trail

| 항목 | 내용 |
|------|------|
| **목적** | 에이전트 행위 불변 로그. 어떤 에이전트가 어떤 파일을 언제 수정했는지, 누가 승인했는지 기록 |
| **프로토콜** | API (쓰기/조회) |
| **API Endpoints** | `POST /api/v1/audit-log` → 감사 이벤트 기록 |
| | `GET /api/v1/audit-log?workspace_id=&agent_id=&from=&to=` → 필터 조회 |
| | `GET /api/v1/audit-log/export` → CSV/JSON 내보내기 |
| **이벤트 유형** | `file.modified`, `task.status_changed`, `deploy.requested`, `review.approved`, `policy.violated` |
| **핵심 로직** | append-only 저장, 변조 방지 해시 체인, 보존 기간 정책, 조직별 접근 제어 |

#### E-2. cost-tracker

| 항목 | 내용 |
|------|------|
| **목적** | 에이전트별 토큰 소비량, API 호출 비용, 작업 효율성 메트릭 수집 |
| **프로토콜** | API (수집/조회) + WS (실시간 대시보드) |
| **수집 메트릭** | 에이전트별 토큰 입/출력량, API 호출 횟수, 태스크당 비용, 시간당 완료 태스크 수, 에이전트 유형별 효율 비교 |
| **API Endpoints** | `POST /api/v1/usage` → 사용량 보고 |
| | `GET /api/v1/usage/summary?workspace_id=&period=` → 기간별 요약 |
| | `GET /api/v1/usage/by-agent?workspace_id=` → 에이전트별 상세 |
| **핵심 로직** | 예산 한도 설정 → 임계값 도달 시 경고, 비용 이상치 감지, 팀/프로젝트별 할당 |

#### E-3. policy-guard

| 항목 | 내용 |
|------|------|
| **목적** | 조직별 정책(금지 패턴, 필수 리뷰, 보안 규칙) 위반 시 에이전트 작업 차단/경고 |
| **프로토콜** | MCP (정책 조회/검증) |
| **MCP Tools** | `check_policy(workspace_id, action, context)` → 정책 위반 여부 검증 |
| | `list_policies(workspace_id)` → 활성 정책 목록 |
| | `get_policy_violations(workspace_id, period)` → 위반 이력 |
| **정책 유형** | `file_restriction` (특정 파일/디렉터리 수정 금지), `review_required` (특정 조건에서 사람 리뷰 필수), `secret_detection` (비밀키 커밋 차단), `branch_protection` (보호 브랜치 직접 푸시 금지), `cost_limit` (비용 한도 초과 차단) |
| **핵심 로직** | Hook 통합 — 에이전트 도구 호출 전 정책 자동 검증, 위반 시 차단 + audit-trail 기록 |

---

## 4. 우선순위 및 로드맵

### 4.1 우선순위 평가 기준

| 기준 | 가중치 | 설명 |
|------|--------|------|
| **즉시 가치 (Immediate Value)** | 30% | 사용자가 설치 후 바로 체감하는 가치 |
| **기술 실현성 (Feasibility)** | 25% | 현재 doriku 인프라(MCP/API/WS) 위에 구축 용이도 |
| **시장 차별성 (Differentiation)** | 25% | 경쟁 솔루션 대비 독자적 가치 |
| **엔터프라이즈 견인력 (Enterprise Pull)** | 20% | 유료 고객 전환 기여도 |

### 4.2 우선순위 매트릭스

| 순위 | Tool | 즉시가치 | 실현성 | 차별성 | 엔터프라이즈 | **총점** |
|------|------|---------|--------|--------|------------|---------|
| **1** | context-sync | 9 | 9 | 9 | 7 | **8.6** |
| **2** | file-lock | 9 | 8 | 8 | 7 | **8.1** |
| **3** | work-broadcast | 8 | 9 | 7 | 6 | **7.6** |
| **4** | audit-trail | 6 | 8 | 7 | 10 | **7.6** |
| **5** | issue-bridge | 7 | 7 | 6 | 9 | **7.2** |
| **6** | decision-log | 7 | 8 | 8 | 5 | **7.1** |
| **7** | policy-guard | 5 | 7 | 8 | 9 | **7.0** |
| **8** | conflict-predict | 8 | 5 | 9 | 6 | **7.0** |
| **9** | cost-tracker | 5 | 8 | 6 | 8 | **6.6** |
| **10** | pr-aggregator | 7 | 6 | 7 | 6 | **6.6** |
| **11** | project-summary | 7 | 6 | 6 | 5 | **6.2** |
| **12** | ci-status | 6 | 7 | 4 | 7 | **6.0** |
| **13** | deploy-coordinator | 5 | 6 | 6 | 7 | **5.9** |
| **14** | code-review-request | 6 | 6 | 6 | 5 | **5.8** |
| **15** | test-coverage-sync | 5 | 5 | 6 | 6 | **5.5** |

### 4.3 제안 로드맵

```
Phase 1 (MVP Core) ─────────────────────────────────────────
  context-sync     ██████████  ← 최우선. 기존 MCP 위에 바로 구축
  file-lock        ██████████  ← WS 인프라 활용. 가장 큰 페인 포인트 해결
  work-broadcast   ████████    ← file-lock과 자연스러운 번들

Phase 2 (Enterprise Foundation) ─────────────────────────────
  audit-trail      ████████    ← 엔터프라이즈 거버넌스 기반
  issue-bridge     ████████    ← Jira/Linear 연동 = 채택 관문
  decision-log     ███████     ← 프로젝트 메모리 완성

Phase 3 (Intelligence) ──────────────────────────────────────
  policy-guard     ███████     ← 보안 + 감사 체계 완성
  conflict-predict ███████     ← AST 분석 = 기술적 해자
  cost-tracker     ██████      ← 비용 가시성

Phase 4 (Ecosystem) ─────────────────────────────────────────
  pr-aggregator       ██████
  project-summary     ██████
  ci-status           ██████
  deploy-coordinator  █████
  code-review-request █████
  test-coverage-sync  █████
```

---

## 5. 기술 원칙

### 5.1 저장소 구조

```
doriku-tools/
├── context-sync/        ← 각 tool은 독립 폴더
│   ├── README.md
│   ├── src/
│   ├── tests/
│   └── package.json (또는 go.mod)
├── file-lock/
├── work-broadcast/
├── ...
├── TASK.md
├── HISTORY.md
├── HISTORY_KR.md
└── .gitignore
```

### 5.2 공통 원칙

1. **독립 배포**: 각 tool은 독립적으로 빌드·배포 가능
2. **프로토콜 명시**: 각 tool은 MCP/API/WS 중 사용하는 프로토콜을 명확히 정의
3. **Fail-silent**: 네트워크 장애 시 에이전트 작업을 차단하지 않음 (기존 doriku 철학 준수)
4. **테스트 필수**: 각 tool은 단위 테스트 + 통합 테스트 포함
5. **보안 우선**: OAuth 인증, 입력 검증, Path Traversal 방지 (MCP 보안 gap 교훈 반영)

---

## 6. 참고 자료

- [Anthropic 2026 Agentic Coding Trends Report](https://resources.anthropic.com/hubfs/2026%20Agentic%20Coding%20Trends%20Report.pdf)
- [MCP Official Roadmap](https://modelcontextprotocol.io/development/roadmap)
- [MCP: What's Working, What's Broken](https://www.stackone.com/blog/mcp-where-its-been-where-its-going)
- [State of MCP Server Security 2025](https://astrix.security/learn/blog/state-of-mcp-server-security-2025/)
- [Agent-MCP Framework](https://github.com/rinadelph/Agent-MCP)
- [AgentBase Orchestrator](https://github.com/AgentOrchestrator/AgentBase)
- [Factory AI](https://factory.ai/)
- [Claude Code Agent Teams](https://code.claude.com/docs/en/agent-teams)
- [Clash - Merge Conflict Manager](https://github.com/clash-sh/clash)
- [RedMonk: 10 Things Developers Want from Agentic IDEs](https://redmonk.com/kholterhoff/2025/12/22/10-things-developers-want-from-their-agentic-ides-in-2025/)
- [METR: AI Impact on Developer Productivity](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/)
- [Stack Overflow 2025 Developer Survey - AI](https://survey.stackoverflow.co/2025/ai)
