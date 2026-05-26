---
name: s-skill-setup
description: |
  s-skills 설치 후 MCP 서버(Linear/Slack/Notion)와 GitHub CLI를 대화형으로 설정하는 스킬.
  사용 중인 도구만 골라 설정하고, 이미 돼 있으면 스킵, 안 돼 있으면 단계별 안내·검증까지 전부 끌고 간다.
  Use when asked "셋업", "setup", "처음 설정", "s-skills 설치했어요", or after installing s-skills.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - AskUserQuestion
  - ToolSearch
---

# s-skills 설정 마법사

**s-skills 스킬을 처음 쓰기 전에 필요한 MCP 서버와 GitHub CLI를 대화로 설정해주는 스킬.** 이미 설정된 건 건너뛰고, 안 된 것만 단계별로 안내·검증한다.

## 원칙

- **비개발자 기준**으로 친절하게. `.mcp.json` 편집이나 토큰 발급을 처음 해보는 사람도 완주할 수 있어야 한다.
- **묻기 전에 먼저 확인**한다. 이미 되어있으면 "이미 됐네요 ✅" 한 줄로 스킵.
- **절대 자동으로 기존 설정을 덮어쓰지 않는다**. 충돌 시 사용자에게 묻는다.
- **존댓말 사용**. "~했습니다/~할까요?" 체.
- **step-by-step, 한 번에 하나씩**. 체크리스트 여러 개 한꺼번에 들이밀지 말 것.

---

## 실행 흐름

### 0단계. 시작 멘트

```
안녕하세요! s-skills 설정을 도와드릴게요. 🛠

몇 가지 질문 드리면서 필요한 것만 골라서 설정해드릴게요.
이미 되어 있는 건 자동으로 건너뜁니다.
```

### 1단계. 저장 위치 선택

설정 파일을 어디에 쓸지 먼저 결정. `AskUserQuestion`:

- 옵션 A: **이 프로젝트만 (추천)** — 현재 디렉토리 `./.mcp.json`. 팀이 이 프로젝트를 쓸 때 동일 설정을 공유할 수 있고, 다른 프로젝트에 영향이 없음.
- 옵션 B: **전역** — `~/.claude/.mcp.json`. 어느 프로젝트에서든 동작.

선택 결과를 `$MCP_PATH` 변수처럼 이후 모든 단계에서 사용.

```bash
# 프로젝트 선택 시 (기본 추천)
MCP_PATH="./.mcp.json"
# 전역 선택 시
MCP_PATH="$HOME/.claude/.mcp.json"
```

### 2단계. 현재 상태 탐지

질문하기 전에 상태를 훑어본다:

```bash
# gh 설치/로그인
which gh >/dev/null 2>&1 && gh auth status 2>&1 | head -3

# 대상 .mcp.json 존재 및 서버 목록
[ -f "$MCP_PATH" ] && cat "$MCP_PATH" | jq '.mcpServers | keys' 2>/dev/null

# Shiftee CLI 바이너리 탐색 (캐시 → 전역 스킬 → 프로젝트 스킬 → Universal)
SHIFTEE_BIN=""
for p in \
  "$HOME/.cache/s-skill-shiftee/shiftee" \
  "$HOME/.claude/skills/s-skill-shiftee/shiftee" \
  "./.claude/skills/s-skill-shiftee/shiftee" \
  "$HOME/.agents/skills/s-skill-shiftee/shiftee"; do
  if [ -x "$p" ]; then SHIFTEE_BIN="$p"; break; fi
done
[ -n "$SHIFTEE_BIN" ] && echo "shiftee found at $SHIFTEE_BIN"
[ -f "$HOME/.config/shiftee-cli/config.json" ] && echo "shiftee logged in"
```

그리고 현재 세션에서 MCP 도구가 로드돼 있는지 확인:

```
ToolSearch "+linear"    → Linear MCP 도구 있는지
ToolSearch "+slack"     → Slack MCP 도구 있는지
ToolSearch "+notion"    → Notion MCP 도구 있는지
```

탐지 결과를 한눈에 보여준다:

```
📋 현재 상태
  GitHub CLI   : ✅ (jongbeomlee로 로그인됨) / ❌ (미설치 또는 로그인 안 됨)
  Linear MCP   : ✅ 연결됨 / ❌ 미설정
  Slack MCP    : ✅ 연결됨 / ❌ 미설정
  Notion MCP   : ✅ 연결됨 / ❌ 미설정
  Shiftee CLI  : ✅ 로그인됨 / ⚠️ 번들은 있지만 로그인 안 됨 / ❌ 바이너리 없음 (자동 다운로드 시도 예정)
```

### 3단계. 인터뷰 + 설치 루프

