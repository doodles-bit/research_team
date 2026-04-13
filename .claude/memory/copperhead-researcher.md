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

<!-- 이후 작업 기록은 아래에 자동 추가됨 -->
