# 기여 가이드

s-skills에 새 스킬을 추가하거나 기존 스킬을 개선하는 분들을 환영합니다. 사내 활용이 1차 목적이지만 외부 fork도 자유롭게 가능합니다.

## 새 스킬 추가 절차

1. **디렉토리 생성**: 레포 루트에 `s-skill-{name}/` 폴더를 만듭니다 (kebab-case).
2. **SKILL.md 작성**: 아래 frontmatter 표준에 맞춰 `s-skill-{name}/SKILL.md`를 작성합니다.
3. **README 카탈로그 갱신**: `README.md`의 "포함된 스킬" 표에 한 줄 추가합니다.
4. **CHANGELOG 항목 추가**: `CHANGELOG.md`의 `[Unreleased]` 섹션에 `Added: ...` 한 줄 추가합니다.
5. **PR 생성**: `.github/PULL_REQUEST_TEMPLATE.md`의 체크리스트를 채워서 제출합니다.

## SKILL.md frontmatter 표준

모든 SKILL.md 파일의 첫 부분은 YAML frontmatter여야 합니다.

```yaml
---
name: s-skill-{name}            # 디렉토리명과 동일
version: 1.0.0                   # Semantic Versioning
description: |
  한 줄 요약 + 사용 시점 트리거 문구.
  Use when asked "트리거1", "트리거2", or after ...
allowed-tools:                   # 최소 권한 원칙. 필요한 도구만 명시
  - Bash
  - Read
  - AskUserQuestion
---
```

### `description` 작성 규칙

- 한국어/영어 트리거 문구를 모두 포함합니다 (스킬 자동 호출 정확도 향상).
- "Use when ..." 절을 마지막에 두어 자동 호출 조건을 명시합니다.

### `allowed-tools` 규칙

- 사용하지 않는 도구는 절대 추가하지 않습니다 (최소 권한 원칙).
- 외부 MCP 도구는 prefix까지 정확히 명시합니다 (`mcp__linear-server__list_issues` 등).

## 로컬 개발 (symlink 권장)

레포를 클론한 뒤 글로벌 스킬 디렉토리에 symlink하면 SKILL.md 수정이 즉시 반영됩니다.

```bash
git clone https://github.com/Salesmap-tech/s-skill.git ~/dev/s-skill
mkdir -p ~/.claude/skills

for dir in ~/dev/s-skill/s-skill-*; do
  ln -sfn "$dir" ~/.claude/skills/"$(basename "$dir")"
done
```

수정 후 Claude Code에서 `/reload-plugins` 또는 재시작.

## PR 리뷰 기준

코드 리뷰 시 아래 항목을 확인합니다.

- **한국어 존댓말 톤** 일관성: SKILL.md 내 사용자 응답 예시가 모두 존댓말("~할게요/~할까요?")인지.
- **파괴적 작업 전 사용자 확인**: 파일 삭제, MCP 제거, 메시지 전송 등 되돌릴 수 없는 작업 전에 `AskUserQuestion`으로 명시 확인이 들어가는지.
- **토큰/비밀키 처리**: 환경변수나 설정 파일을 다룰 때 백업(`.bak`)을 만들고, 토큰 값을 `echo`로 출력하지 않는지.
- **에러 폴백**: 외부 도구(MCP, gh CLI 등) 호출 실패 시 사용자에게 다음 액션을 안내하는지.
- **CHANGELOG 갱신**: 모든 PR이 `CHANGELOG.md` `[Unreleased]`에 항목을 추가했는지.
- **README 카탈로그 갱신**: 새 스킬이라면 `README.md`의 "포함된 스킬" 표에 추가됐는지.

## 변경 사항 기록

PR을 머지하기 전에 `CHANGELOG.md`의 `[Unreleased]` 섹션에 변경 내용을 추가합니다. [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) 형식을 따릅니다.

```markdown
## [Unreleased]

### Added
- {스킬명} 스킬 추가 — {한 줄 요약}

### Changed
- {스킬명}: {변경 내용}

### Fixed
- {스킬명}: {버그 설명}
```

릴리스 시 `[Unreleased]`를 `[X.Y.Z] - YYYY-MM-DD`로 승격하고, 새 빈 `[Unreleased]` 섹션을 위에 만듭니다. `VERSION` 파일도 함께 갱신합니다.

## 질문 / 제안

- 버그: [버그 리포트 이슈 만들기](https://github.com/Salesmap-tech/s-skill/issues/new?template=bug_report.md)
- 새 스킬 제안: [스킬 제안 이슈 만들기](https://github.com/Salesmap-tech/s-skill/issues/new?template=skill_proposal.md)
