---
name: s-skill-shiftee
version: 1.0.0
description: |
  시프티(Shiftee) 근태/휴가 조회 스킬. 첫 호출 시 shiftee CLI를 자동 다운로드하여 휴가 내역,
  출퇴근 기록, 스케줄, 누락 조회, 출퇴근 수정 요청 등 근태 관련 질문에 답한다.
  Use when asked about vacation, attendance, leave, schedule, clock-in/out, 휴가, 출퇴근, 근태, 스케줄, 누락, or anything HR/time-tracking related.
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
---

# 시프티 근태/휴가 스킬

`shiftee` CLI를 사용해서 근태, 휴가, 스케줄 관련 질문에 답한다. 첫 호출 시 GitHub에서 바이너리를 자동으로 캐시한다.

## CLI 부트스트랩 (매 호출 시 가장 먼저)

`shiftee` 바이너리가 캐시되어 있는지 확인하고, 없으면 GitHub에서 한 번 받아 `~/.cache/s-skill-shiftee/shiftee`에 저장한다. 이후 모든 명령은 `$SHIFTEE` 변수로 호출한다.

```bash
SHIFTEE=""
for p in \
  "$HOME/.cache/s-skill-shiftee/shiftee" \
  "$HOME/.claude/skills/s-skill-shiftee/shiftee" \
  "./.claude/skills/s-skill-shiftee/shiftee" \
  "$HOME/.agents/skills/s-skill-shiftee/shiftee"; do
  if [ -x "$p" ]; then SHIFTEE="$p"; break; fi
done

if [ -z "$SHIFTEE" ]; then
  mkdir -p "$HOME/.cache/s-skill-shiftee"
  SHIFTEE="$HOME/.cache/s-skill-shiftee/shiftee"
  curl -fsSL https://raw.githubusercontent.com/Salesmap-tech/s-skill/main/bin/shiftee \
       -o "$SHIFTEE" || { echo "shiftee 다운로드 실패 — 네트워크/방화벽 확인"; exit 1; }
  chmod +x "$SHIFTEE"
fi
```

스킬을 처음 사용하기 전에 한 번 로그인이 필요하다. 로그인 방식은 **이메일/비밀번호**다 (쿠키 붙여넣기 아님):

```bash
$SHIFTEE login
```

1. 이메일 입력
2. 비밀번호 입력 (`getpass` — 화면에 표시되지 않음)
3. CLI가 account 토큰 → employee_id → employee 토큰을 자동 발급한다. 여러 회사/직원에 소속된 경우 번호로 선택한다.

로그인은 대화형 입력(`input`/`getpass`)을 받으므로 Claude가 Bash 도구로 대신 입력할 수 없다. **사용자가 직접 실행하도록 안내한다** — 프롬프트에 `!$SHIFTEE login`을 입력하거나 터미널에서 직접 실행하면 된다.

토큰·이메일·계정 정보는 `~/.config/shiftee-cli/config.json` (0600)에 저장된다. **비밀번호는 저장하지 않는다** — 로그인 POST에만 일회성으로 쓰인다. account 토큰은 약 5년, employee 토큰은 약 1년 유효하고, employee 토큰이 만료돼 401이 나면 장수명 account 토큰으로 employee 토큰만 자동 재발급하므로 보통은 한 번만 로그인하면 된다. account 토큰까지 만료되면 `$SHIFTEE login`을 다시 실행한다.

## 사용 가능한 명령어

| 명령 | 설명 | 예시 |
|------|------|------|
| `$SHIFTEE me` | 내 정보 | |
| `$SHIFTEE today` | 오늘 출퇴근 현황 | |
| `$SHIFTEE attendance` | 이번 달 출퇴근 기록 | |
| `$SHIFTEE attendance -m 3` | 특정 월 출퇴근 기록 | 3월 기록 |
| `$SHIFTEE attendance -d 2026-04-01` | 특정 날짜 출퇴근 | |
| `$SHIFTEE schedule` | 이번 주 스케줄 | |
| `$SHIFTEE schedule --month` | 이번 달 스케줄 | |
| `$SHIFTEE leaves` | 올해 휴가 기록 | |
| `$SHIFTEE missing` | 출퇴근 누락 조회 | |
| `$SHIFTEE fix clock-in DATE TIME` | 출근 수정 요청 | `fix clock-in 2026-03-26 09:30` |
| `$SHIFTEE fix clock-out DATE TIME` | 퇴근 수정 요청 | `fix clock-out 2026-03-26 21:00` |
| `$SHIFTEE fix create DATE TIME --time2 TIME` | 출퇴근 생성 | `fix create 2026-03-26 09:30 --time2 21:00` |
| `$SHIFTEE raw METHOD PATH` | 원시 API 호출 | `raw GET /api/account` |

## 질문 유형별 라우팅

### "올해 휴가 언제 썼어?" / "휴가 내역"
```bash
$SHIFTEE leaves
```

### "오늘 출퇴근 했어?" / "오늘 근무"
```bash
$SHIFTEE today
```

### "이번 달 출퇴근 기록" / "3월 근태"
```bash
$SHIFTEE attendance          # 이번 달
$SHIFTEE attendance -m 3     # 특정 월
```

### "이번 주 스케줄" / "이번 달 스케줄"
```bash
$SHIFTEE schedule            # 이번 주
$SHIFTEE schedule --month    # 이번 달
```

### "출퇴근 누락된 거 있어?"
```bash
$SHIFTEE missing
```

### "출근/퇴근 수정해줘"
수정 요청 전 반드시 사용자에게 날짜와 시간을 확인한다.
```bash
$SHIFTEE fix clock-in 2026-04-15 09:30
$SHIFTEE fix clock-out 2026-04-15 18:00
$SHIFTEE fix create 2026-04-15 09:30 --time2 18:00 -n "사유"
```

## 응답 가이드

- CLI 출력을 그대로 붙여넣지 말고, 사용자가 이해하기 쉽게 요약해서 답한다.
- 휴가 기록은 날짜, 유형(연차/반차/병가 등), 사유를 표 형태로 정리한다.
- 출퇴근 기록도 날짜별로 깔끔하게 정리한다.
- **`fix` 명령은 실제 수정 요청을 보내므로, 실행 전 반드시 사용자 확인을 받는다.** `AskUserQuestion`으로 최종 승인 후 실행.
- "설정 파일이 없습니다" 에러가 나면 아직 로그인 전이다. 대화형 로그인이므로 사용자에게 `!$SHIFTEE login`을 직접 실행하라고 안내한다 (Claude가 비밀번호를 대신 입력할 수 없음).

$ARGUMENTS
