# s-skills

세일즈맵 사내 Claude Code 스킬 모음집.

## 포함된 스킬

| 스킬 | 설명 |
|------|------|
| [s-skill-setup](./s-skill-setup) | 설치 후 MCP·gh·Shiftee를 대화형으로 설정해주는 마법사 |
| [s-skill-teardown](./s-skill-teardown) | setup의 반대 — 항목별로 물어보면서 MCP·토큰·로그인·스킬 파일을 선택 제거 |
| [s-skill-linkedin-scrap](./s-skill-linkedin-scrap) | 키워드·인물명으로 링크드인 포스트 검색·수집·저장 |
| [s-skill-work-log-scrap](./s-skill-work-log-scrap) | GitHub·Linear·Slack 활동을 종합한 활동 요약 리포트 생성 |
| [s-skill-slack](./s-skill-slack) | Slack MCP 래퍼 — 채널/DM 조회·검색·작성 |
| [s-skill-shiftee](./s-skill-shiftee) | 번들된 shiftee CLI로 근태·휴가·스케줄 조회 및 출퇴근 수정 |
| [s-skill-interview-to-ticket](./s-skill-interview-to-ticket) | 고객 인터뷰 피드백을 세일즈맵 CRM 티켓으로 생성 |
| [s-skill-ui-writing](./s-skill-ui-writing) | SMWS 기준 UI 문구 작성 및 검토 |

## 설치

### 방법 1. `skills` CLI 사용 (권장)

아래 명령을 실행하면 **8개 스킬이 전부 자동 선택**되고, **설치할 에이전트만 대화형으로 물어봅니다** (Claude Code 기본 체크). 설치 범위만 둘 중 하나로 고르세요.

**A. 현재 프로젝트에만 설치 (권장)** — 이 레포에서만 사용 (`./.claude/skills/`)

```bash
npx --yes skills add Salesmap-tech/s-skill -s '*'
```

**B. 전역 설치** — 어떤 프로젝트에서든 사용 가능 (`~/.claude/skills/`)

```bash
npx --yes skills add Salesmap-tech/s-skill -s '*' -g
```

에이전트 선택 화면에서 스페이스로 토글, 엔터로 확정. Claude Code만 쓴다면 그냥 엔터.

> ⚠️ `--all` 플래그는 붙이지 마세요. 시스템에 감지된 **모든 AI 에이전트 디렉토리**(`.augment`, `.bob`, `.cortex`, `.roo`, `.windsurf` 등 30+ 개)에 한꺼번에 설치됩니다.

### 방법 2. 수동 설치

```bash
git clone https://github.com/Salesmap-tech/s-skill.git
cp -r s-skill/s-skill-setup ~/.claude/skills/
cp -r s-skill/s-skill-teardown ~/.claude/skills/
cp -r s-skill/s-skill-linkedin-scrap ~/.claude/skills/
cp -r s-skill/s-skill-work-log-scrap ~/.claude/skills/
cp -r s-skill/s-skill-slack ~/.claude/skills/
cp -r s-skill/s-skill-shiftee ~/.claude/skills/
cp -r s-skill/s-skill-interview-to-ticket ~/.claude/skills/
cp -r s-skill/s-skill-ui-writing ~/.claude/skills/
```

Shiftee CLI 바이너리는 스킬 첫 호출 시 자동으로 내려받습니다 (`~/.cache/s-skill-shiftee/shiftee`). 오프라인 환경이라면 수동으로 미리 복사해두세요:

```bash
mkdir -p ~/.cache/s-skill-shiftee
cp s-skill/bin/shiftee ~/.cache/s-skill-shiftee/shiftee
chmod +x ~/.cache/s-skill-shiftee/shiftee
```

## 사전 준비

> 💡 **처음이시라면 `/s-skill-setup` 한 번 실행하세요.**
> Claude Code가 직접 인터뷰하면서 필요한 것만 골라 설정해줍니다. 아래 수동 가이드는 트러블슈팅이 필요할 때만 참고하면 됩니다.

### 1. MCP 서버 설정

프로젝트 루트의 `.mcp.json` 또는 전역 `~/.claude/.mcp.json`에 아래 MCP 서버를 등록합니다. 전체 예시는 [`.mcp.json.example`](./.mcp.json.example) 참고.

#### Linear MCP

HTTP 타입, 별도 설치 불필요. Claude Code가 처음 호출할 때 브라우저로 Linear OAuth 로그인 창이 뜹니다.

