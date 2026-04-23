---
name: s-skill-linkedin-scrap
version: 1.0.0
description: |
  링크드인 포스트 스크랩 스킬. 키워드나 인물명으로 링크드인 포스트를 검색·수집·저장.
  Use when asked to "링크드인 스크랩", "linkedin scrap", "포스트 모아줘", "링크드인 검색",
  "linkedin search", or when the user wants to collect LinkedIn posts.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - WebSearch
  - WebFetch
  - AskUserQuestion
  # Skill(browse)는 선택적 의존성 (gstack이 설치된 경우에만 사용).
  # 미설치여도 이 스킬은 Jina Reader만으로 동작합니다. "3. /browse" 섹션 참고.
  - Skill(browse)
---

# 링크드인 스크랩 스킬

키워드·인물명으로 링크드인 포스트를 검색하고 수집하는 스킬입니다.

## 사용법

- `/s-skill-linkedin-scrap [키워드]` — 키워드로 포스트 검색·수집
- `/s-skill-linkedin-scrap [인물명]` — 특정 인물의 포스트 수집
- `/s-skill-linkedin-scrap [URL]` — 특정 포스트 URL 직접 스크랩

---

## 도구 3종 활용 전략

### 1. WebSearch — 검색 (발견 단계)

포스트를 **찾는 데** 사용합니다.

```
WebSearch: "site:linkedin.com/posts {키워드}"
WebSearch: "site:linkedin.com/in/{인물명} posts"
WebSearch: "linkedin {키워드} {인물명}"
```

- `site:linkedin.com/posts` 쿼리로 공개 포스트를 빠르게 검색
- 키워드 + 인물명 조합으로 타겟 검색
- 검색 결과에서 포스트 URL 목록 수집

### 2. Jina Reader — 읽기 (수집 단계)

포스트 내용을 **읽는 데** 사용합니다. WebFetch로 Jina Reader API 호출:

```
WebFetch: https://r.jina.ai/{linkedin_post_url}
```

- 링크드인 포스트를 깔끔한 마크다운으로 변환
- 로그인 없이 공개 포스트 본문 추출
- 빠르고 안정적인 텍스트 수집

### 3. /browse — 탐색 (보충 단계, **선택적**)

Jina Reader로 안 되는 경우 **보충**으로 사용합니다.

- Jina Reader가 빈 결과를 반환하거나 로그인 벽에 막힐 때
- 프로필 페이지에서 최신 포스트 목록을 훑어야 할 때
- 검색 결과가 부족해서 LinkedIn 내부 검색이 필요할 때

> ⚠️ **`/browse`는 gstack 플러그인에 포함된 스킬입니다.** 설치되어 있지 않으면 이 스킬은 Jina Reader만으로 동작합니다. `/browse` 없이도 대부분의 공개 포스트는 수집 가능합니다.

---

## 작업 흐름

### Step 1: 검색 의도 파악

사용자 입력을 분석합니다:
- **키워드** → 주제별 포스트 검색
- **인물명** → 해당 인물의 포스트 수집
- **URL** → 바로 Step 3으로

모호하면 한 가지만 짧게 질문합니다.

**기간 필터**: 기본적으로 **최근 1개월** 이내 포스트만 수집합니다. 사용자가 다른 기간을 지정하면 그에 따릅니다. 검색 쿼리에 `after:YYYY-MM-DD` 또는 연월 키워드를 포함하고, Jina Reader로 수집 후 날짜를 확인하여 기간 밖 포스트는 제외합니다.

### Step 2: 포스트 검색 (WebSearch)

```
site:linkedin.com/posts "{키워드}" {현재연도}
site:kr.linkedin.com/posts "{키워드}" {현재연도}
```

다양한 검색 변형을 병렬로 실행하여 URL을 최대한 많이 수집합니다. 최소 5개, 최대 15개 정도.

### Step 3: 포스트 본문 수집 (Jina Reader) — 본문 전문 필수

수집한 URL마다 Jina Reader로 **본문 전문**을 가져옵니다:

```
WebFetch → https://r.jina.ai/{url}
```

**핵심 원칙: 사용자가 링크드인에 직접 들어가지 않아도 되도록, 본문 전체를 빠짐없이 수집한다.**

