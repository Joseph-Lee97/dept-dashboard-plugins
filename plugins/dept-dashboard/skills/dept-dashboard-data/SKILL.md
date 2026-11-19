---
name: dept-dashboard-data
description: DEPT 대시보드 집계 데이터(음원 스트리밍·DSP, 유튜브, SNS, 광고 성과, 앨범·발매, 중국 차트, 알림 등)를 조회해 답할 때 사용. 대시보드 숫자·추이·비교·"요즘 어때"를 묻는 질문. "설정 도와줘"·"연결해 줘"·"로그인" 요청에도 사용.
---

# DEPT 대시보드 데이터 조회

읽기 전용 연결이다. 숫자는 지어내지 않고 도구 결과만 쓴다.

## 1. 시작
- 처음 쓰면 `get_dashboard_guide`를 한 번 불러 지금 쓸 수 있는 도구·`latest_data_day`(출처별 마지막 날짜)·`start_here`를 본다. 도구 목록은 안내가 정답이다(아래 이름이 안내에 없으면 그 도구는 아직 없는 것).
- 정해진 업무는 해당 스킬을 따른다: 주간 보고 `dept-weekly-report`, 곡 하나 점검 `dept-song-checkup`, 광고 효율 `dept-ad-review`, SNS 주간 요약 `dept-sns-weekly`.

## 2. 어떤 질문에 어떤 도구 (순서대로)
| 질문 | 먼저 | 그다음(필요할 때만) |
|---|---|---|
| 요즘 어때·이번 주/어제 성과·뭐가 올랐어·이상한 거 있어? | `analyze_overview` (안내에 있으면) | 궁금한 부분만 아래 도구 |
| 곡 하나(이 곡 어때·점검) | `analyze_song` (있으면) / 없으면 `list_artists_songs` → `get_streaming_trend`(title_query) | `get_track_daily`, `get_ad_performance`(group_by=song) |
| 플랫폼별 재생·청취자·팔로워 추이 | `get_streaming_trend` | — |
| 스포티파이 곡별 순위·저장·플레이리스트 추가 / 유튜브 영상별 | `get_track_daily`(source=spotify / youtube) | — |
| 광고비·CPV·나라·캠페인 | `analyze_ads` (있으면) / `get_ad_performance` | `get_ad_funnel`, `get_ad_budget_advice`, `get_ad_realtime_ranking`, `get_campaign_links` |
| SNS 계정 팔로워 / 게시물 반응 | `get_sns_metrics` / `get_sns_post_performance` | — |
| 앨범·발매일·크레딧 | `get_albums_releases` | — |
| 중국 차트 순위 | `get_cn_chart` | — |
| 협업 효과·공연 기록 | `get_collab_effects` | — |
| 대시보드 알림 | `get_alerts` | — |
| 개발 요청 현황 | `get_dev_requests` | — |

- `analyze_*`(종합 분석)는 서버가 증감·평소 대비·요약(`headlines`)을 미리 계산해 준다. 있으면 먼저 쓰고, 숫자를 여러 도구에서 모아 직접 계산하지 않는다.
- 곡·아티스트 이름이 나오면 이름부터 찾는다: `list_artists_songs`(title_query=제목 일부) → song_key/제목 확인. 여러 곡이 맞으면(원곡·어쿠스틱·언어 버전) 원곡 기준으로 하고 "다른 버전: …"을 한 줄로.
- `get_streaming_trend`를 곡 전체로 부를 땐 platform이나 title_query를 꼭 준다(전체 곡 × 전체 플랫폼은 시간 초과).

## 3. 기간
- 말이 없으면 최근 7일(끝 날짜 = 어제), 비교는 직전 같은 길이(직전 7일). "어제"는 하루 vs 그 전날, "이번 달"은 1일~어제 vs 지난달 같은 날수.
- `analyze_*`는 `compare`(prev_day·prev_week·prev_period)와 from·to로 준다. 다른 도구는 from·to를 두 번(이번·직전) 부른다.
- 출처마다 마지막 날짜가 다르다(스포티파이 포 아티스트·유튜브 공식 분석은 2~3일 늦음). 결과의 `data_to`·`latest_day`·`coverage`를 보고, 늦으면 "○○은 M/D까지 기준"이라고 적는다.

