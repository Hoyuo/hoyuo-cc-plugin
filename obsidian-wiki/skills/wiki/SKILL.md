---
name: wiki
description: Obsidian 기반 LLM Knowledge Base 관리. 대화형 증분 컴파일, 소스 타입별 추출, 백링크 감사, 모순 감지, 환각 방지, lint-and-heal을 수행한다. "/wiki compile", "/wiki qna", "/wiki lint", "/wiki ingest", "/wiki index", "/wiki status", "/wiki file", "/wiki overview" 등으로 호출.
user-invocable: true
argument-hint: "<command> [args] — compile | index | qna | lint | ingest | status | file | overview"
allowed-tools: "Read,Write,Edit,Glob,Grep,Bash,Agent,WebFetch"
---

# Wiki — Obsidian LLM Knowledge Base

Obsidian vault 내에서 LLM이 개인 지식 베이스를 **증분적으로 구축·관리**하는 스킬.
RAG처럼 매번 원본에서 재검색하는 것이 아니라, 소스가 추가될 때마다 **지속적으로 축적되는 위키**를 유지한다.
모든 문서는 Obsidian Flavored Markdown으로 작성하며, wikilink/frontmatter/callout/tags를 활용한다.

## 핵심 원칙

1. **환각 방지**: 모든 주장은 소스에 추적 가능해야 한다. LLM의 일반 지식으로 내용을 보충하지 않는다.
2. **양방향 링크**: 문서 생성/수정 시 반드시 백링크 감사를 수행한다. 위키가 compound되려면 양방향 링크가 필수.
3. **대화형 컴파일**: 자율 모드 요청이 없으면, 컴파일 전에 핵심 내용을 사용자에게 먼저 보고하고 확인을 받는다.
4. **편향 방지**: 소스가 하나뿐인 개념은 "반론 및 데이터 격차" 섹션을 포함한다.
5. **불변 소스**: raw/ 파일은 LLM이 읽기만 한다. 수정하지 않는다.

## Vault 구조

```
raw/                    ← 원본 소스 (사용자가 넣음, 불변)
raw/images/             ← 클리핑 이미지
wiki/
  index.md             ← 자동 관리 인덱스 (콘텐츠 카탈로그)
  log.md               ← 시간순 작업 기록 (append-only)
  overview.md          ← 전체 지식베이스 종합 문서 (진화형)
  sources/              ← 소스별 요약 페이지 (raw↔wiki 연결 계층)
  concepts/             ← 단일 개념 문서
  topics/               ← 여러 개념을 아우르는 주제 문서
output/                 ← Q&A 결과, 린트 리포트
```

### 계층 설명