상태가 ❌인 항목에 대해서만 "쓰시나요?" 묻고, ✅인 건 그냥 스킵.

순서: **GitHub → Linear → Slack → Notion → Shiftee**

각 항목에 대해 아래의 "설치 분기"를 수행.

---

## 설치 분기

### GitHub CLI 분기

**Q.** `AskUserQuestion`: "코드/이슈 관리를 위해 GitHub를 쓰시나요?"

**Y면:**

1. `which gh` 체크
2. 없으면:
   ```
   GitHub CLI가 없네요. 터미널에서 다음 명령어를 실행해주세요:

   macOS:   brew install gh
   Windows: winget install --id GitHub.cli
   Linux:   https://github.com/cli/cli/blob/trunk/docs/install_linux.md 참고

   설치 끝나셨나요?
   ```
   → `AskUserQuestion`(네/아직). 네면 `which gh` 재검증.

3. `gh auth status` 체크
4. 로그인 안 돼있으면:
   ```
   이제 로그인할게요. 터미널에 이거 치시고, 기본값으로 엔터 엔터 쭉 누르시면 됩니다:

   gh auth login

   브라우저가 열리고 one-time code 입력하라고 나와요. 완료하셨나요?
   ```
   → `AskUserQuestion`(네/문제있음). 네면 `gh auth status` 재검증.

5. 성공: "✅ GitHub 로그인 완료: @username"

---

### Linear MCP 분기

**Q.** `AskUserQuestion`: "프로젝트 관리/이슈 트래킹에 Linear를 쓰시나요?"

**Y면:**

1. `$MCP_PATH`에 다음 블록을 merge:
   ```json
   {
     "mcpServers": {
       "linear-server": {
         "type": "http",
         "url": "https://mcp.linear.app/mcp"
       }
     }
   }
   ```
2. 병합 규칙: 파일이 없으면 새로 생성. 있으면 `jq`로 `mcpServers["linear-server"]` 추가 (기존 키 있으면 "덮어쓸까요?" 물어봄).

3. 안내:
   ```
   ✅ Linear 설정 추가 완료!

   Claude Code를 한 번 재시작해주세요 (Cmd+Q 후 다시 열기).
   재시작 후 Linear에 처음 접근할 때 브라우저가 뜨면 Linear 계정으로 로그인해주세요.
   ```

4. 재시작 여부 확인 → `ToolSearch "+linear"`로 검증.

---

### Slack MCP 분기 (가장 복잡)

**Q.** `AskUserQuestion`: "팀 커뮤니케이션에 Slack을 쓰시나요?"

**Y면:**

도입부:
```
Slack은 토큰 발급이 필요해서 5분 정도 걸려요.
브라우저에서 Slack 앱 하나 만들어주셔야 하는데, 제가 단계별로 안내할게요.
준비되셨으면 시작할게요!
```

**단계 1** — 앱 생성:
```
👉 https://api.slack.com/apps 열어주세요.
👉 'Create New App' → 'From scratch' 클릭
👉 App Name: `s-skills-mcp` (아무거나 OK)
👉 Pick a workspace: 회사 워크스페이스 선택
👉 'Create App' 클릭

생성 완료하셨어요?
```
→ `AskUserQuestion`(네/안됨). 안됨이면 에러/스크린샷 요청.

**단계 2** — 스코프 설정:
```
👉 왼쪽 메뉴에서 'OAuth & Permissions' 클릭
👉 페이지 하단의 'User Token Scopes' 섹션으로 스크롤
👉 'Add an OAuth Scope' 버튼을 눌러서 아래 스코프를 **하나씩 추가**해주세요

읽기 계열:
  channels:history, channels:read, groups:history, groups:read,
  im:history, im:read, mpim:history, mpim:read, search:read,
  users:read, users:read.email, usergroups:read, files:read,
  reactions:read, team:read, emoji:read, links:read

쓰기 계열:
  chat:write, files:write, reactions:write, usergroups:write, links:write

(많아 보이지만 미리 넓게 잡아두면 나중에 스킬 추가돼도 재발급할 필요 없어요.)

다 추가하셨어요?
```
→ `AskUserQuestion`(네/도움 필요). 도움 필요면 "어떤 스코프에서 막히셨어요?" 물어봄.

**단계 3** — 워크스페이스에 설치:
```
👉 같은 페이지 맨 위로 스크롤해서 'Install to Workspace' 클릭
👉 브라우저 새 탭에서 권한 승인 화면 → 'Allow' 클릭

(주의: 워크스페이스에 따라 관리자 승인이 필요할 수 있어요.
 "요청을 보냈습니다"가 뜨면 관리자한테 승인 요청이 간 거예요. 승인받으시면 계속 진행.)

설치 완료하셨어요?
```
→ `AskUserQuestion`(네/관리자 승인 대기/에러).

