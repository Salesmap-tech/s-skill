---
name: s-skill-teardown
description: |
  s-skills 정리 스킬. Slack 앱/토큰, MCP 설정(Slack/Linear/Notion), Shiftee 로그인,
  GitHub CLI 로그인, 설치된 스킬 파일 자체까지 하나씩 물어보면서 선택적으로 제거한다.
  setup의 반대편.
  Use when asked "셋업 지워", "정리", "teardown", "uninstall", "로그아웃", "토큰 폐기",
  "슬랙 봇 지워", "시프티 로그아웃", "스킬 지워", or after "이제 안 써".
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - AskUserQuestion
  - ToolSearch
---

# s-skills 정리 마법사

**`/s-skill-setup`의 반대.** 설정해둔 MCP 서버, 토큰, 로그인 상태를 **하나씩 물어보면서** 정리한다.

## 원칙

- **전부 한 번에 지우지 않는다**. 항목마다 "이거 지울까요?" 묻고 Yes 받은 것만 처리.
- **파괴적 작업 전 반드시 재확인**. 특히 토큰/설정 파일은 백업을 먼저 만든다.
- **외부 시스템의 삭제는 안내만**. Slack 앱 자체 삭제처럼 브라우저에서 해야 하는 건 링크와 클릭 순서를 알려준다.
- **한국어 존댓말**. "~할까요?/~했습니다" 체.
- **"스킵" 항상 가능**. 질문마다 건너뛸 수 있어야 한다.

---

## 실행 흐름

### 0단계. 시작 멘트

```
안녕하세요. s-skills 정리를 도와드릴게요. 🧹

설정해둔 것들을 하나씩 물어보면서 지울지 선택하게 해드립니다.
실수로 다 지워버리지 않도록 항목마다 확인 받을게요.
```

### 1단계. 저장 위치 확인

setup과 동일하게 `MCP_PATH` 결정:

- 옵션 A: **전역** — `~/.claude/.mcp.json`
- 옵션 B: **이 프로젝트만** — `./.mcp.json`
- 옵션 C: **둘 다 검사** — 두 경로에서 발견되는 모든 설정을 각각 확인

`AskUserQuestion`으로 선택.

### 2단계. 현재 상태 탐지

지우기 전에 뭐가 있는지 먼저 보여준다:

```bash
# 각 MCP_PATH에 대해:
[ -f "$MCP_PATH" ] && jq '.mcpServers | keys' "$MCP_PATH" 2>/dev/null

# Shiftee 로그인 상태
[ -f "$HOME/.config/shiftee-cli/config.json" ] && echo "shiftee logged in"

# GitHub CLI 로그인 상태
gh auth status 2>&1 | head -3

# 설치된 s-skill 파일들 (전역/프로젝트 양쪽)
ls -1 "$HOME/.claude/skills" 2>/dev/null | grep -E '^s-skill-'
ls -1 ".claude/skills" 2>/dev/null | grep -E '^s-skill-'
```

그리고 ToolSearch로 세션에 로드된 MCP 도구들도 확인:

```
ToolSearch "+slack"     → Slack MCP
ToolSearch "+linear"    → Linear MCP
ToolSearch "+notion"    → Notion MCP
```

결과를 한 번에 보여준다:

```
🔍 현재 설정된 항목
  Slack MCP     : ✅ (xoxp-토큰 저장됨)
  Linear MCP    : ✅
  Notion MCP    : ✅
  Shiftee CLI   : ✅ (로그인됨)
  GitHub CLI    : ✅ (jongbeomlee로 로그인)
  설치된 스킬   : s-skill-setup, s-skill-shiftee, s-skill-slack, ... (전역)
```

### 3단계. 인터뷰 + 제거 루프

순서: **Shiftee → Slack → Linear → Notion → GitHub → 스킬 파일**
(가장 로컬/단순 → 외부 연관 → 마지막에 스킬 파일 자체 철거)

없는 항목은 자동 스킵. 있는 항목만 "지울까요?" 질문.

---

## 제거 분기

### Shiftee CLI 로그아웃

**감지:** `~/.config/shiftee-cli/config.json` 존재 여부.

**Q.** `AskUserQuestion`: "Shiftee 로그인 정보(토큰)를 지울까요?"
- 옵션: **네, 지우기** / **아니요, 유지** / **뭐 하는 거예요?**

**"뭐 하는 거예요?"** 면 설명:
> `~/.config/shiftee-cli/config.json`에 저장된 account_token, employee_token을 삭제합니다.
> 다시 쓰려면 `shiftee login`을 실행해서 이메일/비밀번호로 재로그인하면 됩니다.
> Shiftee 계정 자체가 삭제되는 건 아닙니다.

**"네"** 면:
```bash
cp ~/.config/shiftee-cli/config.json ~/.config/shiftee-cli/config.json.bak
rm ~/.config/shiftee-cli/config.json
# 자동 다운로드된 캐시 바이너리도 정리
rm -rf ~/.cache/s-skill-shiftee
```
검증: `[ ! -f ~/.config/shiftee-cli/config.json ] && echo "삭제 완료"`

