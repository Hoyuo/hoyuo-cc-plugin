---
name: prompt-engineering
description: Claude Code 시스템 프롬프트 아키텍처를 기반으로 프롬프트 작성, 분석, 개선을 지원하는 스킬. "프롬프트 작성", "프롬프트 분석", "에이전트 프롬프트", "시스템 프롬프트", "프롬프트 엔지니어링", "prompt engineering" 등으로 트리거.
user-invocable: true
argument-hint: "<command> [args] — write | analyze | patterns | reference"
allowed-tools: "Read,Write,Edit,Glob,Grep,Agent"
---

# Prompt Engineering — Claude Code 시스템 프롬프트 기반

Claude Code의 실제 시스템 프롬프트 아키텍처를 분석하여 축적한 프롬프트 엔지니어링 지식을 활용하는 스킬.
`references/` 디렉토리에 Claude Code의 31개 시스템 프롬프트 원문이 있다. (00~30)

## Commands

### `/prompt-engineering write <목적>`

주어진 목적에 맞는 프롬프트를 작성한다.

1. 목적을 분석하여 적합한 프롬프트 유형 판단 (시스템 프롬프트, 에이전트 프롬프트, 스킬 프롬프트 등)
2. references/에서 유사한 패턴의 프롬프트를 참조
3. Claude Code의 검증된 패턴을 적용하여 프롬프트 작성
4. 작성된 프롬프트를 사용자에게 제시

### `/prompt-engineering analyze <프롬프트>`

기존 프롬프트를 분석하고 개선점을 제안한다.

1. 프롬프트의 구조, 명확성, 완성도를 평가
2. Claude Code 시스템 프롬프트의 패턴과 비교
3. 구체적인 개선 제안을 단계별로 제시
4. 개선된 버전을 작성

### `/prompt-engineering patterns`

Claude Code에서 사용하는 핵심 프롬프트 패턴을 정리하여 보여준다.

### `/prompt-engineering reference [번호]`

references/ 내 특정 프롬프트 원문을 읽고 분석한다.
번호 생략 시 전체 목록 표시.

## 핵심 프롬프트 패턴 (Claude Code 기반)

references/ 디렉토리의 프롬프트를 분석하여 추출한 패턴:

### 1. 계층적 구조 (Layered Architecture)
Claude Code는 프롬프트를 정적 접두부 + 동적 접미부로 분리한다.
- 정적: 정체성, 보안, 도구 사용 규칙 (캐싱 가능)
- 동적: 환경 정보, 메모리, 세션별 설정
- 참조: `references/01_main_system_prompt.md`

### 2. 역할 기반 에이전트 프롬프트 (Role-Based Agent Prompts)
각 에이전트에 명확한 역할, 도구 제한, 행동 규칙을 부여한다.
- 검증 에이전트: 적대적 테스트 전문가 (`references/07_verification_agent.md`)
- 탐색 에이전트: 읽기 전용 제약 (`references/08_explore_agent.md`)
- 패턴: 역할 → 도구 → 제약 → 출력 형식

### 3. 보안 분류 체계 (Security Classification)
2단계 분류기로 도구 호출의 위험도를 판단한다.
- Stage 1: 빠른 분류 (allow/deny)
- Stage 2: 불확실 시 확장 추론
- 참조: `references/12_yolo_auto_mode_classifier.md`

### 4. 컨텍스트 윈도 관리 (Context Window Management)
대화가 길어질 때 정보를 압축/요약하는 전략.
- Micro-compaction → Compact service → Away summary
- 참조: `references/21_compact_service.md`, `references/22_away_summary.md`

### 5. 멀티 에이전트 조정 (Multi-Agent Coordination)
여러 에이전트 간 작업 분배와 통신 프로토콜.
- 조정자가 4단계 워크플로로 관리
- 참조: `references/05_coordinator_system_prompt.md`, `references/06_teammate_prompt_addendum.md`

