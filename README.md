# hoyuo-cc-plugin

Claude Code 플러그인 모음.

## 설치

```bash
claude /plugin git@github.com:Hoyuo/hoyuo-cc-plugin.git
```

## 플러그인 목록

### default-setup

팀 공용 Claude Code 기본 환경 셋업.

- `CLAUDE.md`에 `<!-- default-setup:start/end -->` 마커로 코딩 가이드라인 삽입/업데이트
- `.claude/settings.json`에 팀 표준 설정 병합
- 기존 사용자 설정은 보존

```
/default-setup
```

### obsidian-wiki

Obsidian 기반 LLM Knowledge Base 관리.

- raw 데이터를 wiki 문서로 컴파일
- 인덱스/백링크 자동 관리
- wiki 대상 Q&A 및 결과 축적
- 데이터 정합성 린팅

```
/wiki compile          # raw/ → wiki/ 컴파일
/wiki index            # _index.md 재생성
/wiki qa <질문>        # wiki 대상 Q&A
/wiki lint             # 정합성 체크
/wiki ingest <url>     # 웹 클리핑 → raw/
/wiki status           # 현황 통계
/wiki file             # output → wiki 축적
```

### prompt-engineering

Claude Code 시스템 프롬프트 아키텍처 기반 프롬프트 엔지니어링.

- 31개 Claude Code 시스템 프롬프트 원문 참조
- 검증된 패턴 기반 프롬프트 작성/분석

```
/prompt-engineering write <목적>       # 프롬프트 작성
/prompt-engineering analyze <프롬프트>  # 프롬프트 분석/개선
/prompt-engineering patterns           # 핵심 패턴 보기
/prompt-engineering reference [번호]   # 원문 참조
```

## 구조

```
.claude-plugin/marketplace.json
plugins/
  default-setup/
  obsidian-wiki/
  prompt-engineering/
```

## License

MIT
