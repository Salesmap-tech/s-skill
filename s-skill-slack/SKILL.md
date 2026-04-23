---
name: s-skill-slack
version: 1.0.0
description: |
  슬랙 조회/작성 스킬. Slack MCP를 사용해서 채널 메시지 검색, 스레드 읽기, 메시지 작성을 대신 해준다.
  Use when asked to "슬랙 찾아줘", "슬랙 보내줘", "slack 메시지", "슬랙 검색",
  "최근 슬랙", "슬랙 디엠", "slack search", or anything involving Slack reads/writes.
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
  - ToolSearch
  - mcp__slack__channels_list
  - mcp__slack__conversations_history
  - mcp__slack__conversations_replies
  - mcp__slack__conversations_search_messages
  - mcp__slack__conversations_add_message
  - mcp__slack__conversations_mark
  - mcp__slack__conversations_unreads
  - mcp__slack__users_search
---

# 슬랙 스킬

Slack MCP를 통해 메시지를 **찾거나 써주는** 스킬. 사용자가 슬랙 앱을 열지 않아도 되도록 조회/작성을 대신 수행한다.

## 사용법

- `/s-skill-slack [질문]` — 자연어로 요청하면 의도를 파악해서 조회·작성
- 예시:
  - `/s-skill-slack #general 최근 이슈 뭐 있었어?`
  - `/s-skill-slack 지난주 엔지니어링 채널에서 릴리즈 관련 메시지`
  - `/s-skill-slack 김철수한테 "회의 5분 늦는다" 보내줘`
  - `/s-skill-slack 안 읽은 DM 알려줘`

---

## 의도 파악

사용자 입력을 먼저 **읽기(read)** / **쓰기(write)** 로 분류한다.

### 읽기 예시
- "~에서 ~찾아줘", "~에 뭐 올라왔어", "최근 메시지", "안 읽은 것"
- "~한테 온 DM", "스레드 내용", "어제 ~채널"

→ `conversations_search_messages` / `conversations_history` / `conversations_replies` / `conversations_unreads` 사용.

### 쓰기 예시
- "~한테 보내줘", "~채널에 써줘", "~라고 전달", "답장해줘"

→ **반드시 실행 전 본문·대상·채널을 사용자에게 재확인**. `conversations_add_message` 사용.

모호하면 `AskUserQuestion`으로 한 번만 짧게 물어본다.

---

## 작업 흐름

### 1. Slack MCP 연결 확인

`ToolSearch "+slack"`으로 도구가 로드됐는지 확인. 없으면:

> ⚠️ **Slack 연동 안 됨** — `/s-skill-setup`을 실행해서 Slack MCP를 먼저 설정해주세요.

### 2. 대상(채널/유저) 해석

- 사용자가 `#채널명` 또는 `@이름`으로 지정하면 그대로 사용.
- 이름만 있으면 `users_search` 또는 `channels_list`로 ID 해석. 후보 여러 개면 `AskUserQuestion`.

### 3. 기간 파싱 (읽기)

| 입력 | 의미 |
|------|------|
| (없음) / 최근 | 최근 24시간 |
| 오늘 | 오늘 0시 ~ 지금 |
| 어제 | 어제 0시 ~ 24시 |
| 이번주 / this week | 이번 주 월요일 ~ 지금 |
| 지난주 / last week | 지난 주 월요일 ~ 일요일 |
| Nd (예: 3d, 7d) | 최근 N일 |
| YYYY-MM-DD..YYYY-MM-DD | 특정 기간 |

### 4. 조회 실행

- **채널 히스토리**: `conversations_history` (channel_id, oldest, latest)
- **키워드 검색**: `conversations_search_messages` (query 문자열. `in:#channel from:@user after:YYYY-MM-DD` 연산자 활용)
- **스레드 조회**: 검색 결과에서 thread_ts 발견 시 `conversations_replies`
- **안 읽은 것**: `conversations_unreads`

### 5. 쓰기 실행

쓰기 요청은 반드시 아래 순서를 지킨다:

1. 본문 초안을 채팅에 보여준다:
   ```
   📝 보낼 메시지:
   to: #{channel} (또는 @{user})
   > {본문}

   이대로 보낼까요?
   ```
2. `AskUserQuestion`: **네 / 수정 / 취소**.
3. "네"일 때만 `conversations_add_message` 실행.
4. 성공하면 permalink 또는 ts를 회신.

---

## 출력 형식

### 읽기 결과
```
🔎 {채널 또는 검색어} — {기간}

- **{시간}** @{작성자}: {메시지 요약 또는 본문}
  🔗 {permalink}
- ...

## 요약
{핵심 논의 2~3줄}
```

### 쓰기 결과
```
✅ 전송 완료 — #{channel}
🔗 {permalink}
```

---

## 행동 규칙

1. **쓰기 전 반드시 확인**. 사용자가 "보내줘"라고 해도 본문·대상·채널을 보여주고 재승인을 받은 뒤에 전송.
2. **DM과 개인 정보 주의**. 민감한 내용은 읽어와도 채팅에 그대로 긴 인용을 피하고 요약 우선.
3. **대량 조회 방지**. 기본 limit 20~50. 사용자가 "전부"라고 하면 페이지네이션.
4. **한국어 존댓말**. "~했습니다/~할까요?" 체.
5. **권한 에러 시 친절 안내**. `missing_scope` 등은 `/s-skill-setup`으로 재설정 유도.

$ARGUMENTS
