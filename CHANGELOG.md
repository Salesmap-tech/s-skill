# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- `CONTRIBUTING.md` — 새 스킬 추가 절차, SKILL.md frontmatter 표준, 로컬 symlink 가이드, PR 리뷰 기준
- `CHANGELOG.md` — Keep a Changelog 형식 변경 이력
- `VERSION` 파일 — Semantic Versioning 추적
- `.mcp.json.example` — 사용자가 복사해 토큰만 채우면 동작하는 MCP 설정 샘플 (README L65 참조 깨짐 해소)
- `.github/ISSUE_TEMPLATE/bug_report.md` — 버그 리포트용 한국어 이슈 템플릿
- `.github/ISSUE_TEMPLATE/skill_proposal.md` — 새 스킬 제안용 한국어 이슈 템플릿
- `.github/PULL_REQUEST_TEMPLATE.md` — PR 체크리스트 (CHANGELOG 갱신, 신규 사용자 흐름 검토 등)

### Changed
- `README.md` 기여 섹션을 `CONTRIBUTING.md` 포인터로 단축

## [1.0.0] - 2026-04-29

### Added
- 초기 6개 스킬: `s-skill-setup`, `s-skill-teardown`, `s-skill-linkedin-scrap`, `s-skill-work-log-scrap`, `s-skill-slack`, `s-skill-shiftee`
- 프로젝트 단위 설치 옵션 (README)
- `s-skill-teardown`에 스킬 파일 제거 단계 추가
- `s-skill-shiftee` 첫 호출 시 바이너리 자동 다운로드

### Changed
- 프로젝트 단위 설치를 기본 권장으로 변경 (전역보다 우선)
- `s-skill-setup`이 MCP 설정 시 프로젝트 단위 권장
- 스킬 설치 시 모든 스킬을 기본 선택, 에이전트 선택만 인터랙티브로
- `--all` 플래그 제거 — 에이전트 선택을 인터랙티브로 강제
- `s-skill-shiftee` 로그인을 이메일/비밀번호가 아닌 브라우저 쿠키 붙여넣기 방식으로 문서화

### Fixed
- `s-skill-shiftee` 로그인 시 발생하던 `KeyError` (JWT에서 ID 읽도록 수정)
- 설치 가이드의 잘못된 레포 경로

[Unreleased]: https://github.com/Salesmap-tech/s-skill/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/Salesmap-tech/s-skill/releases/tag/v1.0.0
