# PalM 연구원 (팔모R) — Memory

> PalM(Palworld Mobile) 데이터 탐색·연구 전담.
> 상세 이력·쿼리 주의사항: `palm-researcher-archive.md`
> **PalM 알파테스트 미탐색 영역 소진 — 현재 비활성**

---

## 최우선 원칙
1. 데이터 기반 팩트만 말한다 (할루시네이션 금지)
2. 확증 편향 경계 — 가설을 세우면 반증도 함께 탐색
3. 가설 기각은 실패가 아니라 발견

## 데이터 환경
- Databricks: VPN + `D:/queryer/query.py`
- 주요 테이블: `main.log_palm_live.*`
- 알파테스트 기간: 2025-12-05 ~ 2025-12-11
- 반드시 event_date 필터 적용 (기간 외 데이터 존재)

## 연구 이력 (6건 완료, 미탐색 소진)

| # | 주제 | 가설 판정 | 핵심 발견 | 리포트 |
|---|------|----------|----------|--------|
| 1 | dungeon-01 재도전 | 기각 | 연속 실패→재도전율 상승, 건강한 벽 | dungeon-01-retry-behavior.md |
| 2 | 세션·플레이 리듬 | 3건 채택 | Day3-4 몰입 급락(-66%), 첫 세션-잔존 상관 | session-play-rhythm.md |
| 3 | 팰 포획 행동 | 채택/채택/부분채택 | 다양성-잔존 상관, HP 75%+ 전원 실패 | pal-capture-behavior.md |
| 4 | 경제/자원 흐름 | 채택/채택/기각 | 기지 Day3 역전, 구운 열매 70.7%, 잔존 효과 소멸 | economy-resource-flow.md |
| 5 | 콘텐츠 소비 순서 | 채택/조건부/채택 | 71.4% 동일 시퀀스, Lv1-5 이탈 51.7% | content-consumption-sequence.md |
| 6 | 멀티레이드 참여 | 기각/채택/부분채택 | 잔존 효과 기각, 86.9% 솔로, R2 사망 급증 | multiraid-participation.md |

> 리포트 경로: `reports/research/palm/`

## 핵심 데이터 품질 경고

- lobby_login/logout: 알파 기간 0건 (사용 불가)
- msu_game_session_handling: 서버 세션(raid/square) 전용 (1,051명만)
- ingame_login 타임스탬프: UTC 기준 (KST = +9시간)
- 알파 기간 외 데이터 존재 — event_date 필터 필수
- streak 로직: "비-dead exit 누적합으로 그룹 정의" 방식이 올바름
- target_hp_percent: 0-1 스케일(비율), 퍼센트가 아님
- ingame_login: player_level이 아닌 `user_level` 컬럼
- ingame_nature_resource_collect: `collect_item_list`는 array<struct> → LATERAL VIEW EXPLODE
- ingame_pal_enrich: 동종 팰만 단련 재료로 사용 가능

## 팀 피드백 Round 1 (13건 기준)

**팀장 피드백 — 잘한 점:**
- 성장 궤적 뚜렷: 초기 3건 오류 → 4건째부터 2연속 검증 즉시 PASS
- 가설 기각을 두려워하지 않는 태도, 메타 분석 역량

**팀장 피드백 — 개선할 점:**
- 수치를 리포트에 옮길 때 검산 습관 유지
- 범위 분석 시 MAX/MIN 사전 확인

**셀프 체크리스트 (제출 전 확인):**
1. 핵심 수치 5개를 원본 쿼리 결과와 대조했는가?
2. 비율의 분모가 명시되어 있는가?
3. 요약의 수치와 본문의 수치가 일치하는가?

> 상세: `reports/research/team-feedback-round1.md`

<!-- 이후 작업 기록은 아래에 자동 추가됨 -->
