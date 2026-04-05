---
name: wiki
description: Obsidian 기반 LLM Knowledge Base 관리. raw 데이터 컴파일, wiki 인덱싱, Q&A, 린팅, 웹 클리핑 ingest를 수행한다. "/wiki compile", "/wiki qa", "/wiki lint", "/wiki ingest", "/wiki index", "/wiki status", "/wiki file" 등으로 호출.
user-invocable: true
argument-hint: "<command> [args] — compile | index | qa | lint | ingest | status | file"
allowed-tools: "Read,Write,Edit,Glob,Grep,Bash,Agent,WebFetch"
---

# Wiki — Obsidian LLM Knowledge Base

Obsidian vault 내에서 LLM이 개인 지식 베이스를 관리하는 스킬.
모든 문서는 Obsidian Flavored Markdown으로 작성하며, wikilink/frontmatter/callout/tags를 활용한다.

## Vault 구조

```
raw/                ← 원본 소스 (사용자가 넣음)
raw/images/         ← 클리핑 이미지
wiki/
  _index.md         ← 자동 관리 인덱스
  concepts/         ← 단일 개념 문서
  topics/           ← 여러 개념을 아우르는 주제 문서
output/             ← Q&A 결과, 린트 리포트
```

## Commands

### `/wiki compile [path]`

raw/ 소스를 읽고 wiki/ 에 Obsidian 마크다운 문서를 생성/업데이트한다.

1. path 생략 시 raw/ 전체 스캔, path 지정 시 해당 파일만 처리
2. 각 raw 파일을 읽고 핵심 개념/주제를 추출
3. 기존 wiki 문서와 비교 — 신규면 생성, 기존이면 업데이트
4. wiki-compiler 에이전트에 위임하여 문서 생성
5. 완료 후 wiki-indexer 에이전트로 _index.md 갱신

문서 생성 규칙:
- 하나의 개념 → `wiki/concepts/개념명.md`
- 여러 개념을 아우르는 주제 → `wiki/topics/주제명.md`
- frontmatter 필수: title, tags, created, updated, sources
- 본문에 `> [!summary]` callout으로 요약 포함
- 관련 문서는 `[[wikilink]]`로 연결
- 원문이 영어인 경우 핵심 용어 영어 병기

### `/wiki index`

wiki/ 전체를 스캔하여 _index.md를 재생성한다.
wiki-indexer 에이전트에 위임.

_index.md 포맷:
```markdown
---
title: Wiki Index
updated: YYYY-MM-DD
total_articles: N
total_concepts: N
total_topics: N
---
# Wiki Index

## Concepts
- [[개념명]] — 한줄 요약 #tag1 #tag2

## Topics
- [[주제명]] — 한줄 요약 #tag1 #tag2

## Recent Updates
| 날짜 | 문서 | 변경 내용 |
|------|------|----------|
```

### `/wiki qa <질문>`

wiki를 대상으로 질문에 답변한다.

1. _index.md를 읽어 관련 문서 파악
2. 관련 wiki 문서를 읽고 정보 수집
3. 답변을 `output/qa-YYYY-MM-DD-제목.md` 로 저장
4. 답변에 사용된 소스는 `[[wikilink]]`로 표기
5. frontmatter에 question, date, sources 포함

### `/wiki lint`

wiki 전체 데이터 정합성을 체크한다.
wiki-linter 에이전트에 위임.

체크 항목:
- 깨진 `[[wikilink]]` (존재하지 않는 문서 참조)
- frontmatter 누락 또는 비일관성 (필수 필드: title, tags, created, updated)
- 중복 내용 (유사한 제목/내용의 문서)
- 고아 문서 (어디서도 링크되지 않는 문서)
- 새 문서 후보 (여러 문서에서 언급되지만 문서가 없는 개념)
- 결과를 `output/lint-report-YYYY-MM-DD.md` 에 저장

### `/wiki ingest <url>`

웹 페이지를 클리핑하여 raw/ 에 저장한다.

1. WebFetch로 URL 내용을 가져옴
2. 본문을 추출하여 마크다운으로 변환
3. `raw/YYYY-MM-DD-제목.md` 로 저장
4. frontmatter에 source_url, clipped_date 기록
5. 저장 후 사용자에게 compile 여부 확인

### `/wiki status`

Knowledge Base 현황을 표시한다.

1. wiki/ 내 문서 수 (concepts, topics 각각)
2. 총 단어 수
3. 최근 업데이트된 문서 5개
4. 미처리 raw 파일 (wiki에 반영되지 않은 raw 파일)
5. 태그별 분포

### `/wiki file [path]`

output/ 의 Q&A 결과를 wiki에 축적한다.

1. path 지정 시 해당 파일을 wiki에 통합
2. path 생략 시 output/ 내 파일 목록 표시
3. 통합 시 기존 문서 업데이트 또는 새 문서 생성 판단
4. wiki-compiler 에이전트에 위임
5. 완료 후 인덱스 갱신

## 문서 형식 템플릿

```markdown
---
title: 문서 제목
tags:
  - tag1
  - tag2
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources:
  - "[[원본문서]]"
aliases:
  - 별칭
---

# 문서 제목

> [!summary]
> 한두 문장으로 핵심 요약.

## 내용

본문 내용. [[다른문서]]로 링크하며 지식을 연결한다.

## 관련 문서

- [[관련개념1]]
- [[관련주제1]]
```

## 규칙

- wiki/ 내 문서는 이 스킬을 통해서만 생성/수정한다
- 문서 간 연결은 반드시 `[[wikilink]]` 형식을 사용한다
- 모든 문서 변경 시 _index.md를 업데이트한다
- 한국어가 기본이나, 원문이 영어인 경우 핵심 용어는 영어 병기
- concepts/는 단일 개념, topics/는 여러 개념을 아우르는 주제
- 기존 문서 업데이트 시 updated 날짜를 갱신한다
- sources에는 원본 raw 파일이나 URL을 기록한다
