# default-setup

팀 공용 Claude Code 기본 환경 셋업.

## 사용법

```
/default-setup
```

## 기능

- `CLAUDE.md`에 `<!-- default-setup:start/end -->` 마커로 코딩 가이드라인 삽입/업데이트
- `.claude/settings.json`에 팀 표준 설정 병합
- 기존 사용자 설정은 보존

## 가이드라인 항목

1. **Think Before Coding** — 가정을 명시하고, 불확실하면 질문
2. **Simplicity First** — 요청된 것만 구현, 추측성 코드 금지
3. **Surgical Changes** — 요청과 관련된 라인만 수정
4. **Goal-Driven Execution** — 성공 기준 정의 후 검증까지 루프
5. **Verify Before You Use** — 외부 API는 추측하지 말고 확인