**단계 4** — 토큰 복사:
```
👉 설치 후 'OAuth & Permissions' 페이지로 돌아가면
👉 상단에 'User OAuth Token' 이라는 필드가 생겨있어요
👉 `xoxp-` 로 시작하는 긴 토큰을 복사해서 여기에 붙여넣어주세요

(이 토큰은 당신의 Slack 계정처럼 동작하니까 깃에 올리거나 공유하지 마세요.)
```
→ `AskUserQuestion` 또는 사용자가 토큰 paste하게 유도. 받은 토큰이 `xoxp-`로 시작하는지 validation.

**단계 5** — 주입 + 검증:

`$MCP_PATH`에 merge:
```json
{
  "mcpServers": {
    "slack": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "slack-mcp-server"],
      "env": {
        "SLACK_MCP_XOXP_TOKEN": "xoxp-...사용자가_준_값...",
        "SLACK_MCP_ADD_MESSAGE_TOOL": "true"
      }
    }
  }
}
```

안내:
```
✅ Slack 토큰 저장 완료!

Claude Code를 재시작해주세요. 재시작 후에 Slack MCP가 도구 목록에 나타나면 성공이에요.
```
→ 재시작 확인 후 `ToolSearch "+slack"` 검증.

**흔한 실패**:
- "403 missing_scope" → 누락된 스코프 재추가 후 토큰 재발급 필요
- 관리자 승인 대기 → 승인 받을 때까지 pause
- 토큰이 `xoxb-`로 시작 → Bot token임, `xoxp-` (User) 토큰을 써야 함

---

### Notion MCP 분기

**Q.** `AskUserQuestion`: "문서 관리에 Notion을 쓰시나요?"

**Y면:**

1. `$MCP_PATH`에 merge:
   ```json
   {
     "mcpServers": {
       "notion": {
         "type": "http",
         "url": "https://mcp.notion.com/mcp"
       }
     }
   }
   ```

2. 안내:
   ```
   ✅ Notion 설정 추가 완료!

   Claude Code를 재시작해주세요.
   재시작 후 Notion MCP에 처음 접근할 때 브라우저가 뜨면 Notion 계정으로 로그인하고
   'Allow access' 눌러주세요. 어떤 페이지에 접근 권한을 줄지 선택하는 화면이 나옵니다.
   ```

3. 재시작 확인 → `ToolSearch "+notion"` 검증.

---

### Shiftee CLI 분기

**Q.** `AskUserQuestion`: "근태/휴가 조회에 Shiftee를 쓰시나요?"

**Y면:**

1. 바이너리 확보 (4곳 탐색 → 없으면 GitHub에서 자동 다운로드):
   ```bash
   SHIFTEE_BIN=""
   for p in \
     "$HOME/.cache/s-skill-shiftee/shiftee" \
     "$HOME/.claude/skills/s-skill-shiftee/shiftee" \
     "./.claude/skills/s-skill-shiftee/shiftee" \
     "$HOME/.agents/skills/s-skill-shiftee/shiftee"; do
     if [ -x "$p" ]; then SHIFTEE_BIN="$p"; break; fi
   done

   if [ -z "$SHIFTEE_BIN" ]; then
     mkdir -p "$HOME/.cache/s-skill-shiftee"
     SHIFTEE_BIN="$HOME/.cache/s-skill-shiftee/shiftee"
     curl -fsSL https://raw.githubusercontent.com/Salesmap-tech/s-skill/main/bin/shiftee \
          -o "$SHIFTEE_BIN" && chmod +x "$SHIFTEE_BIN"
   fi
   ```
   `curl`까지 실패한 경우(오프라인/방화벽):
   ```
   shiftee 바이너리를 받지 못했어요. 네트워크를 확인하거나 아래 수동 명령을 써주세요:

   git clone https://github.com/Salesmap-tech/s-skill.git /tmp/s-skill
   mkdir -p ~/.cache/s-skill-shiftee
   cp /tmp/s-skill/bin/shiftee ~/.cache/s-skill-shiftee/shiftee
   chmod +x ~/.cache/s-skill-shiftee/shiftee
   ```
   → 이 분기 스킵.

2. 로그인 상태 확인:
   ```bash
   [ -f "$HOME/.config/shiftee-cli/config.json" ]
   ```
   이미 있으면 "✅ Shiftee 이미 로그인됨" 한 줄로 스킵.