- Jina Reader가 요약만 반환하면, prompt에 "포스트 본문 전문을 그대로 추출. 요약하지 말 것"을 명시합니다.
- Jina Reader 실패(503, 999 에러 등) 시 `/browse`가 **설치되어 있으면** 대체합니다.
  - `/browse` 유무 확인: `ls ~/.claude/plugins/*/skills/browse 2>/dev/null || ls ~/.claude/skills/browse 2>/dev/null` 또는 툴 호출 실패 여부로 판단.
  - `/browse`가 없으면 해당 URL은 건너뛰고 `[본문 수집 실패 — Jina Reader 차단]`으로 기록합니다.
- `/browse`로도 실패하면 URL만 기록하고 `[본문 수집 실패]` 표시합니다.

### Step 4: 포스트별 개별 파일로 저장

**각 포스트를 개별 .md 파일로 저장합니다.** 하나의 파일에 여러 포스트를 넣지 않습니다.

**디렉토리 구조**:
```
.claude/skills/s-skill-linkedin-scrap/collections/
  └── {YYYY-MM-DD}-{검색슬러그}/
      ├── _index.md          ← 검색 요약 (목차 역할)
      ├── 01-{포스트슬러그}.md
      ├── 02-{포스트슬러그}.md
      └── ...
```

**개별 포스트 파일 포맷** (`01-{슬러그}.md`):

```markdown
---
title: "{포스트 제목 또는 첫 줄 요약}"
author: "{이름}"
author_title: "{직함}"
date: "YYYY-MM-DD"
url: "{원본 링크}"
likes: N
comments: N
query: "{검색 키워드}"
collected_at: "YYYY-MM-DD"
---

# {포스트 제목 또는 첫 줄 요약}

## 작성자

**{이름}** — {직함}

## 본문

{포스트 전문 — 원문 그대로. 줄바꿈, 이모지, 해시태그 모두 보존}

## 왜 이 글이 인기가 많은가

{3~5줄 분석. 다음 관점에서:}
- **훅(Hook)**: 첫 1~2줄이 어떻게 관심을 끄는가
- **구조**: 글의 전개 방식 (리스트형, 스토리텔링, 비교 등)
- **공감 포인트**: 독자가 왜 좋아요를 누르는가 (실용성, 감정, 논쟁성 등)
- **타이밍**: 트렌드나 시의성과의 관계
- **CTA/공유 유도**: 댓글이나 공유를 유도하는 요소가 있는가

## 글쓰기 레퍼런스 메모

{이 글에서 내 글쓰기에 참고할 수 있는 포인트 1~2줄}
```

**인덱스 파일 포맷** (`_index.md`):

```markdown
---
query: "{검색 키워드}"
collected_at: "YYYY-MM-DD"
period: "{시작일} ~ {종료일}"
count: N
---

# {검색 주제} 스크랩

| # | 제목 | 작성자 | 좋아요 | 파일 |
|---|------|--------|--------|------|
| 1 | {제목} | {작성자} | {N} | [01-{슬러그}.md](./01-{슬러그}.md) |
| 2 | ... | ... | ... | ... |

## 전체 인사이트

{수집된 포스트들을 종합한 트렌드 분석 2~3줄}
```

### Step 5: 결과 요약

채팅에서 간단히 요약합니다:
- 검색 키워드/인물, 기간
- 수집된 포스트 수
- 주목할 만한 포스트 1~2개 하이라이트 + 왜 인기인지 한 줄
- 저장 디렉토리 경로

---

## 행동 규칙

1. **검색 → 읽기 → 저장** 순서를 지킵니다.
2. Jina Reader를 **우선** 사용하고, 실패 시 /browse로 보충합니다.
3. **본문은 전문을 그대로** 저장합니다. 임의로 요약하거나 편집하지 않습니다. 사용자가 링크드인에 안 들어가도 글 전체를 읽을 수 있어야 합니다.
4. **포스트별 개별 .md 파일**로 저장합니다. 하나의 파일에 여러 포스트를 넣지 않습니다.
5. **인기 분석 필수** — 각 포스트마다 "왜 이 글이 인기가 많은가" 섹션을 포함합니다.
6. **기간 필터** — 기본 최근 1개월. 사용자가 다른 기간을 지정하면 그에 따릅니다.
7. 중복 포스트는 URL 기준으로 제거합니다.
8. 수집 불가능한 포스트(비공개 등)는 URL만 기록하고 넘어갑니다.

$ARGUMENTS
