---
name: dept-dashboard-data
description: DEPT 대시보드 집계 데이터(곡·아티스트 스트리밍, 광고 성과, SNS 팔로워·게시물)를 조회해 표·요약으로 답할 때 사용. "We Talk 10월 멜론", "지난주 광고비", "인스타 팔로워 추이" 같은 질문.
---

# DEPT 대시보드 데이터 조회

DEPT 대시보드 MCP 서버(dept-dashboard)의 읽기 전용 도구로 답한다. 데이터는 로그인한 대시보드 멤버 권한만큼만 나온다.

## 순서
1. 곡·아티스트 이름이 나오면 먼저 `list_artists_songs`로 artist_id·song_key를 찾는다(제목 일부로 검색 가능).
2. 질문에 맞는 도구를 고른다.
   - 플랫폼별 스트리밍·조회 추이(멜론·스포티파이·유튜브 등): `get_streaming_trend` (scope=song|artist, granularity=day|week|month)
   - 스포티파이 곡별·유튜브 영상별 하루 값: `get_track_daily` (source=spotify|youtube)
   - 광고비·노출·조회·클릭·CPV·CPM·CTR: `get_ad_performance` (group_by=campaign|song|country|day|platform, platform=google|meta)
   - SNS·팬 채널 팔로워·구독자: `get_sns_metrics`
   - 최근 게시물 성과(D+1/3/7/28): `get_sns_post_performance`
3. 기간이 없으면 최근 7일(비교는 그 전 7일)로 하고 답 첫 줄에 기간을 적는다.
4. 숫자는 표로, 핵심 변화는 한두 줄 요약. 데이터가 비면 "그 기간 값 없음"이라고 말하고 지어내지 않는다.

## 주의
- 정산 금액은 권한자만 나온다. 안 나오면 권한 때문이라고 안내한다.
- 처음 쓸 때 대시보드 구글 로그인 → [허용]이 필요하다.