```json
"linear-server": {
  "type": "http",
  "url": "https://mcp.linear.app/mcp"
}
```

#### Slack MCP

`slack-mcp-server` npm 패키지를 stdio 방식으로 실행합니다. **본인의 Slack user OAuth 토큰(`xoxp-...`)이 필요합니다.**

```json
"slack": {
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "slack-mcp-server"],
  "env": {
    "SLACK_MCP_XOXP_TOKEN": "xoxp-여기에-본인-토큰",
    "SLACK_MCP_ADD_MESSAGE_TOOL": "true"
  }
}
```

토큰 발급 방법:
1. https://api.slack.com/apps 에서 새 앱 생성 (또는 기존 앱 사용)
2. **User Token Scopes**에 최소 `channels:history`, `channels:read`, `search:read`, `users:read`, `chat:write` 추가
3. 워크스페이스에 앱 설치 → **OAuth & Permissions** 탭에서 `User OAuth Token` 복사 (`xoxp-...`로 시작)

#### Notion MCP

HTTP 타입, 별도 설치 불필요. 처음 호출 시 브라우저 OAuth.

```json
"notion": {
  "type": "http",
  "url": "https://mcp.notion.com/mcp"
}
```

### 2. GitHub CLI

```bash
brew install gh
gh auth login
```

로그인 시 최소 스코프: `repo`, `read:org`. 기본 옵션으로 진행하면 됩니다.

### 3. Salesmap API 토큰 (선택)

`s-skill-interview-to-ticket` 스킬을 쓰려면 세일즈맵 API 토큰이 필요합니다. 토큰을 `~/.zshrc`(또는 `~/.bashrc`)에 환경변수로 등록하세요.

```bash
export SALESMAP_API_TOKEN="여기에-본인-토큰"
```

등록 후 셸을 재시작하거나 `source ~/.zshrc`를 실행합니다.

### 4. Shiftee CLI 로그인 (선택)

`s-skill-shiftee` 스킬을 쓰려면 설치 후 한 번 로그인하면 됩니다. 바이너리는 첫 호출 시 `~/.cache/s-skill-shiftee/shiftee`로 자동 다운로드되니, 그 경로를 바로 쓰시면 됩니다.

```bash
~/.cache/s-skill-shiftee/shiftee login
```

로그인 방식은 **이메일/비밀번호**입니다 (쿠키 복사 아님):

1. 이메일 입력
2. 비밀번호 입력 (화면에 표시되지 않음)
3. CLI가 account 토큰 → 직원 정보 → employee 토큰을 자동 발급 (여러 회사 소속이면 번호로 선택)

토큰·계정 정보는 `~/.config/shiftee-cli/config.json`(0600)에 저장되고, 이후 모든 `s-skill-shiftee` 호출이 자동 인증됩니다. 비밀번호는 저장하지 않습니다(로그인 시에만 사용). 토큰 수명이 길어(account ~5년, employee ~1년) 보통 한 번만 로그인하면 되고, employee 토큰이 만료돼도 account 토큰으로 자동 갱신됩니다.

## 사용법

설치 후 Claude Code에서 슬래시 명령으로 호출합니다.

```
/s-skill-linkedin-scrap [키워드]
/s-skill-work-log-scrap [기간]
/s-skill-slack [자연어 요청]
/s-skill-shiftee [자연어 요청]
/s-skill-interview-to-ticket [Notion URL, Linear URL, 또는 인터뷰 내용]
/s-skill-ui-writing [작성: 컴포넌트 유형 + 상황] 또는 [검토: 기존 문구]
```

정리가 필요하면 `/s-skill-teardown` — 항목별로 물어보면서 선택적으로 제거합니다.

각 스킬 상세 사용법은 개별 `SKILL.md` 참조.

## 기여

기여 가이드(스킬 추가 절차, SKILL.md frontmatter 표준, 로컬 symlink 개발법, PR 리뷰 기준)는 [`CONTRIBUTING.md`](./CONTRIBUTING.md) 참고. 변경 이력은 [`CHANGELOG.md`](./CHANGELOG.md).

- 버그 리포트: [이슈 만들기](https://github.com/Salesmap-tech/s-skill/issues/new?template=bug_report.md)
- 새 스킬 제안: [이슈 만들기](https://github.com/Salesmap-tech/s-skill/issues/new?template=skill_proposal.md)

## 라이선스

사내 전용. 외부 배포 금지.