3. 로그인 안내 (**이메일/비밀번호** 방식. 대화형 입력이라 Claude가 대신 칠 수 없으니 사용자가 직접 실행):
   ```
   Shiftee 로그인을 진행할게요. 아래 명령을 터미널에서 직접 실행해주세요:

   $SHIFTEE_BIN login

   1. 이메일 입력
   2. 비밀번호 입력 (화면에는 안 보입니다)
   → CLI가 토큰과 직원 정보를 자동으로 받아옵니다. 여러 회사에 소속돼 있으면 번호로 골라주세요.

   토큰·계정 정보는 ~/.config/shiftee-cli/config.json (0600)에 저장됩니다. 완료하셨나요?
   ```
   → `AskUserQuestion`(네/문제있음). 네면 `$SHIFTEE_BIN me`로 검증.

4. 검증 실패 시 흔한 원인:
   - 이메일/비밀번호 오타 → shiftee.io에 같은 계정으로 로그인되는지 확인
   - 토큰 만료 → employee 토큰은 account 토큰으로 자동 갱신됨. account 토큰까지 만료면 `$SHIFTEE_BIN login` 재실행
   - Python 3 없음 → `python3 --version`으로 확인, 없으면 `brew install python` 안내

---

## 4단계. 최종 검증 (live read-only 호출)

모든 분기가 끝나면, 실제로 동작하는지 한 번씩 ping:

| 항목 | 검증 호출 |
|------|-----------|
| gh | `gh api user --jq .login` |
| Linear | `mcp__linear-server__list_projects` (limit 1) |
| Slack | `mcp__slack__channels_list` (limit 1) |
| Notion | `mcp__notion__...search` (query "test") |
| Shiftee | `$SHIFTEE_BIN me` (1단계에서 탐지한 경로) |

하나라도 실패하면:
- 에러 메시지 출력
- 해당 분기의 "흔한 실패" 섹션 참고해서 구체적인 재시도 안내

---

## 5단계. 마무리 리포트

모두 통과하면:

```
🎉 s-skills 세팅 완료!

설정된 항목:
- ✅ GitHub CLI (@username)
- ✅ Linear MCP
- ✅ Slack MCP
- ✅ Notion MCP
- ✅ Shiftee CLI

이제 아래 스킬들을 쓸 수 있어요:
- /s-skill-linkedin-scrap [키워드]   — 링크드인 포스트 수집
- /s-skill-work-log-scrap [기간]      — 내가 한 일 요약
- /s-skill-slack [요청]               — 슬랙 조회·작성
- /s-skill-shiftee [요청]             — 근태·휴가 조회

뭐 도와드릴까요?
```

일부만 설정한 경우:
```
🎉 부분 세팅 완료!

- ✅ GitHub CLI
- ✅ Linear MCP
- ⏭️ Slack MCP (스킵)
- ⏭️ Notion MCP (스킵)
- ⏭️ Shiftee (스킵)

나중에 추가하고 싶으시면 다시 /s-skill-setup 실행하시면 됩니다.
```

---

## `.mcp.json` 병합 규칙 (구현 노트)

사용할 스크립트 패턴 (Bash + jq):

```bash
# $MCP_PATH 없으면 생성
if [ ! -f "$MCP_PATH" ]; then
  echo '{"mcpServers": {}}' > "$MCP_PATH"
fi

# 서버 추가 (기존 있으면 덮어쓸지 먼저 묻고)
EXISTING=$(jq ".mcpServers[\"$SERVER_NAME\"] // empty" "$MCP_PATH")
if [ -n "$EXISTING" ]; then
  # AskUserQuestion: "기존 $SERVER_NAME 설정이 있네요. 덮어쓸까요?"
  # Y: 아래로 진행. N: skip.
  :
fi

# jq로 merge (in-place 아니니 temp 파일 쓰기)
jq --argjson new "$NEW_BLOCK" ".mcpServers[\"$SERVER_NAME\"] = \$new" "$MCP_PATH" > "$MCP_PATH.tmp" \
  && mv "$MCP_PATH.tmp" "$MCP_PATH"
```

- `jq`가 없으면 `brew install jq` 먼저 안내.
- 쓰기 전 원본을 `$MCP_PATH.bak`으로 백업.

---

## 일반 행동 규칙

1. **묻기 전에 체크**. 이미 되어있으면 질문하지 않는다.
2. **한 번에 한 질문**. `AskUserQuestion`은 2~4개 옵션짜리 하나씩.
3. **실패 시 친절 모드로**. 에러 메시지 그대로 보여주고, 해석해주고, 다음 액션 제안.
4. **파괴적 작업 금지**. 기존 `.mcp.json`을 덮어쓸 때는 반드시 백업 + 재확인.
5. **비밀키 노출 주의**. 토큰은 파일에 쓰되, 출력에는 절대 echo하지 않는다.
6. **"스킵" 옵션 항상 제공**. 질문마다 "나중에 설정"이 가능해야 한다.

$ARGUMENTS
