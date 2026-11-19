---
name: dept-weekly-report
description: DEPT 주간 성과 보고를 정해진 양식으로 만들 때 사용. "주간 보고", "이번 주 성과 정리", "지난주 리포트", "주간 요약 써 줘" 같은 요청. 음원 스트리밍·곡·유튜브·SNS·광고를 한 장으로 묶는다.
---

# DEPT 주간 성과 보고

## 기간
- 말이 없으면 최근 7일(끝 = 어제) vs 직전 7일. "지난주"는 지난주 월~일 vs 그 전 주(from·to로 준다).

## 데이터 모으기
**A. `analyze_overview`가 있으면(안내 `get_dashboard_guide`의 tools에 있음) 그것 한 번이면 된다.**
- 기본 기간: 인자 없이. 지난주처럼 날짜를 정하면 `{from, to, compare:"prev_period"}`.
- 쓰는 칸: `headlines`(결론), `dsp_platforms`, `songs.rising/falling/spikes`, `youtube`, `sns.accounts/drops`, `sns_posts`, `ads.total/by_platform/anomalies`, `freshness`.

**B. 없으면 아래를 이번 기간·직전 기간 두 번씩 부른다.**
1. 음원: `get_streaming_trend` {scope:"song", platform:"melon", metric:"streams", from, to, granularity:"total"} — 멜론·`genie`(metric plays)·`spotify`(metric plays)도 같은 식으로 플랫폼마다. 행의 `increase` 합 = 그 플랫폼 기간 재생 증가. 오른/내린 곡은 곡별 `increase`를 이번 vs 직전으로 비교.
2. 스포티파이 공식: `get_track_daily` {source:"spotify", from, to, granularity:"week"} — 곡별 streams·saves·playlist_adds 상위 5.
3. 유튜브: `get_track_daily` {source:"youtube", from, to, granularity:"week"} — views·subs_gained 합, 상위 영상 3.
4. SNS: `get_sns_metrics` {from: 직전 시작, to, granularity:"week"} — 계정별 value_last 차이. 게시물: `get_sns_post_performance` {days:7, sort:"views", limit:5}.
5. 광고: `get_ad_performance` {group_by:"platform", from, to} 이번·직전, 상위 캠페인은 {group_by:"campaign"}.
6. 알림: `get_alerts` {} — 켜진 알림이 있으면 짚을 점에.

## 양식 (이대로)
```
[DEPT 주간 성과] M/D(요일)~M/D(요일) · 직전 주 대비

한 줄 결론: <가장 큰 변화 하나 + 숫자>

1. 음원 스트리밍
| 플랫폼 | 이번 주 | 직전 주 | 증감 |
(누적 재생 플랫폼은 늘어난 재생, 28일 합 플랫폼은 따로 표시 — 서로 더하지 않음)

2. 곡
- 많이 오른 곡: 곡명 +N회(+P%) ×3
- 많이 내린 곡: 곡명 −N회(−P%) ×3
- 평소 대비 급변: (있을 때만)

3. 유튜브(공식 분석)
조회수 N(±P%) · 시청 시간 N분 · 구독 증가 N · 많이 본 영상 1~3

4. SNS
| 계정 | 팔로워 | 이번 주 증감 | 직전 주 증감 |  (주요 계정만, 줄어든 곳 표시)
게시물 N개(직전 N개) · 조회 1위: 플랫폼 "제목" N회

5. 광고(구글·메타)
| 구분 | 광고비 | 조회 | CPV | CTR | (전체·구글·메타, 직전 대비 %)
확인할 것: (CPV 급등·광고비 멈춤 등 있을 때만)

짚을 점
- 2~3개(원인 짐작은 데이터로 보이는 것만: 예 "틱톡 리액션 게시물 6만 회와 같은 주")

기준: <쓴 도구> · 기간 · 늦은 데이터(예: 스포티파이 포 아티스트·유튜브 공식 분석 M/D까지)
```

## 지킬 것
- 정산 금액·매출은 넣지 않는다(이 연결로 안 나옴). 광고비는 집행 실적(KRW)만.
- 표의 숫자는 도구 결과 그대로, 천 단위 쉼표. 없는 칸은 "값 없음".
- 유튜브 조회수를 음원 재생에 더하지 않는다. kind가 다른 플랫폼끼리 합치지 않는다.
- 데이터가 늦은 출처는 "M/D까지 기준"을 붙인다.