결과:
```
✅ Shiftee 로그아웃 완료 + 캐시 정리. 백업은 ~/.config/shiftee-cli/config.json.bak 에 남겨뒀어요.
```

---

### Slack MCP 제거 (2단계)

Slack은 **MCP 설정 제거 + Slack 앱 자체 삭제** 두 단계로 나눠 물어본다.

#### 3-1. MCP 설정에서 제거

**감지:** `$MCP_PATH`의 `.mcpServers.slack` 존재 여부.

**Q.** `AskUserQuestion`: "Slack MCP 설정을 `.mcp.json`에서 제거할까요?"
- 옵션: **네, 제거** / **아니요, 유지**

**"네"** 면:

1. 백업: `cp "$MCP_PATH" "$MCP_PATH.bak"`
2. `jq`로 slack 키 제거:
   ```bash
   jq 'del(.mcpServers.slack)' "$MCP_PATH" > "$MCP_PATH.tmp" && mv "$MCP_PATH.tmp" "$MCP_PATH"
   ```
3. 안내:
   ```
   ✅ .mcp.json에서 slack 항목 제거했습니다. Claude Code를 재시작해주세요.
   (백업: $MCP_PATH.bak)
   ```

#### 3-2. Slack 앱/토큰 폐기

**Q.** `AskUserQuestion`: "Slack 워크스페이스에서 `s-skills-mcp` 앱도 삭제할까요? (토큰이 완전히 무효화됩니다)"
- 옵션: **네, 브라우저에서 삭제** / **토큰만 revoke** / **아니요, 앱 유지**

**"네, 브라우저에서 삭제"** 면 수동 안내:
```
Slack 앱 자체를 완전히 삭제하려면:

👉 https://api.slack.com/apps 열기
👉 목록에서 이전에 만든 앱(예: `s-skills-mcp`) 클릭
👉 왼쪽 메뉴 맨 아래 'Basic Information' 페이지 최하단
👉 'Delete App' 버튼 클릭 → 확인

이렇게 하면 발급된 xoxp- 토큰도 영구히 무효화됩니다.
완료하셨나요?
```
→ `AskUserQuestion`(네/아직).

**"토큰만 revoke"** 면:
```
앱은 남기고 토큰만 폐기하려면:

👉 https://api.slack.com/apps 에서 해당 앱 → 'OAuth & Permissions'
👉 'User OAuth Token' 옆 'Revoke Token' 클릭

완료하셨나요?
```

#### 3-3. 검증

재시작 후 `ToolSearch "+slack"`으로 Slack MCP가 사라졌는지 확인.

---

### Linear MCP 제거

**감지:** `$MCP_PATH`의 `.mcpServers["linear-server"]` 존재 여부.

**Q.** `AskUserQuestion`: "Linear MCP 설정을 제거할까요?"

**"네"** 면:
```bash
jq 'del(.mcpServers["linear-server"])' "$MCP_PATH" > "$MCP_PATH.tmp" && mv "$MCP_PATH.tmp" "$MCP_PATH"
```

**추가 질문.** `AskUserQuestion`: "Linear 쪽에서 OAuth 연동 권한도 취소할까요?"
- **네**: 아래 안내
- **아니요**: 스킵

**"네"** 면:
```
👉 Linear 웹앱 → 오른쪽 상단 프로필 → Settings → API → OAuth applications
👉 'Claude' 또는 'Linear MCP' 항목의 'Revoke' 클릭

완료하셨나요?
```

---

### Notion MCP 제거

**감지:** `$MCP_PATH`의 `.mcpServers.notion` 존재 여부.

**Q.** `AskUserQuestion`: "Notion MCP 설정을 제거할까요?"

**"네"** 면:
```bash
jq 'del(.mcpServers.notion)' "$MCP_PATH" > "$MCP_PATH.tmp" && mv "$MCP_PATH.tmp" "$MCP_PATH"
```

**추가 질문.** Notion 워크스페이스에서 권한 철회:
```
👉 Notion → Settings & members → Connections (또는 My connections)
👉 'Claude' 연결 찾아서 'Remove' 클릭

완료하셨나요?
```

---

### GitHub CLI 로그아웃

**감지:** `gh auth status` 0 exit인지.

**Q.** `AskUserQuestion`: "GitHub CLI에서 로그아웃할까요? (`gh auth logout`)"
- 옵션: **네, 로그아웃** / **아니요, 유지**

**"네"** 면 `gh auth logout` 실행. 대화형 입력이 필요하면 사용자에게 터미널에서 직접 실행하도록 안내:
```
터미널에서 직접 실행해주세요:

gh auth logout

호스트 선택(github.com) → 확인. 완료하셨나요?
```

---

### 스킬 파일 제거 (s-skills 자체)

여기까지는 **토큰·설정**만 지웠고, 스킬 **프로그램 파일**(`~/.claude/skills/s-skill-*` 또는 `./.claude/skills/s-skill-*`)은 아직 그대로 남아 있음. 이 단계에서 파일까지 철거한다.

