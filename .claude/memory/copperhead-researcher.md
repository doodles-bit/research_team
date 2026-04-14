# Copperhead 연구원 (코퍼헤드R) — Memory

> Copperhead(SDS) 데이터 탐색·연구 전담.
> 상세 이력·테이블 스키마: `copperhead-researcher-archive.md`

---

## 최우선 원칙
1. 데이터 기반 팩트만 말한다 (할루시네이션 금지)
2. 확증 편향 경계 — 가설을 세우면 반증도 함께 탐색
3. 가설 기각은 실패가 아니라 발견

## 데이터 환경
- Databricks: VPN + `D:/queryer/query.py`
- 테이블: `main.log_copperhead_beta.*`
- 베타 기간: 2026-02-04 ~ 현재
- 모든 컬럼 string → CAST 필수

## 연구 이력 (8건 완료)

| # | 주제 | 가설 판정 | 핵심 발견 | 리포트 |
|---|------|----------|----------|--------|
| 1 | 미션 성공/실패 | 채택/부분기각/기각 | 텔레메트리 80.7% 누락, Defend 9.1% | mission-success-failure-pattern.md |
| 2 | 성능 텔레메트리 | 채택/기각/채택 | FPS 50.7% 누락, 맵별 2.66배, 세션 -31% | performance-telemetry-analysis.md |
| 3 | 세션 생명주기 | 부분채택/채택/채택 | 참여 실패 12.1%, 솔로 77.2% | session-lifecycle-multiplayer-quality.md |
| 4 | 전투 다운 | 채택/채택/기각 | Trooper 45.6%, 경직 41.3%, PvP 0건 | player-down-combat-pattern.md |
| 5 | PC환경·로그인 | 채택/채택/부분채택 | 자동화 60.3%, 원격 60.4% | device-environment-login-flow.md |
| 6 | 무기·전투 생존 | 채택/부분채택/조건부채택 | AR/Sword/DMR 64.4%, Sword HitReact 52.1% | weapon-loadout-combat-pattern.md |
| 7 | 맵 진행 퍼널 | 채택/기각/기각 | 로비-only 37.4%, 허브-스포크, 스폰 40.1% | player-spawn-map-progression.md |
| 8 | 로비 CCU·체류 | 채택/채택/부분채택 | 금요일 피크, 이중 분포, 42~53분 타임아웃 | lobby-concurrent-user-dwell-pattern.md |

> 리포트 경로: `reports/research/copperhead/`

## 핵심 데이터 품질 경고

### 자동화 테스트 (모든 분석에 영향)
- PFLIGHT(5대), MINSPEC(1대), RECSPEC(1대) = 전체 세션의 60.3%
- 로비에서도 53.0% — 반드시 device_id 기반 필터링
- 매핑: kos_user_id ↔ account_id 조인으로 식별

| computername | device_id |
|-------------|-----------|
| SDS-PFLIGHT-10 | fb74c741db9c3b720f1740657ec1e5b0 |
| SDS-PFLIGHT-05 | 04b153914acd931ee10a4fea61b1f747 |
| SDS-RECSPEC-CPH | c5d9c80e9c01dce06228adb1e74610b5 |
| SDS-MINSPEC-CPH | 2ebcf8687bbca35262d26876956898af |
| SDS-PFLIGHT-08 | 1914568039d5103b7b25025e9e5764ba |
| SDS-PFLIGHT-03 | 455699d6bc8a2ebf95a0c3eeee11a326 |
| SDS-PFLIGHT-11 | 43c20f9f55e08b19afebc0cda57b49a0 |

### 텔레메트리 누락
- 미션 결과: 시작 대비 19.3%만 기록 (80.7% 누락)
- heartbeat FPS/Ping: 50.7% 누락 (4/10 이후 유효)
- 스폰: sessionstart 대비 40.1% 커버리지

### NULL 처리
- CAST(NULL AS STRING) → 'None' 변환 주의 — IS NULL 별도 처리 필수
- instigatorname NULL = 환경/낙하 피해 (자해와 구분)

### 조인 키
- lobby ↔ sessionstart: `kos_user_id = account_id`
- lobby 내부: `analytics_id`(세션), `krafton_id`(사용자), `device_id`(기기)
- krafton_id(UUID)와 account_id(hex) 형식 다름 → kos_user_id 사용

### 기타
- 미션명 86.5%가 None + Very Easy (MIS_Proto2_2k 맵)
- 베타 = 내부 개발팀 중심 (평일:주말 9:1)
- 전용 무기 테이블 없음 — cphplayerdowned.targettags 파싱으로 무기 분석

## 탐색 현황

### 완료 (21개)
cphmissionstarted, cphmissionsucceeded, cphmissionfailed, cphplayerheartbeat, cphsessioncreated, cphsessionjoined, cphleavesessionevent, cphplayerspawned, cphplayerlogin, cphplayerdownedevent, cphcreatesessionevent, cphjoinsessionevent, cphplayerdowned, view_client_gpp_device_info, view_client_gpp_user_entry, sessionstart, sessionend, view_lobby_concurrent_user, view_lobby_user_heartbeat, view_lobby_connected, view_lobby_disconnected

### 미탐색
- view_iam_logged_in: 14,265건 — IAM 로그인
- view_iam_registered: 14,020건 — IAM 등록
- view_client_gpp_token_refresh: 1,443건 — 토큰 갱신
- 빈 테이블: au_base, concurrent_user, logged_in, registered, telemetry_session_start

## 팀 피드백 Round 1 (13건 기준)

**팀장 피드백 — 잘한 점:**
- 데이터 품질 이슈 발견 능력 우수 (미션 80.7% 누락, 자동화 60.3%, heartbeat 50.7% 누락)
- 미탐색 테이블 적극 도전, targettags 파싱 등 창의적 데이터 활용
- REVISE 후 회복력: 동일 유형 미재발

**팀장 피드백 — 개선할 점:**
- 매 연구마다 다른 유형의 새 오류 발생 (8건 8가지)
- 공통 원인: 수치를 옮기고 집계할 때 원본과 대조하지 않음

**셀프 체크리스트 (제출 전 반드시 확인):**
1. 핵심 수치 5개를 원본 쿼리 결과와 대조했는가?
2. 요약에 쓴 수치가 본문과 일치하는가?
3. 모든 비율에 분모(N=?)가 명시되어 있는가?
4. 소계/합계가 개별 행의 합과 일치하는가?
5. NULL 컬럼을 IS NULL/IS NOT NULL로 별도 처리했는가?
6. 테이블 데이터 출처(어떤 테이블 기준)가 명시되어 있는가?
7. 히스토그램 구간 합계가 전체 N과 일치하는가?

> 상세 오류 패턴: `reports/research/team-feedback-round1.md`

<!-- 이후 작업 기록은 아래에 자동 추가됨 -->
