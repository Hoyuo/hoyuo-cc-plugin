---
name: default-setup
description: 팀 공용 Claude Code 기본 환경을 셋업한다. CLAUDE.md에 마커 태그로 코딩 가이드라인을 삽입/업데이트하고, settings.json을 적용한다. "/default-setup", "기본 세팅", "팀 셋업", "초기 설정" 등으로 호출.
user-invocable: true
argument-hint: "— CLAUDE.md에 가이드라인 삽입/업데이트, settings.json 적용"
allowed-tools: "Read,Write,Edit,Bash,Glob"
---

# Default Setup — 팀 공용 Claude Code 환경 셋업

프로젝트 CLAUDE.md에 마커 태그(`<!-- default-setup:start/end -->`)로 팀 표준 가이드라인을 삽입/업데이트하고, .claude/settings.json을 적용하는 스킬.

## 실행 시 동작

`/default-setup` 실행 시:

### CLAUDE.md 처리

1. 프로젝트 루트에 `CLAUDE.md`를 Read
2. `<!-- default-setup:start -->` ~ `<!-- default-setup:end -->` 마커를 검색
   - **마커가 있으면**: 마커 사이 내용을 아래 템플릿으로 교체 (Edit)
   - **마커가 없으면**: 파일 끝에 마커 + 템플릿을 추가 (Edit)
   - **CLAUDE.md 자체가 없으면**: 마커 포함하여 새로 생성 (Write)
3. 마커 밖의 사용자 내용은 절대 수정하지 않는다

### settings.json 처리

1. `.claude/settings.json`을 Read
   - **있으면**: 아래 템플릿의 키만 병합 (기존 사용자 설정 보존)
   - **없으면**: `.claude/` 디렉토리 생성 후 템플릿으로 새로 생성
2. 사용자가 이미 설정한 다른 키는 건드리지 않는다

### 완료

적용 결과 요약 출력.

## CLAUDE.md 마커 템플릿

CLAUDE.md에 삽입할 내용 (마커 태그 포함):

```markdown
<!-- default-setup:start -->
## Coding Guidelines

### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:

- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:

- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:

1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

### 5. Verify Before You Use

**Don't trust memory for external APIs. Check first.**

- Don't guess API signatures, parameters, or return types. Read the source or docs.
- Don't assume an import path exists. Verify the module exports what you expect.
- If a library version matters, check which version is installed before using new APIs.
- If you're unsure whether a method exists, say so. Don't silently fabricate it.

Common mistakes to avoid:

- Inventing helper functions that don't exist in the library.
- Using v3 API syntax when the project has v2 installed.
- Assuming default parameter values without checking.

The test: Every external call in your code should be traceable to real documentation or source code.

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.
<!-- default-setup:end -->
```

## settings.json 템플릿

`.claude/settings.json`에 적용할 설정:

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": [
      "Bash(*)",
      "Edit(*)",
      "Write(*)",
      "Read(*)",
      "Glob(*)",
      "Grep(*)",
      "WebFetch",
      "WebSearch",
      "NotebookEdit(*)",
      "Task:*"
    ],
    "ask": [
      "Bash(git push:*)",
      "Bash(git reset:*)",
      "Bash(rm -rf:*)"
    ]
  },
  "env": {
    "ENABLE_TOOL_SEARCH": "1",
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1",
    "CLAUDE_CODE_MAX_OUTPUT_TOKENS": "64000"
  },
  "cleanupPeriodDays": 365,
  "enableAllProjectMcpServers": true,
  "language": "korean",
  "alwaysThinkingEnabled": true,
  "teammateMode": "auto"
}
```

## 규칙

- CLAUDE.md의 마커 태그 밖 내용은 절대 수정하지 않는다
- settings.json 병합 시 기존 사용자 설정을 덮어쓰지 않고, 템플릿 키만 추가/업데이트한다
- MCP 서버 설정은 개인별로 다르므로 포함하지 않는다
- 특정 플러그인 활성화 설정은 포함하지 않는다