**감지:** 전역/프로젝트 양쪽에서 `s-skill-*` 디렉토리 존재 여부.

```bash
GLOBAL_SKILLS=$(ls -1 "$HOME/.claude/skills" 2>/dev/null | grep -E '^s-skill-' | tr '\n' ' ')
LOCAL_SKILLS=$(ls -1 ".claude/skills" 2>/dev/null | grep -E '^s-skill-' | tr '\n' ' ')
```

둘 다 없으면 이 섹션 전체 스킵.

**Q.** `AskUserQuestion`: "설치된 s-skill 파일을 제거할까요? (재설치 전까지는 슬래시 명령이 사라집니다)"
- 옵션:
  - **네, 전역에서 제거** (`~/.claude/skills/`)
  - **네, 프로젝트에서 제거** (`./.claude/skills/`)
  - **네, 둘 다 제거**
  - **아니요, 유지**
  - **뭐 하는 거예요?**

**"뭐 하는 거예요?"** 면 설명:
> `npx skills remove`로 설치된 스킬 파일(심볼릭 링크/복사본)을 지웁니다. 이건 로컬 파일만 지우는 거라 GitHub의 원본 레포와는 무관합니다. 나중에 다시 쓰려면 `npx skills add Salesmap-tech/s-skill -s '*' -g` 한 번이면 복원됩니다.

**"네, ..."** 중 하나를 고르면:

1. 해당 scope 플래그(`-g` 또는 없음)로 `npx skills remove`를 대화형으로 실행하도록 **사용자에게 터미널 명령 안내**한다. (Claude Code Bash는 TTY가 아니어서 `skills remove`의 체크박스 UI가 제대로 안 돌 수 있음.)

```
터미널에서 직접 실행해주세요:

# 전역 스킬 제거 (대화형 선택)
npx skills remove -g

# 프로젝트 스킬 제거 (대화형 선택)
npx skills remove
```

2. 목록을 보여주면서 원치 않는 것만 해제하고 엔터. `s-skill-setup`은 마지막까지 남겨두면 나중에 재설치할 때 편하다는 팁을 같이 안내.

3. 완료 확인 `AskUserQuestion`: "제거 완료하셨나요?" (네 / 아직 / 실패했어요)

4. 검증:
```bash
ls -1 "$HOME/.claude/skills" 2>/dev/null | grep -E '^s-skill-' || echo "전역: 없음"
ls -1 ".claude/skills" 2>/dev/null | grep -E '^s-skill-' || echo "프로젝트: 없음"
```

**주의:** Shiftee CLI 바이너리는 이제 `~/.cache/s-skill-shiftee/shiftee`에 자동 캐시되며, 위 "Shiftee 로그아웃" 단계에서 해당 캐시도 같이 지워진다. 그래서 **Shiftee 로그아웃 단계를 먼저 처리**한 뒤에 이 단계가 와야 한다 — 이미 순서가 그렇게 돼 있음.

---

## 4단계. 최종 리포트

```
🧹 정리 완료

- ✅ Shiftee 로그아웃
- ✅ Slack MCP 제거 (앱은 브라우저에서 수동 삭제 대기)
- ⏭️ Linear MCP (유지)
- ⏭️ Notion MCP (유지)
- ⏭️ GitHub CLI (유지)
- ✅ 스킬 파일 제거 (전역 4개, 프로젝트 유지)

백업 파일들:
- ~/.config/shiftee-cli/config.json.bak
- $MCP_PATH.bak

다시 세팅하고 싶으시면:
  npx skills add Salesmap-tech/s-skill -s '*' -g  # 스킬 재설치
  /s-skill-setup                                  # 토큰/MCP 재설정
```

---

## `.mcp.json` 편집 규칙 (구현 노트)

- **쓰기 전 반드시 `$MCP_PATH.bak`으로 백업**.
- `jq del(...)`은 in-place 동작이 아니므로 tmp 파일로 쓴 뒤 `mv`.
- 삭제 후 `.mcpServers`가 비면 빈 객체로 유지 (`{"mcpServers": {}}`). 파일 자체를 지우지 않는다 — 다른 MCP 설정이 있을 수 있음.
- `jq` 없으면 `brew install jq` 안내 후 중단.

## 행동 규칙

1. **한 번에 하나씩 질문**. 체크리스트 여러 개 동시 금지.
2. **삭제 전 백업**. `.mcp.json`, Shiftee config 둘 다 `.bak` 남긴다.
3. **외부 시스템 삭제는 안내 + 확인**. 대신 눌러주지 않는다.
4. **"건너뛰기" 옵션 항상 제공**.
5. **파일에 쓴 토큰은 출력에 echo 금지**. 백업 파일 이름만 언급.
6. **에러 시 원상 복구 안내**. `mv $MCP_PATH.bak $MCP_PATH` 한 줄로 되돌릴 수 있다는 걸 알려준다.

$ARGUMENTS
