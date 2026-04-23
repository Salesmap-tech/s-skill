# s-skills

세일즈맵 사내 Claude Code 스킬 모음집.

## 포함된 스킬

| 스킬 | 설명 |
|------|------|
| [s-skill-setup](./s-skill-setup) | 설치 후 MCP·gh·Shiftee를 대화형으로 설정해주는 마법사 |
| [s-skill-teardown](./s-skill-teardown) | setup의 반대 — 항목별로 물어보면서 MCP·토큰·로그인을 선택 제거 |
| [s-skill-linkedin-scrap](./s-skill-linkedin-scrap) | 키워드·인물명으로 링크드인 포스트 검색·수집·저장 |
| [s-skill-work-log-scrap](./s-skill-work-log-scrap) | GitHub·Linear·Slack 활동을 종합한 활동 요약 리포트 생성 |
| [s-skill-slack](./s-skill-slack) | Slack MCP 래퍼 — 채널/DM 조회·검색·작성 |
| [s-skill-shiftee](./s-skill-shiftee) | 번들된 shiftee CLI로 근태·휴가·스케줄 조회 및 출퇴근 수정 |

## 설치

### 방법 1. `skills` CLI 사용 (권장)

아래 명령을 실행하면 **설치할 에이전트를 대화형으로 물어봅니다** (Claude Code만 고르거나, 여러 개 선택 가능). 설치 범위만 둘 중 하나로 고르세요.

**A. 전역 설치** — 어떤 프로젝트에서든 사용 가능 (`~/.claude/skills/`)

```bash
npx --yes skills add Salesmap-tech/s-skill -g
```

**B. 현재 프로젝트에만 설치** — 이 레포에서만 사용 (`./.claude/skills/`)

```bash
npx --yes skills add Salesmap-tech/s-skill
```

실행 흐름:
1. **설치할 스킬 선택** (스페이스로 토글, 엔터 확정)
2. **설치할 에이전트 선택** — Claude Code가 기본 체크돼 있음. 필요한 것만 남기고 엔터

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
chmod +x ~/.claude/skills/s-skill-shiftee/shiftee
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

### 3. Shiftee CLI 로그인 (선택)

`s-skill-shiftee` 스킬을 쓰려면 설치 후 한 번 로그인하면 됩니다.

```bash
~/.claude/skills/s-skill-shiftee/shiftee login
```

이메일/비밀번호를 입력하면 토큰이 `~/.config/shiftee-cli/config.json`(0600)에 저장됩니다. 이후 모든 `s-skill-shiftee` 호출이 자동 인증됩니다.

## 사용법

설치 후 Claude Code에서 슬래시 명령으로 호출합니다.

```
/s-skill-linkedin-scrap [키워드]
/s-skill-work-log-scrap [기간]
/s-skill-slack [자연어 요청]
/s-skill-shiftee [자연어 요청]
```

정리가 필요하면 `/s-skill-teardown` — 항목별로 물어보면서 선택적으로 제거합니다.

각 스킬 상세 사용법은 개별 `SKILL.md` 참조.

## 기여

새 스킬을 추가하려면:

1. 루트에 스킬 이름으로 폴더 생성 (예: `my-skill/`)
2. 폴더 안에 `SKILL.md` 작성 (frontmatter 포함)
3. 이 README의 스킬 테이블에 추가
4. PR

## 라이선스

사내 전용. 외부 배포 금지.