- **raw/** — 원본 소스. 사용자가 추가하며, LLM은 읽기만 한다. 불변의 진실 소스(source of truth).
- **wiki/sources/** — 각 raw 소스에 대한 요약 페이지. raw와 wiki를 연결하는 중간 계층. 핵심 내용, 주요 주장, 반론/한계를 포함한다. 소스 타입(article, paper, media 등)에 따라 추출 중점이 달라진다.
- **wiki/concepts/** — 하나의 명확한 개념, 기술, 도구 (예: Transformer, RAG).
- **wiki/topics/** — 여러 개념을 아우르는 넓은 주제 (예: LLM 아키텍처).
- **wiki/index.md** — 전체 문서의 카탈로그. 카테고리/태그별 분류. 쿼리 시 LLM이 먼저 읽어 관련 문서를 찾는다.
- **wiki/log.md** — 모든 작업의 시간순 기록. parseable 포맷으로 `grep "^## \[" wiki/log.md | tail -5` 같은 명령으로 조회 가능.
- **wiki/overview.md** — 전체 지식베이스의 종합 요약. 소스가 추가될 때마다 진화하는 살아있는 문서.

## Commands

### `/wiki compile [path]`

raw/ 소스를 읽고 wiki/ 에 Obsidian 마크다운 문서를 생성/업데이트한다.
하나의 소스가 10~15개 wiki 페이지에 영향을 줄 수 있는 **증분 컴파일**.

1. path 생략 시 raw/ 전체 스캔, path 지정 시 해당 파일만 처리
2. 소스 타입 분류 (article, paper, documentation, media, book-chapter, data-report, discussion)
3. 각 raw 파일을 읽고 핵심 개념/주제/주장을 추출
4. **사용자에게 핵심 takeaway 보고** — 3~5개 핵심 내용, 신규/업데이트 대상 문서, 기존 문서와의 충돌 여부를 먼저 보여주고 강조/생략할 부분을 확인. 자율 컴파일 명시 요청 시에만 생략.
5. `wiki/sources/`에 소스 요약 페이지 생성 (raw 파일과 1:1 대응, 소스 타입별 추출 템플릿 적용)
6. 기존 wiki 문서와 비교 — 신규면 생성, 기존이면 업데이트
7. **모순 감지** — 새 정보가 기존 문서와 충돌 시 `> [!warning] 모순 감지` callout으로 플래깅
8. **백링크 감사** — 새/수정 문서의 제목·aliases를 기존 문서에서 Grep, 매칭 시 양방향 wikilink 추가
9. wiki-compiler 에이전트에 위임하여 문서 생성
10. 완료 후 wiki-indexer 에이전트로 index.md 갱신
11. `wiki/log.md`에 작업 기록 append

문서 생성 규칙:
- 소스 요약 → `wiki/sources/소스명.md` (type: source, source_type: article|paper|...)
- 하나의 개념 → `wiki/concepts/개념명.md` (type: concept)
- 여러 개념을 아우르는 주제 → `wiki/topics/주제명.md` (type: topic)
- frontmatter 필수: title, tags, type, created, updated, sources
- 본문에 `> [!summary]` callout으로 요약 포함
- 모든 주장에 출처를 `([[sources/소스명]])` 형태로 명시
- 관련 문서는 `[[wikilink]]`로 연결
- 원문이 영어인 경우 핵심 용어 영어 병기
- 소스가 하나뿐인 개념은 "반론 및 데이터 격차" 섹션 포함

### `/wiki ingest <url>`

웹 페이지를 클리핑하여 raw/ 에 저장하고, 선택적으로 compile까지 수행한다.

1. WebFetch로 URL 내용을 가져옴
2. 본문을 추출하여 마크다운으로 변환
3. `raw/YYYY-MM-DD-제목.md` 로 저장
4. frontmatter에 source_url, clipped_date, source_type 기록
5. 저장 후 사용자에게 핵심 내용을 요약 보고
6. compile 여부를 확인하고, 승인 시 `/wiki compile` 워크플로우 실행
7. `wiki/log.md`에 ingest 기록 append

### `/wiki index`

wiki/ 전체를 스캔하여 index.md를 재생성한다.
wiki-indexer 에이전트에 위임.

index.md 포맷:
```markdown
---
title: Wiki Index
updated: YYYY-MM-DD
total_articles: N
total_sources: N
total_concepts: N
total_topics: N
---
# Wiki Index

## Sources
- [[소스명]] — 한줄 요약 `#article` #tag1

## Concepts
- [[개념명]] — 한줄 요약 #tag1 #tag2

## Topics
- [[주제명]] — 한줄 요약 #tag1 #tag2

## Recent Updates
| 날짜 | 문서 | 유형 |
|------|------|------|
```

### `/wiki qna <질문>`

wiki를 대상으로 질문에 답변한다.
**위키에서만 답변한다** — LLM의 일반 지식으로 답변하지 않는다.

1. index.md를 읽어 관련 문서 파악
2. 관련 wiki 문서를 읽고 정보 수집. [[wikilink]]를 1단계까지 따라간다.
3. **위키에서 답을 찾을 수 없는 경우** 명시적으로 "위키에 해당 정보가 없습니다"라고 답변하고, 어떤 소스를 추가하면 좋을지 제안한다.
4. 답변의 모든 주장에 `[[wikilink]]` 출처를 명시한다.
5. 소스 간 **합의점과 불일치**를 명시한다.
6. 답변을 `output/qna-YYYY-MM-DD-제목.md` 로 저장
7. frontmatter에 question, date, sources, informed_by 포함
8. **좋은 답변은 wiki에 환류**: 답변이 가치 있다면 사용자에게 `/wiki file`로 wiki에 통합할 것을 제안

답변 형식은 질문 유형에 맞춘다:
- 사실 확인 → 산문 + 인라인 `[[wikilink]]` 출처
- 비교 → 표 + 각 셀에 출처
- 방법론 → 번호 매긴 단계 + 출처
- 종합 분석 → "알려진 것 / 미해결 질문 / 격차" 구조

### `/wiki lint`

wiki 전체 데이터 정합성을 체크하고 **수정을 제안**한다.
wiki-linter 에이전트에 위임.

체크 항목 (11가지):
- 깨진 `[[wikilink]]` — HIGH — 유사 문서 추천 또는 새 문서 생성 권고
- 미해결 모순 — HIGH — 어느 소스가 더 최신/신뢰성 있는지 판단 근거
- 환각 의심 — HIGH — sources 필드 비어있거나 출처 없는 주장 탐지
- frontmatter 누락/비일관성 — MEDIUM
- 소스 커버리지 — MEDIUM — 미처리 raw 파일, 고립 소스 요약
- 백링크 누락 — MEDIUM — 텍스트 언급은 있지만 wikilink 없는 경우
- 중복 내용 — LOW — 병합 대상과 방향 제안
- 고아 문서 — LOW — 관련 문서 추천
- overview 최신성 — LOW
- 단일 소스 편향 — LOW — "반론 및 데이터 격차" 섹션 없는 단일 소스 문서
- 새 문서 후보 — INFO — 참조 빈도 순 정렬

수정 제안:
- 각 이슈에 구체적인 수정 방안을 제시
- 사용자가 승인하면 자동 수정 (heal) 수행 — 적용 전 diff 확인
- 결과를 `output/lint-report-YYYY-MM-DD.md` 에 저장
- `wiki/log.md`에 린트 기록 append

### `/wiki overview`

전체 지식베이스의 종합 문서(overview.md)를 생성하거나 갱신한다.

1. overview.md가 없으면 새로 생성, 있으면 갱신
2. index.md와 주요 문서를 읽어 전체 지식의 흐름을 파악
3. 주요 발견, 핵심 주제, 미해결 질문, 지식 격차를 정리
4. 소스 간 관계와 종합적 통찰을 서술
5. `wiki/log.md`에 overview 갱신 기록 append

overview.md 포맷:
```markdown
---
title: Knowledge Base Overview
updated: YYYY-MM-DD
total_sources: N
---

# Knowledge Base Overview

> [!summary]
> 이 지식베이스의 전체 범위와 핵심 통찰을 요약.

## 핵심 주제

주요 주제와 그 관계를 서술.

## 주요 발견

소스들을 종합하여 도출된 핵심 통찰.

## 반론 및 논쟁점

소스 간 의견이 갈리는 주제와 양측 입장.

## 미해결 질문

추가 조사가 필요한 열린 질문들.

## 지식 격차

부족한 영역, 추가로 필요한 소스 방향.
```

### `/wiki status`

Knowledge Base 현황을 표시한다.

1. wiki/ 내 문서 수 (sources, concepts, topics 각각)
2. 총 단어 수
3. 최근 업데이트된 문서 5개
4. 미처리 raw 파일 (wiki/sources/에 요약 페이지가 없는 raw 파일)
5. 미해결 모순 수
6. 환각 의심 문서 수 (sources 필드 비어있는 문서)
7. 태그별 분포

### `/wiki file [path]`

output/ 의 Q&A 결과를 wiki에 축적한다.
좋은 답변은 사라지지 않고 지식베이스에 환류(promote)된다.

1. path 지정 시 해당 파일을 wiki에 통합
2. path 생략 시 output/ 내 파일 목록 표시
3. 통합 시 기존 문서 업데이트 또는 새 문서 생성 판단
4. wiki-compiler 에이전트에 위임 (백링크 감사 포함)
5. 완료 후 인덱스 갱신
6. `wiki/log.md`에 환류 기록 append (promote 태그)

## 소스 타입 분류

raw 소스를 처리할 때 먼저 타입을 분류하고, 타입에 맞는 추출 전략을 적용한다.

| 타입 | source_type | 추출 중점 |
|------|------------|----------|
| 학술 논문 | paper | 연구 질문, 방법론, 결과, 한계점, 인용 |
| 웹 기사/블로그 | article | 핵심 주장, 근거, 저자 관점 |
| 기술 문서 | documentation | API/설정, 사용법, 아키텍처 |
| 영상/팟캐스트 | media | 핵심 발언, 타임스탬프, 화자별 주장 |
| 서적 챕터 | book-chapter | 챕터 요약, 등장인물/개념, 논증 구조 |
| 데이터/리포트 | data-report | 수치, 트렌드, 방법론, 결론 |
| 토론/스레드 | discussion | 합의점, 논쟁점, 주요 참여자 |

## 문서 형식 템플릿

### 소스 요약 페이지

```markdown
---
title: "소스 제목"
tags:
  - source
  - tag1
type: source
source_type: article | paper | documentation | media | book-chapter | data-report | discussion
created: YYYY-MM-DD
updated: YYYY-MM-DD
raw: "[[raw/원본파일명]]"
source_url: "https://..."
author: "저자명"
---

# 소스 제목

> [!summary]
> 이 소스의 핵심 내용을 2-3문장으로 요약.

## 핵심 내용

소스의 주요 내용을 구조화하여 정리. [[wikilink]]로 연결.

## 주요 주장

- 주장 1
- 주장 2

## 반론 및 한계

소스 자체가 인정한 한계점이나 반론. 없으면 생략.

## 관련 문서

- [[관련개념1]]
```

### Concept/Topic 문서

```markdown
---
title: 문서 제목
tags:
  - tag1
  - tag2
type: concept | topic
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources:
  - "[[sources/소스요약페이지]]"
aliases:
  - 별칭
---

# 문서 제목

> [!summary]
> 한두 문장으로 핵심 요약.

## 내용

본문 내용. [[다른문서]]로 링크하며 지식을 연결한다.
모든 주장에 출처를 ([[sources/소스명]]) 형태로 명시.

## 반론 및 데이터 격차

이 개념에 대한 알려진 반론, 한계, 부족한 데이터.
소스가 하나뿐일 때 특히 중요 — 편향을 방지한다.

## 관련 문서

- [[관련개념1]]
- [[관련주제1]]
```

## 로그 형식

`wiki/log.md`는 append-only 시간순 기록이다. 모든 커맨드 실행 시 기록한다.
`grep "^## \[" wiki/log.md | tail -10` 으로 최근 작업 조회 가능.

```markdown
# Wiki Log

## [YYYY-MM-DD] compile | 소스 제목
- 처리 소스: [[sources/소스파일]]
- 소스 타입: article
- 생성: [[concepts/새문서1]], [[topics/새문서2]]
- 수정: [[concepts/기존문서1]]
- 백링크 추가: N건
- 모순 감지: N건

## [YYYY-MM-DD] ingest | URL 제목
- URL: https://...
- 소스 타입: article
- 저장: [[raw/YYYY-MM-DD-제목]]

## [YYYY-MM-DD] lint
- 검사 문서: N개
- HIGH: N, MEDIUM: N, LOW: N, INFO: N
- 자동 수정: N건
- 리포트: [[output/lint-report-YYYY-MM-DD]]

## [YYYY-MM-DD] overview
- 반영 소스: N개
- 신규 통찰: N건

## [YYYY-MM-DD] file | 환류 문서 제목
- 원본: [[output/qna-파일]]
- 통합: [[concepts/문서명]] (업데이트)
```

## 규칙

- wiki/ 내 문서는 이 스킬을 통해서만 생성/수정한다
- raw/ 파일은 불변이다 — LLM은 읽기만 한다
- **환각 방지**: 모든 주장에 소스 출처를 명시한다. LLM 일반 지식으로 보충하지 않는다.
- **백링크 감사**: 문서 생성/수정 시 반드시 기존 문서에서 관련 언급을 검색하여 양방향 링크를 추가한다.
- **대화형 컴파일**: 자율 모드 명시 요청이 없으면 컴파일 전에 takeaway를 보고하고 사용자 확인을 받는다.
- 문서 간 연결은 반드시 `[[wikilink]]` 형식을 사용한다
- 모든 문서 변경 시 index.md를 업데이트한다
- 모든 작업 시 log.md에 기록을 append한다
- 한국어가 기본이나, 원문이 영어인 경우 핵심 용어는 영어 병기
- type 필드로 문서 유형을 명시한다 (source, concept, topic)
- source 타입 문서는 source_type 필드로 소스 종류를 명시한다
- sources 필드는 raw 파일이 아닌 wiki/sources/ 요약 페이지를 참조한다
- 기존 문서 업데이트 시 updated 날짜를 갱신한다
- 모순 발견 시 기존 내용을 삭제하지 않고 양쪽을 보존하며 callout으로 플래깅한다
- 소스가 하나뿐인 개념은 "반론 및 데이터 격차" 섹션을 포함한다
- 좋은 QnA 답변은 wiki에 환류(promote)할 것을 제안한다
- qna 답변 시 위키에서 답을 찾을 수 없으면 명시적으로 알리고 소스 추가를 제안한다

## 안티패턴 (하지 말 것)

- **일반 지식으로 답변** — 위키에서만 답변한다. 위키와 일반 지식의 불일치 자체가 유용한 시그널.
- **출처 없는 주장** — 모든 주장에 `[[sources/...]]` 출처를 명시한다.
- **백링크 감사 생략** — 양방향 링크 없이는 위키가 compound 되지 않는다.
- **사용자 확인 없이 대량 컴파일** — 자율 모드 명시 요청이 없으면 항상 takeaway 먼저 보고.
- **격차를 숨기기** — 위키에 정보가 없으면 없다고 명시한다. 없는 정보를 만들어내지 않는다.
- **저장 없는 쿼리** — 좋은 답변은 반드시 output/에 저장하고, 가치 있으면 promote를 제안한다.
