---
name: dept-sns-weekly
description: DEPT SNS·팬 채널 주간 요약을 만들 때 사용. "SNS 주간 요약", "이번 주 팔로워 얼마나 늘었어?", "인스타·틱톡 반응 정리", "잘 된 게시물 뭐야?" 같은 요청. 계정별 팔로워 증감과 게시물 반응(D+1/3/7 기록)을 묶는다.
---

# SNS 주간 요약

> **먼저 보고 일하기:** `data_catalog`가 있으면 먼저 불러(해당 recipe·데이터셋 cautions·도구 인자 확인) 쓸 데이터·도구와 이유를 한두 줄로 말한 뒤 조회한다. 숫자에는 `evidence`의 as_of·대시보드 링크를 각주로 붙인다. 자세한 규칙은 `dept-dashboard-data` 0·6절.

## 기간
- 말이 없으면 최근 7일(끝 = 어제) vs 직전 7일.

## 데이터 모으기
1. 계정 지표: `analyze_overview`가 있으면 그 결과의 `sns.accounts`(계정별 value·change·prev_change)와 `sns.drops`(줄어든 계정)를 쓴다.
   없으면 `get_sns_metrics` {from:<직전 시작>, to, granularity:"day", metric:"followers"} — 계정(platform+channel_label)마다 기간 끝 값 − 기간 시작 직전 값. 유튜브는 metric "subscribers", QQ뮤직은 "fans", 비스테이지는 "members".
2. 게시물: `get_sns_post_performance` {days:7, sort:"views", limit:10} — 상위 게시물, `milestones`의 d1/d3/d7.
   직전 주와 비교할 땐 {days:14}로 불러 posted_at으로 나눈다.
3. 플랫폼만 보고 싶으면 platform(instagram, tiktok, youtube, xiaohongshu, bilibili, douyin, weibo 등)이나 account로 좁힌다.

## 답 형식
```
SNS 주간 요약 · 기간 vs 직전

한 줄 결론: <가장 많이 는 계정 + 가장 잘 된 게시물>

| 계정 | 팔로워(구독자) | 이번 주 증감 | 직전 주 증감 |
(주요 계정 위주, 협력자 계정은 변화 있을 때만, 줄어든 곳은 표시)

잘 된 게시물 Top 3
| 플랫폼 | 올린 날 | 제목(짧게) | 조회 | 좋아요 | D+1 조회 |

게시물 수: N개(직전 N개)

짚을 점 2~3개(예: 팔로워 급증 계정과 같은 주 게시물, 줄어든 계정)
기준: <도구> · 기간 · 마지막 수집일
```

## 지킬 것
- 게시물 조회는 지금까지 쌓인 값이라 늦게 올린 게시물이 작게 보인다 — 공정한 비교는 D+1/D+3/D+7 기록으로.
- 팔로워 같은 누적값은 날짜끼리 더하지 않고 끝 값 − 시작 값.
- 계정 아이디·연락처 외 개인정보는 없다. 정산 금액도 이 연결에 없다.
