# obsidian-wiki

Obsidian 기반 LLM Knowledge Base 관리.

raw 소스를 증분 컴파일하여 wiki를 구축하고, 소스 요약 · 인덱싱 · QnA · 린팅 · overview 관리를 수행한다.

## 사용법

```
/wiki compile [path]     # raw/ → wiki/ 증분 컴파일
/wiki index              # index.md 재생성
/wiki qna <질문>         # wiki 대상 Q&A
/wiki lint               # 데이터 정합성 체크
/wiki ingest <url>       # 웹 클리핑 → raw/
/wiki status             # 현황 통계
/wiki file [path]        # output → wiki 환류
/wiki overview           # 종합 문서 생성/갱신
```

## 핵심 원칙

- **환각 방지** — 모든 주장은 소스에 추적 가능. LLM 일반 지식으로 보충하지 않음
- **양방향 링크** — 문서 생성/수정 시 백링크 감사 수행
- **대화형 컴파일** — 컴파일 전 핵심 내용을 사용자에게 보고하고 확인
- **편향 방지** — 단일 소스 개념은 "반론 및 데이터 격차" 섹션 포함

## Vault 구조

```
raw/              ← 원본 소스 (불변)
wiki/
  index.md        ← 자동 관리 인덱스
  log.md          ← 시간순 작업 기록
  overview.md     ← 전체 지식베이스 종합 문서
  sources/        ← 소스별 요약 페이지
  concepts/       ← 단일 개념 문서
  topics/         ← 주제 문서
output/           ← QnA 결과, 린트 리포트
```

## 에이전트

| 에이전트 | 모델 | 역할 |
|---------|------|------|
| wiki-compiler | opus | raw 소스를 wiki 문서로 컴파일 |
| wiki-indexer | sonnet | index.md 자동 생성/갱신 |
| wiki-linter | opus | 데이터 정합성 체크 및 수정 제안 |
