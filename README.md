# DEPT 대시보드 플러그인

대시보드 집계 데이터(스트리밍·광고 성과·SNS 등)를 AI에서 읽기 전용으로 조회합니다.
처음 쓸 때 대시보드 구글 로그인 → **[허용]** 하면 연결됩니다. (대시보드 멤버만)
쓸 수 있는 데이터·질문 예시는 서버가 알려 줍니다(도구 `get_dashboard_guide`) — 데이터가 바뀌어도 플러그인을 다시 깔 필요가 없습니다.

> 공개 저장소입니다. 깃허브 계정 없이 설치할 수 있어요. 데이터는 처음 쓸 때 대시보드 구글 로그인을 거쳐야 나옵니다.
> (터미널에서 `gh auth login` 또는 GitHub 로그인이 된 git).

## Claude Code

```
/plugin marketplace add Joseph-Lee97/dept-dashboard-plugins
/plugin install dept-dashboard@dept
```
설치 뒤 `/mcp` → `dept-dashboard` → 로그인(구글) → [허용].

업데이트:
```
/plugin marketplace update dept
```
(또는 `/plugin` → Marketplaces → dept → 자동 업데이트 켜기)

## Codex

```
codex plugin marketplace add Joseph-Lee97/dept-dashboard-plugins
codex plugin add dept-dashboard@dept
```
처음 도구를 쓸 때 브라우저가 열리면 구글 로그인 → [허용].

업데이트:
```
codex plugin marketplace upgrade dept
```

## 스킬(업무 흐름)

- `dept-dashboard-data` — 질문별 도구·순서, 기간 기본값(최근 7일 vs 직전 7일), 숫자 읽는 법, 답 형식
- `dept-weekly-report` — 주간 성과 보고(음원·곡·유튜브·SNS·광고) 정해진 양식
- `dept-song-checkup` — 곡 하나 점검(플랫폼별 재생·스포티파이 공식 분석·유튜브·플레이리스트·광고·집행 제안)
- `dept-ad-review` — 광고 효율 점검과 손볼 것 목록
- `dept-sns-weekly` — SNS 계정 팔로워 증감·잘 된 게시물 주간 요약

서버에 종합 분석 도구(`analyze_overview`·`analyze_song`·`analyze_ads`)가 있으면 스킬이 그것부터 쓰고, 없으면 개별 도구로 같은 결과를 만듭니다.

## 구조

- `.claude-plugin/marketplace.json` — Claude Code 마켓플레이스
- `.agents/plugins/marketplace.json` — Codex 마켓플레이스
- `plugins/dept-dashboard/` — 플러그인 (Claude `.claude-plugin/plugin.json` · Codex `.codex-plugin/plugin.json` · 공용 `plugin.json`, `.mcp.json`/`mcp.json`, `skills/`, `assets/`)

MCP 주소: `https://dept-mcp.lazzy.chat/mcp` (토큰·데이터는 이 저장소에 없습니다)