## 4. 숫자 읽는 법(안내에 있는 것만)
- DSP = 음원 플랫폼(멜론·지니·스포티파이·애플 뮤직 등). 답엔 응답의 `label`(한국어 이름)을 쓰고 코드(s4a, cm, ch:… 등)는 쓰지 않는다.
- `kind`: 누적값(cumulative)은 기간에 늘어난 양 `increase`를 쓰고 날짜끼리 더하지 않는다 · 최근 28일 합(window28)은 그대로(더하지 않음) · 하루 값(flow)은 기간 합 · 순위 등(snapshot)은 마지막 값.
- 음원 스트리밍 합계에 유튜브 조회수를 더하지 않는다. kind가 다른 플랫폼끼리 합치지 않고 플랫폼별로 적는다.
- 스포티파이 `listeners`(get_track_daily)는 일별 청취자 합이라 고유 청취자가 아니다.
- 광고: 광고비는 KRW. CPV = 광고비 ÷ 조회(낮을수록 좋음), CPM = 노출 1,000회당 광고비, CTR = 클릭 ÷ 노출. 집행 제안의 `cpv_index`는 국가 평균 CPV = 1(낮을수록 좋음).
- `truncated=true`면 기간·묶음을 줄여 다시 부른다(잘린 결과로 결론 내지 않음). 값이 비면 "그 기간 값 없음".

## 5. 하지 않는 것
- 정산 금액·단가·매출은 이 연결로 나오지 않는다(권한과 상관없이). 물으면 "정산 금액은 대시보드 정산 화면에서(권한자만) 봐 주세요"라고 한다. TIPS 예측, 개인 연락처, 계정 정보도 없다.
- 쓰기·수정 도구는 없다.

## 6. 답 형식
1. **결론 한 줄**(무엇이 얼마나 변했나, 기간 포함).
2. **표**(항목 | 이번 | 직전 | 증감). 숫자는 천 단위 쉼표, 증감은 %와 실제 수 둘 다.
3. **짚을 점 2~3개**(급변·이상·늦은 데이터).
4. 끝에 한 줄 **기준**: 쓴 도구 이름과 기준일(예: `기준: analyze_overview · 11/12~11/18 vs 11/5~11/11, 유튜브는 11/15까지`).

## 7. 처음 연결(로그인) 여는 법
`get_dashboard_guide` 같은 dept-dashboard 도구가 이 채팅에 안 보이면 아직 로그인 전이다. 사용자에게 방법을 설명하지 말고 바로 로그인 창을 연다.

- **Codex**
  1. 터미널에서 `codex mcp list`로 dept-dashboard(또는 비슷한 이름) 서버가 있는지 확인한다.
  2. 없으면 `codex mcp add dept-dashboard --url https://dept-mcp.lazzy.chat/mcp`로 추가한다(이미 있으면 건너뛴다).
  3. `codex mcp login dept-dashboard` 실행 → 나온 로그인 주소를 사용자의 기본 외부 브라우저로 연다. 내장 브라우저·컴퓨터 사용·플러그인 화면 클릭은 쓰지 않는다. 사용자가 대시보드 구글 로그인 → [허용]을 마칠 때까지 명령을 끄지 않고 기다린다.
  4. `codex mcp list`에서 로그인됨을 확인하고 "새 채팅을 열어야 도구가 붙어요"라고 알린다.
- **Claude Code**: `/mcp` → dept-dashboard → 인증(구글 로그인 → [허용]).
- 권한 때문에 브라우저를 못 열면 로그인 주소를 사용자에게 보여 주고 같은 컴퓨터의 브라우저에서 열어 달라고 한다.
- ChatGPT 대화(웹·데스크탑)에서는 회사 워크스페이스 플러그인 연결이 안 된다. Codex에서 쓰라고 안내한다.
