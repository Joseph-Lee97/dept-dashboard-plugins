---
name: dept-song-checkup
description: DEPT 곡 하나를 점검할 때 사용. "○○ 요즘 어때?", "이 곡 점검해 줘", "○○ 스트리밍·유튜브·광고 한 번에", "신곡 반응 어때?" 같은 요청. 플랫폼별 재생·스포티파이 공식 분석·유튜브·플레이리스트·차트·광고·집행 제안을 묶는다.
---

# 곡 하나 점검

> **먼저 보고 일하기:** `data_catalog`가 있으면 먼저 불러(해당 recipe·데이터셋 cautions·도구 인자 확인) 쓸 데이터·도구와 이유를 한두 줄로 말한 뒤 조회한다. 숫자에는 `evidence`의 as_of·대시보드 링크를 각주로 붙인다. 자세한 규칙은 `dept-dashboard-data` 0·6절.

## 1. 곡 찾기
- `list_artists_songs` {title_query:"<제목 일부>"} → 제목·발매일 확인. 여러 개가 맞으면(어쿠스틱·언어 버전) 사용자가 말한 제목과 똑같은 원곡으로 하고 "다른 버전: …" 한 줄.
- 없으면 띄어쓰기·영문/한글을 바꿔 한 번 더 찾고, 그래도 없으면 비슷한 제목 후보를 보여 준다.

## 2. 데이터 모으기
**A. `analyze_song`이 있으면 한 번:** {query:"<제목>"} (기간 말이 있으면 compare·from·to). 쓰는 칸: `headlines`, `meta`, `dsp`, `spotify_for_artists`, `youtube`, `ads`, `ad_advice`, `cn_chart`, `candidates`.

**B. 없으면:** (기간 기본 최근 7일 vs 직전 7일 — 이번·직전 두 번씩)
1. 곡 정보: `get_albums_releases` {mode:"songs", query:"<제목>"} — 앨범·발매일·발매 형태.
2. 플랫폼별: `get_streaming_trend` {scope:"song", title_query:"<제목>", from, to, granularity:"total"} — 행마다 `label`·`kind`를 보고 누적값은 `increase`, 28일 합은 value 그대로. 플레이리스트 수는 `category:"플레이리스트"`로.
3. 스포티파이 공식 분석: `get_track_daily` {source:"spotify", title_query:"<제목>", from, to, granularity:"week"} — streams·saves·playlist_adds.
4. 유튜브: `get_track_daily` {source:"youtube", title_query:"<제목>", from, to, granularity:"week"} — 영상별 views.
5. 광고: `get_ad_performance` {group_by:"song", from, to} 에서 그 곡 줄 · 캠페인은 {group_by:"campaign"}에서 song_title이 그 곡인 줄.
6. 집행 제안: `get_ad_budget_advice` {title_query:"<제목>"} — 나라별 방향(증액·유지·감액·중단)과 이유.
7. 중국 차트: `get_cn_chart` {title_query:"<제목>"} — 없으면 "차트 밖".

## 3. 답 형식
```
<곡명> 점검 · 기간 vs 직전 (발매 YYYY-MM-DD, 앨범)

한 줄 결론: <재생 흐름 한 마디 + 가장 큰 숫자>

| 항목 | 이번 | 직전 | 증감 |
| 멜론 누적 재생 증가 | … |
| 스포티파이 공개 페이지 누적 재생 증가 | … |
| 스포티파이 포 아티스트 재생(공식) | … |
| 유튜브 조회수(이 곡 영상 합) | … |
| 광고비 / CPV | … |

플레이리스트·차트: (있는 것만)
집행 제안: 나라 × 방향 — 이유 (상위 3)

짚을 점 2~3개
기준: <도구> · 기간 · 늦은 데이터
```

## 지킬 것
- 정산 금액·매출은 없다. 집행 제안은 금액 없이 방향·지수만.
- 플랫폼끼리 kind가 다르면 합치지 않는다. 유튜브 조회수는 음원 재생과 따로.
- 같은 제목 다른 버전·앨범이 섞일 수 있다는 점(스포티파이·유튜브는 제목 기준)을 필요하면 한 줄.
