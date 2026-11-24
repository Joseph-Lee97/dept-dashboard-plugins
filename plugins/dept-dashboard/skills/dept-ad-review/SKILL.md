---
name: dept-ad-review
description: DEPT 구글·메타 광고 효율을 점검하고 손볼 것을 뽑을 때 사용. "광고 효율 점검", "이번 주 광고 어때?", "CPV 비싼 캠페인", "광고 뭐 고쳐야 해?", "증액할 곳 있어?" 같은 요청.
---

# 광고 효율 점검

> **먼저 보고 일하기:** `data_catalog`가 있으면 먼저 불러(해당 recipe·데이터셋 cautions·도구 인자 확인) 쓸 데이터·도구와 이유를 한두 줄로 말한 뒤 조회한다. 숫자에는 `evidence`의 as_of·대시보드 링크를 각주로 붙인다. 자세한 규칙은 `dept-dashboard-data` 0·6절.

## 기간
- 말이 없으면 최근 7일(끝 = 광고 마지막 수집일) vs 직전 7일. 집행 제안은 최근 28일 기준(도구 고정). 기간 비교는 `dept-dashboard-data` 3절을 따른다. 수집 범위가 서로 다르거나 불명확한 0원·CPV 변화는 이상 신호로 보지 않는다.

## 데이터 모으기
**A. `analyze_ads`가 있으면 한 번:** {} 또는 {from, to, compare:"prev_period"}. 쓰는 칸: `headlines`, `to_fix`(손볼 것), `total`, `by_platform`, `campaigns`, `countries`, `funnel`, `creatives`, `advice`, `unlinked`.

**B. 없으면:**
1. 플랫폼 합계: `get_ad_performance` {group_by:"platform", from, to} — 이번·직전 두 번.
2. 캠페인: `get_ad_performance` {group_by:"campaign", from, to} — 광고비 상위·CPV 비싼 곳. 직전 기간도 불러 CPV 변화 비교.
3. 나라: `get_ad_performance` {group_by:"country", from, to}.
4. 퍼널·소재: `get_ad_funnel` {view:"funnel", kind:"total", from, to} · {view:"creatives", from, to, limit:5}.
5. 최근 순위·변동: `get_ad_realtime_ranking` {days:7}.
6. 집행 제안: `get_ad_budget_advice` {} — `action_counts`, 증액(up)·감액(down)·중단(stop) 줄의 `reason`.
7. 곡 미연결: `get_campaign_links` {unlinked_only:true, active_only:true}.
8. 알림: `get_alerts` {} — 광고비 0원·CPV 급등 알림.

## 손볼 것 고르는 기준(데이터로만)
- CPV가 가장 좋은 캠페인의 2배 이상인 캠페인 → 소재·타깃 확인.
- 직전 기간 대비 CPV +50% 이상(광고비 5만 원 이상).
- 직전엔 쓰다가 이번 기간 광고비 0원 → 일부러 끈 건지 확인.
- 집행 제안 중단·감액·증액 줄(이유 그대로).
- 곡 미연결인데 최근 30일 광고비 있는 캠페인.

## 답 형식
```
광고 효율 점검 · 기간 vs 직전

한 줄 결론: <광고비·CPV 변화 + 가장 손볼 곳>

| 구분 | 광고비 | 조회 | CPV | CTR | 직전 대비 |
| 전체 / 구글 / 메타 |

캠페인 상위 5 (광고비 순): 캠페인 | 곡 | 광고비 | CPV | CPV 변화
나라 상위 5: 나라 | 광고비 | CPV
퍼널: 조회율 · 끝까지 시청 · 클릭률

손볼 것
1. …(근거 숫자)
2. …
3. …

기준: <도구> · 기간 · 광고 마지막 수집일
```

## 지킬 것
- 광고비는 집행 실적(KRW)만. 정산 수익·예상 정산·매출은 이 연결로 안 나오며, 집행 제안도 금액 없이 CPV·조회율 기준이다.
- CPV·CTR은 도구가 준 값을 쓰고, 직접 나눌 땐 광고비 ÷ 조회, 클릭 ÷ 노출.
- 손볼 것은 숫자 근거가 있는 것만, 최대 5개.