### 6. 메모리 시스템 (Memory System)
계층적 우선순위로 메모리를 로드하고 관리한다.
- Enterprise → User → Project → Local
- @include 지시어로 전이적 로드
- 참조: `references/24_memory_instruction.md`, `references/16_memory_selection.md`

### 7. 스킬 구조 (Skill Structure)
재사용 가능한 지식 단위를 스킬로 패키징한다.
- 참조: `references/25_skillify.md`, `references/19_simplify_skill.md`

### 8. 출력 효율성 (Output Efficiency)
간결하고 직접적인 응답을 유도하는 패턴.
- "답이나 행동부터, 이유는 그 다음"
- 필러 워드, 서두, 불필요한 전환 제거
- 참조: `references/01_main_system_prompt.md` (Output efficiency 섹션)

## 프롬프트 작성 원칙

references/에서 추출한 실전 원칙:

1. **명확한 정체성 선언** — 첫 줄에서 "너는 X이다"
2. **도구와 제약을 분리** — 할 수 있는 것 vs 해서는 안 되는 것
3. **구체적 예시 포함** — 추상적 규칙보다 예시가 효과적
4. **실패 케이스 명시** — "이럴 때는 하지 마라"
5. **출력 형식 지정** — 기대하는 응답 구조를 템플릿으로
6. **단계별 절차** — 복잡한 작업은 번호 매긴 스텝으로
7. **안전 장치** — 위험한 행동에 대한 확인 절차
8. **톤과 스타일** — 일관된 커뮤니케이션 스타일 명시

## 레퍼런스 목록

| # | 파일 | 설명 |
|---|------|------|
| 00 | `00_overview.md` | 전체 아키텍처 개요 |
| 01 | `01_main_system_prompt.md` | 메인 시스템 프롬프트 |
| 02 | `02_simple_mode.md` | Simple Mode |
| 03 | `03_default_agent_prompt.md` | 기본 에이전트 프롬프트 |
| 04 | `04_cyber_risk_instruction.md` | 사이버 위험 지침 |
| 05 | `05_coordinator_system_prompt.md` | 조정자 시스템 프롬프트 |
| 06 | `06_teammate_prompt_addendum.md` | 팀메이트 프롬프트 부록 |
| 07 | `07_verification_agent.md` | 검증 에이전트 |
| 08 | `08_explore_agent.md` | Explore 에이전트 |
| 09 | `09_agent_creation_architect.md` | 에이전트 생성 아키텍트 |
| 10 | `10_statusline_setup_agent.md` | 상태 줄 설정 에이전트 |
| 11 | `11_permission_explainer.md` | 권한 설명기 |
| 12 | `12_yolo_auto_mode_classifier.md` | Auto Mode 분류기 |
| 13 | `13_tool_prompts.md` | Tool별 프롬프트 |
| 14 | `14_tool_use_summary.md` | Tool 사용 요약 |
| 15 | `15_session_search.md` | 세션 검색 |
| 16 | `16_memory_selection.md` | 메모리 선택 |
| 17 | `17_auto_mode_critique.md` | Auto Mode 비평 |
| 18 | `18_proactive_mode.md` | Proactive Mode |
| 19 | `19_simplify_skill.md` | Simplify 스킬 |
| 20 | `20_session_title.md` | 세션 제목 |
| 21 | `21_compact_service.md` | Compact 서비스 |
| 22 | `22_away_summary.md` | 자리 비움 요약 |
| 23 | `23_chrome_browser_automation.md` | Chrome 브라우저 자동화 |
| 24 | `24_memory_instruction.md` | 메모리 지침 |
| 25 | `25_skillify.md` | Skillify 스킬 |
| 26 | `26_stuck_skill.md` | Stuck 스킬 |
| 27 | `27_remember_skill.md` | Remember 스킬 |
| 28 | `28_update_config_skill.md` | Update Config 스킬 |
| 29 | `29_agent_summary.md` | 에이전트 요약 |
| 30 | `30_prompt_suggestion.md` | 프롬프트 제안 |
