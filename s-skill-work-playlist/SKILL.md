---
name: s-skill-work-playlist
version: 1.0.0
description: |
  1시간 분량의 무작위 작업용 음악 플레이리스트를 생성하고 YouTube 플레이리스트 링크를 만들어주는 스킬.
  "플레이리스트", "노래 추천", "일할 때 들을 음악", "작업용 음악", "BGM", "뭐 들을까",
  "음악 틀어줘", "playlist", "music for work", "코딩할 때 음악" 등의 요청 시 반드시 사용한다.
  음악이나 노래에 관한 추천 요청이 들어오면 이 스킬을 사용한다.
allowed-tools:
  - WebSearch
---

# Work Playlist Generator

1시간 분량의 무작위 작업용 플레이리스트를 만들고, 한 번에 재생할 수 있는 YouTube 링크를 제공한다.

## 1. 곡 선정

12~15곡을 선정한다 (곡당 평균 4~5분, 합계 약 60분 목표).

### 무작위성 확보

곡 선정 전에, 현재 날짜·시간·요일을 확인하고 이를 seed 삼아 출발 장르와 분위기를 정한다. 예를 들어 수요일 오후라면 "Ethio-jazz로 시작해볼까" 같은 식으로 매번 다른 진입점을 잡는다.

다음 순서로 진행한다:
1. 아래 4개 권역에서 **각각 최소 2개 장르**를 먼저 고른다 (총 8개+ 장르).
2. 각 장르에서 1~2곡씩 뽑는다.
3. 한 장르가 연속 3곡 이상 나오지 않도록 순서를 섞는다.

| 권역 | 예시 (이 밖의 장르도 자유롭게 포함) |
|------|--------------------------------------|
| 서양 | Jazz, Ambient, Shoegaze, Krautrock, Yacht rock, Chamber pop, Dub, Tropicalia... |
| 아시아 | City pop, K-indie, J-jazz, Thai funk, Kayokyoku, Filipino OPM, Vietnamese pop... |
| 아프리카/중남미/중동 | Afrobeat, Amapiano, Ethio-jazz, MPB, Fado, Cumbia, Gnawa, Rai... |
| 크로스오버/기타 | Electro swing, Vaporwave, Folktronica, Nu-jazz, Library music, Psybient... |

위 예시는 출발점일 뿐이다. 목록에 없는 장르를 넣는 것을 적극 권장한다.

### 선정 기준

- **실존하는 곡만** 추천한다. 아티스트명과 곡명이 정확해야 한다. 확실하지 않으면 넣지 않는다.
- 작업 중 듣기 좋은 곡 — 과하게 시끄럽거나 산만한 곡은 피하되, "듣기 좋다"의 기준을 좁게 잡지 않는다. Amapiano나 Afrobeat도 작업용으로 훌륭하다.
- **유명한 곡 ≤ 40%, 덜 알려진 곡 ≥ 60%**. "이런 곡도 있었어?" 하는 발견의 즐거움을 준다.
- "작업용 음악" 하면 떠오르는 단골 아티스트(Bonobo, Nujabes, Tycho, Khruangbin, Nils Frahm, Explosions in the Sky 등)는 **최대 3곡**까지만 허용한다.

## 2. YouTube 영상 검색

각 곡의 YouTube 영상 ID를 찾아야 한다. **영상 ID를 절대 추측하거나 지어내지 않는다** — 반드시 WebSearch로 검색한다.

### 배치 검색 전략

곡을 3~4곡씩 묶어서 한 번의 WebSearch에 여러 곡을 포함한다. 예: `("아티스트A" "곡A" OR "아티스트B" "곡B") site:youtube.com`. 한 번에 안 나오는 곡만 개별 재검색한다.

### 검색 규칙

1. 검색 결과에서 YouTube URL을 찾고, `v=` 파라미터 뒤의 11자리 영상 ID를 추출한다.
2. 공식 오디오/뮤직비디오 또는 Topic 채널 영상을 우선으로 선택한다.
3. 찾을 수 없는 곡은 건너뛰고 같은 장르의 다른 곡으로 대체한다.
4. 최종 플레이리스트에 최소 10곡 이상이 포함되어야 한다.

## 3. 결과 출력

아래 형식을 따른다:

```
오늘의 작업용 플레이리스트 (약 XX분)

 1. 아티스트 - 곡명 (장르)
 2. 아티스트 - 곡명 (장르)
 3. ...
...

한 번에 듣기: https://www.youtube.com/watch_videos?video_ids=ID1,ID2,ID3,...

개별 링크:
 1. https://www.youtube.com/watch?v=ID1
 2. https://www.youtube.com/watch?v=ID2
 ...
```

간결하게 곡 목록과 링크만 출력한다. 각 곡에 대한 설명이나 감상 코멘트는 넣지 않는다.
"한 번에 듣기" 링크가 작동하지 않을 경우를 대비해 개별 링크도 항상 함께 제공한다.
