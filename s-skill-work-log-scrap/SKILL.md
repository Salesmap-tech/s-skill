---
name: s-skill-work-log-scrap
version: 1.0.0
description: |
  내가 뭐했는지 분석해주는 스킬. GitHub, Linear, Slack 데이터를 종합해서
  활동 요약 리포트를 생성한다.
  Use when asked to "뭐했는지", "활동 분석", "activity", "what did I do",
  "오늘 뭐했지", "이번주 뭐했지", "작업 요약", "활동 요약", "work log", "log scrap".
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
  - Grep
  - Agent
  - ToolSearch
  - AskUserQuestion
  - WebFetch
---

# 활동 분석 스킬

GitHub, Linear, Slack 세 소스에서 내 활동을 수집하고 종합 리포트를 만든다.

## 사용법

- `/s-skill-work-log-scrap` — 오늘 활동 분석
- `/s-skill-work-log-scrap 이번주` — 이번 주 활동 분석
- `/s-skill-work-log-scrap 3d` — 최근 3일 활동 분석
- `/s-skill-work-log-scrap 7d` — 최근 7일 활동 분석
- `/s-skill-work-log-scrap 2025-04-07..2025-04-14` — 특정 기간 활동 분석

인자가 없으면 **오늘** 기준으로 분석한다.

---

## 기간 파싱

| 입력 | 의미 |
|------|------|
| (없음) / 오늘 | 오늘 하루 |
| 어제 | 어제 하루 |
| 이번주 / this week | 이번 주 월~오늘 |
| 지난주 / last week | 지난 주 월~일 |
| Nd (예: 3d, 7d) | 최근 N일 |
| YYYY-MM-DD..YYYY-MM-DD | 특정 기간 |

기간을 파싱해서 `--since`/`--until` 또는 ISO 날짜 필터로 변환한다.

---

## 데이터 수집 (3개 소스를 병렬로)

반드시 **3개 Agent를 동시에 병렬 실행**하여 데이터를 수집한다.

### 1. GitHub (gh CLI)

Bash에서 `gh` 명령어를 사용한다.

```bash
# 내 PR 목록 (생성/머지)
gh pr list --author @me --state all --limit 50 --json title,state,url,createdAt,mergedAt,additions,deletions,reviewDecision

# 내 커밋 (모든 접근 가능한 레포)
gh api "/search/commits?q=author:@me+committer-date:>=${SINCE_DATE}&sort=committer-date&per_page=50" --jq '.items[] | {sha: .sha[0:7], message: .commit.message, repo: .repository.full_name, date: .commit.committer.date}'

# 내가 리뷰한 PR
gh pr list --search "reviewed-by:@me" --state all --limit 30 --json title,state,url,createdAt

# 내 이슈 활동
gh issue list --author @me --state all --limit 30 --json title,state,url,createdAt,closedAt
```

수집 항목:
- PR 생성/머지/리뷰 현황
- 커밋 수와 주요 변경사항
- 이슈 생성/종료

### 2. Linear (MCP)

ToolSearch로 Linear MCP 도구를 찾아서 사용한다.

```
ToolSearch: "+linear"
```

찾은 도구로 다음을 조회:
- 나에게 할당된 이슈 중 해당 기간에 상태 변경된 것
- 내가 생성한 이슈
- 내가 완료한 이슈
- 내가 남긴 코멘트

Linear 도구를 찾지 못하거나 연결 실패 시, 리포트에 아래 안내를 포함한다:
> ⚠️ **Linear 연동 안 됨** — MCP 서버 설정이 필요합니다. `~/.claude/mcp_servers.json`에 Linear MCP 서버를 추가해주세요.

### 3. Slack (MCP)

ToolSearch로 Slack MCP 도구를 찾아서 사용한다.

```
ToolSearch: "+slack"
```

찾은 도구로 다음을 조회:
- 내가 보낸 메시지 (주요 채널)
- 참여한 스레드
- 주요 논의 주제

Slack 도구를 찾지 못하거나 연결 실패 시, 리포트에 아래 안내를 포함한다:
> ⚠️ **Slack 연동 안 됨** — MCP 서버 설정이 필요합니다. `~/.claude/mcp_servers.json`에 Slack MCP 서버를 추가해주세요.

---

## 리포트 생성

수집된 데이터를 종합해서 아래 형식으로 **채팅에 직접 출력**한다.

### 출력 형식

```markdown
# 활동 리포트 — {기간}

## 요약
- 한 줄 요약 (가장 임팩트 있었던 작업 중심)

## GitHub
- PR: N개 생성, N개 머지, N개 리뷰
- 커밋: N개 (주요 레포: repo1, repo2)
- 주요 작업:
  - [PR 제목](url) — 상태
  - ...

## Linear
- 완료: N개
- 진행중: N개
- 생성: N개
- 주요 작업:
  - [이슈 제목] — 상태
  - ...

## Slack
- 메시지: N개
- 주요 논의:
  - #채널: 주제 요약
  - ...

## 하이라이트
- 오늘/이번주 가장 의미 있었던 작업 1~3가지를 짧게 정리
```

### 규칙

1. **숫자 먼저, 디테일은 그 다음**. 요약→상세 순서.
2. **빈 소스는 간결하게 처리**. 데이터 없으면 "활동 없음"으로 한 줄.
3. **하이라이트는 주관적 판단 포함**. 단순 나열이 아니라, 어떤 작업이 왜 중요했는지 한 마디.
4. **존댓말 사용**. "~했습니다" 체.
5. **파일 저장 안 함**. 채팅에만 출력.

---

## 에러 처리

- `gh` 인증 안 됨 → "GitHub CLI 인증이 필요합니다. `gh auth login`을 실행해주세요." 안내
- MCP 도구 못 찾음 / 연결 실패 → 해당 소스는 건너뛰되, 리포트에 세팅 안내 메시지를 반드시 포함
- 데이터 0건 → "해당 기간에 활동이 없습니다" 표시

$ARGUMENTS
