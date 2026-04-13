# Copperhead 연구원 (코퍼헤드R) — Memory

> Copperhead(Striking Distance Studios) 데이터 탐색·연구 전담.
> 자율적으로 데이터를 탐색하고 가설을 세우고 검증하여 인사이트를 발굴한다.

---

## 최우선 원칙
1. 데이터 기반 팩트만 말한다 (할루시네이션 금지)
2. 확증 편향 경계 — 가설을 세우면 반증도 함께 탐색
3. 가설 기각은 실패가 아니라 발견

## 분석팀 축적 지식 (참고)
- 분석팀이 완료한 Copperhead 분석: 미션 대시보드, heartbeat 텔레메트리 대시보드
- 분석팀 repo: `C:\Users\doodles\project_repo\ai_agent_team`

## 연구 이력

### 2026-04-13 — 미션 성공/실패 패턴 분석 (첫 번째 연구)
- **리포트**: `reports/research/copperhead/mission-success-failure-pattern.md`
- **데이터 기간**: 2026-02-04 ~ 2026-04-13
- **핵심 발견**:
  - Defend Mission Normal 성공률 9.1% (극단적 이상치)
  - 미션 결과 기록 누락률 80.7% (9,111 시작 중 1,757건만 결과 있음)
  - Very Easy가 Normal보다 소요 시간이 긴 역전 현상
  - 번호 미션은 순차 진행 구조가 아닌 것으로 추정
- **가설 판정**: H1 채택, H2 부분 기각, H3 기각 (재도전율 35%, 예상 20% 미만 초과)
- **상태**: 검증 MINOR 수정 완료, 팀장 리뷰 대기

### 2026-04-13 — 미션 성공/실패 패턴 분석: 검증 후 MINOR 수정
- **검증 리포트**: `reports/research/copperhead/mission-success-failure-pattern-verification.md`
- **수정 내용**:
  1. H3 판정 "채택" -> "기각": 원본은 "재도전 후 성공 2명(5%)"를 근거로 채택했으나, 가설은 "재시작 비율"을 물었음. 실제 재시작율 14/40=35%로 예상(20% 미만) 초과. 확증 편향 패턴.
  2. Mission 1 미기록 인원 312(72.9%) -> 314(73.4%): 성공/실패 2명 중복 미반영 산술 오류
  3. 솔로 vs 멀티 비교에 난이도 교란(confounding) 주의사항 추가
  4. 성공률 테이블에 산출 기준(성공/(성공+실패)) 명기
- **교훈**: 가설의 측정 지표를 정확히 정의할 것 -- "재시작 비율"과 "재시작 후 성공 비율"은 전혀 다른 지표

## 데이터 품질 주의사항

### 미션 텔레메트리 누락 (중요)
- `cphmissionstarted` 9,111건 대비 `cphmissionsucceeded` + `cphmissionfailed` = 1,757건 (19.3%)
- 원인 불명: 중도 이탈, 크래시, 텔레메트리 누락 중 하나

### 미션명 미기록
- 전체 시작의 86.5% (7,884건)가 `missionname = None`, `missiondifficultylevel = Very Easy`
- 해당 건은 `MIS_Proto2_2k_01~04` 맵에서 발생
- 결과 이벤트의 None 미션도 missionresult, duration, objectives 모두 None

### 인코딩 이슈
- 일부 미션명이 깨져서 표시됨 (예: "ӹ 1" = "미션 1"의 한글 인코딩 깨짐으로 추정)

### 이중 테이블 구조
- `cphmissionstartevent` (2025-09 ~ 2026-02, 7,641건): 이전 버전 이벤트
- `cphmissionstarted` (2026-02 ~ 현재, 9,111건): 현재 버전 이벤트
- `cphmissionendevent` (9건), `cphmissionend`/`cphmissionstart` (각 1건): 사실상 사용 불가

### 베타 특성
- 평일 활동이 주말의 ~9배 (내부 개발팀 중심)
- 컴퓨터명에 "SDS-" (Striking Distance Studios) 접두어 관찰
- 외부 테스터와 내부 개발자 구분 불가

## 주요 테이블 & 조인 키

| 테이블 | 주요 용도 | 건수 |
|--------|----------|------|
| cphmissionstarted | 미션 시작 | 9,111 |
| cphmissionsucceeded | 미션 성공 (duration, objectives 포함) | 1,121 |
| cphmissionfailed | 미션 실패 (duration, objectives 포함) | 636 |
| cphplayerdowned | 플레이어 다운 (instigator 정보) | 395 |
| cphplayerheartbeat | 플레이어 성능 데이터 | 미확인 |

- **조인 키**: `account_id` (플레이어), `onlinesessionid` (게임 세션)
- `sessionid`는 텔레메트리 세션으로 `onlinesessionid`와 별개

### 2026-04-13 — 성능 텔레메트리(Heartbeat) 분석 (두 번째 연구)
- **리포트**: `reports/research/copperhead/performance-telemetry-analysis.md`
- **데이터 기간**: 2026-03-30 ~ 2026-04-11 (190,902건, 339명)
- **핵심 발견**:
  - heartbeat 데이터 50.7% FPS/Ping 누락 -- 3/30 100% 유효(68건), 4/1~4/4 100% 누락, 4/10 이후 0% (텔레메트리 수정)
  - Listen Server(호스트) 평균 FPS 91.9 > Client 82.3 (가설 H2 기각), LS 계정 수 135
  - 단, 30 FPS 미만 구간의 89%가 Listen Server에서 발생 (호스트 부하의 양극단 분포)
  - 맵별 FPS 차이 2.66배: VS_Art(64.2) ~ Sentinel(170.7)
  - 4+ 플레이어 세션 FPS(70.5) = 솔로(105.4) 대비 -33.1%
  - 세션 0~1분(109.6 FPS) -> 10~20분(76.0 FPS) 구간 FPS가 시작 대비 31% 하락
- **가설 판정**: H1 채택(누락 시기 집중), H2 기각(호스트 FPS 저하 아님), H3 채택(맵별 차이 존재)
- **상태**: 검증 REVISE 수정 완료, 재검증/팀장 리뷰 대기

## heartbeat 테이블 메타데이터

### 스키마 주의사항
- 모든 값 컬럼이 string 타입 -> CAST 필수
- snake_case 중복 컬럼(`average_fps`, `ping_ms` 등)은 모두 None -- camelCase 사용할 것
- heartbeat 주기: 약 1초 (중앙값 1초)
- `pingms`: Listen Server일 때 항상 0

### 데이터 품질
- 3/30: 68건 전부 유효(100%), 3/31: 2,402건 중 24건 유효(1.0%)
- 4/1~4/4: FPS/Ping/missionname 100% 누락
- 4/6: 1.3% 유효, 4/7: 70.7%, 4/8: 78.1%, 4/9: 63.1%
- **4/10~4/11만 신뢰 가능 (52,873건, 유효율 100%)**
- GYM_FlyingAI, GYM_Player: FPS 100% 누락 (4/10 이후 포함)

### 주요 통계 (유효 건 기준)
- FPS: 평균 87.5, 중앙값 83.0, P10=45, P90=129
- Client Ping: 평균 108.1, 중앙값 94, P90=148, P99=566
- FPS-Ping 상관: r=-0.033 (무상관)
- 200+ FPS 이상치: 327건, 21명 (1명이 180건, 평균 1157 FPS)

### 플레이어 분포
- US 184명, CA 85명, GB 69명, KR 2명
- SDS- 33기기/223계정, BBI- 5기기/30계정, Other 8기기/86계정
- heartbeat 339명 중 미션 데이터와 겹치는 계정: 236명

### 2026-04-13 — 성능 텔레메트리 리포트 REVISE 수정
- **검증 리포트**: `reports/research/copperhead/performance-telemetry-analysis-verification.md`
- **수정 내용** (검증원 REVISE 판정 반영):
  1. 날짜별 건수 테이블 전면 재검증: 3/30(5,367->68), 3/31(15,568->2,402), 4/1(19,005->16,434), 4/2(15,783->10,457), 4/3(28,222->44,586)
  2. 3/30 유효율 수정: 0%->100% (68건 전부 유효). "3/30~4/4 100% 누락" -> "4/1~4/4 100% 누락"으로 서술 변경
  3. 세션 0~1분 FPS: 101.2->109.6, 하락률 25%->31%, 건수도 소폭 수정
  4. SDS 기기 수: 78->33
  5. LS 계정 수: 146->135
  6. 신뢰 구간 건수: 52,897->52,873
  7. Ping 1-50ms 구간: 건수 6,564->6,589, 평균 FPS 95.0->100.2
  8. MIS_Proto2_2k_01 LS FPS: 109.5->109.4
  9. 인과 표현 완화: "유발하는 것으로 보인다"->"집중적으로 발생한다", "메모리 누수 점검"->"원인 추가 조사 필요"
- **교훈**: 날짜별 건수 불일치는 event_date vs ingestion date 혼동 가능성. 쿼리 재실행으로 반드시 확인할 것. account_id 컬럼명 주의(accountid 아님).

<!-- 이후 작업 기록은 아래에 자동 추가됨 -->
