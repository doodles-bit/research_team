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

### 2026-04-13 — 세션 생명주기와 멀티플레이어 접속 품질 분석 (세 번째 연구)
- **리포트**: `reports/research/copperhead/session-lifecycle-multiplayer-quality.md`
- **데이터 기간**: 2026-02-04 ~ 2026-04-11
- **핵심 발견**:
  - 세션 참여 실패율 12.1% (1,955건 중 237건), 92.4%가 UnknownError
  - 첫 시도 실패율은 3.5%로 낮음 -- 높은 전체 실패율은 재시도에 의한 증폭
  - 76명(7.5%)이 전체 실패 100% 발생 (실패 집중도 극단적)
  - 빌드별 실패율 0%~34.8%, 4월 초 빌드에서 0~2.4%로 개선 추세
  - 솔로 세션 77.2%, 멀티 세션 미션 진행률 42.7% vs 솔로 27.7% (차이 15.0%p)
  - 세션 생성에서 첫 미션까지 중앙값 1.02분
- **가설 판정**: H1 부분 채택 (경과 시간 효과 있으나 재시도가 주원인), H2 채택 (빌드별 차이 명확), H3 채택 (인과관계 불확정)
- **상태**: 검증 MINOR 수정 완료, 팀장 리뷰 대기

### 2026-04-13 — 세션 생명주기 리포트: 검증 후 MINOR 수정
- **검증 판정**: MINOR (2건 수정)
- **수정 내용**:
  1. Success 고유 계정 수 오류: 911 -> 997 (섹션 4.2 표). `SELECT COUNT(DISTINCT account_id) FROM cphsessionjoined WHERE sessionjoinstatus = 'Success'` 결과 997명
  2. 평균 개인 실패율 표기 혼동: "49.5%(시도 8.4건 중 3.1건 실패)" -> 개인별 평균(mean of ratios) 49.5%와 합산 기준(ratio of means) 37.3%를 분리 서술. 두 통계량의 차이를 설명 추가
- **교훈**: mean of ratios와 ratio of means를 혼동하지 말 것. 괄호 안 보조 수치가 본문 수치와 대응하는지 반드시 확인

## 세션 테이블 메타데이터

### cphsessioncreated (신 테이블, 2026-02~)
- 건수: 2,857 (unique 세션 2,747, unique 플레이어 2,305)
- 주요 컬럼: account_id, onlinesessionid, sessioncreationstatus(항상 Success), sessionserverregion, mapname(항상 MainMenu), buildversion, computername, sessioncreatorid/name
- sessioncreationtime: string 타입, CAST 필수

### cphsessionjoined (신 테이블, 2026-02~)
- 건수: 1,955 (unique 플레이어 1,008)
- 주요 컬럼: sessionjoinstatus (Success/UnknownError/SessionDoesNotExist), sessionplayercount, sessiondurationinminutesatjoin, sessionjointime
- 모든 값 string 타입 -> CAST 필수
- joiningplayerid: 항상 None

### cphleavesessionevent (구 테이블, 2025-09~2026-01)
- 건수: 847
- sessionleavestatus: Success(533)/Failed(314)
- sessiondurationinminutesatleave: 최대값 10.6억 분(데이터 오류)
- sessionmembers: JSON-like string

### cphplayerspawned (2026-02~)
- 건수: 24,935
- 주요 컬럼: mapname, playername, playerlocation(좌표), playerrotation
- 21개 맵 관측: Lobby_Basic(8,129), MIS_Proto2_2k_01~04(각 3,700~4,700), 기타 GYM/POI 맵

### cphcreatesessionevent (구 테이블, 2025-09~2026-04)
- 건수: 3,048, cphsessioncreated와 유사 구조

### cphjoinsessionevent (구 테이블, 2025-09~2026-04)
- 건수: 2,247, cphsessionjoined와 유사 구조
- sessionmembers 컬럼 있음
- 사전 베타(2025-09~2026-01) 실패율: 10.6% (237/2,241)

### cphplayerlogin (2026-02~)
- 건수: 178 (매우 적음)
- loginsuccessful, logintime 포함

### cphplayerdownedevent (구 테이블, 2025-09~)
- 건수: 3 (사실상 사용 불가)

## 데이터 품질 주의사항 (추가)

### 빌드 버전 불일치 부재
- 참여자와 세션 생성자의 빌드 버전이 다른 경우는 0건
- 베타 환경에서 동일 빌드 강제 적용으로 추정

### 서버 지역 분포
- us-west-2: 94.5% (2,595/2,747 세션), eu-central-1: 4.5%, ap-northeast-2: 1.0%
- 비US 지역은 표본이 작아 통계적 비교 어려움

### 세션 참여 실패 집중도
- 76명(7.5%)이 237건 실패 전부 발생. 933명(92.5%)은 실패 0회
- 상위 3명이 51건(21.5%) 발생

### 2026-04-13 — 플레이어 다운(전투 사망) 패턴 분석 (네 번째 연구)
- **리포트**: `reports/research/copperhead/player-down-combat-pattern.md`
- **데이터 기간**: 2026-02-04 ~ 2026-04-10
- **핵심 발견**:
  - 다운 357건 중 AI 적 259건(72.5%), 자해 61건(17.1%), 환경/원인미상 37건(10.4%). PvP 다운은 0건
  - Trooper(사격형)가 AI 다운의 45.6%, Husk(근접형) 29.3%. 두 유형 합산 74.9%
  - 피격 경직(HitReact) 동반 다운이 AI 다운의 41.3% — 연쇄 피격 패턴 가능성
  - 다운 많은 세션이 오히려 성공률 높음(1회 35.9% → 7+회 72.7%) — 생존자 편향(미션 시간 교란)
  - AAntle 1인이 64건(17.9%) 기록, 자해 44건 중 40건이 SelfRevive 테스트
  - 상호작용(터렛/기기) 중 다운 9건(2.5%) — 무방비 취약 상태
- **가설 판정**: H1 채택(특정 적 유형 쏠림), H2 채택(경직 동반 41.3%), H3 기각(다운↑=성공↑, 생존자 편향)
- **상태**: 검증 MINOR 수정 완료, 팀장 리뷰 대기

## cphplayerdowned 테이블 메타데이터

### 스키마 주요 컬럼
- account_id: 보고한 플레이어의 계정 ID
- username: 보고한 플레이어 이름 (클라이언트 측)
- targetname: 다운된 플레이어 이름
- instigatorname: 다운 가해자 (BP_AICharacter_* = AI, 플레이어명 = 자해/타인)
- instigatortags: 공격 유형 태그 (Damage.Projectile, Damage.Melee, Damage.Explosive, Damage.Ability)
- targettags: 다운 시 플레이어 상태 (무기, 클래스, CharacterState 등)
- damagetags, eventtag: 대부분 None
- instigatorlocation, targetlocation: 3D 좌표 (V(X=, Y=, Z=) 형식, string)
- mapname, onlinesessionid, buildversion, computername 등
- 모든 값 컬럼 string 타입 → CAST 필수

### 데이터 품질
- 전체 357건, 132 계정, 55 고유 다운 플레이어(targetname), 39 고유 보고자(username)
- 중복 없음: 357 unique eventid = 357 total rows
- "Other Player" 다운은 0건 — 비AI instigator는 모두 instigator=target(자해 61건) 또는 instigator=NULL(환경/원인미상 37건)
- username ≠ targetname인 151건은 팀원의 다운을 관찰한 기록
- AAntle 극단값: 64건(17.9%), 자해 44건(SelfRevive 테스트)
- 서브클래스 미표기: AI 다운 259건 중 126건(48.6%)에 SubclassPassive 태그 없음

### AI 적 유형 목록 (instigatorname 패턴)
- BP_AICharacter_Trooper_Mover: 사격형 병사 (최다 118건)
- BP_AICharacter_Husk_Mover: 근접 돌격형 (55건, Damage.Melee)
- BP_AICharacter_Husk_Bomber_Mover: 폭발형 허스크 (20건, Damage.Explosive)
- BP_AICharacter_Trooper_Melee_Mover: 근접 병사 (38건)
- BP_AICharacter_Shaman_Mover: 원거리 특수형 (24건)

### 연관 테이블 조인
- onlinesessionid로 cphmissionsucceeded/cphmissionfailed와 조인 가능
- 미션 결과 테이블의 missioncompletiondurationminutes 활용 가능

## 미탐색 테이블 현황 (4차 연구 시점)
### 탐색 완료
cphmissionstarted, cphmissionsucceeded, cphmissionfailed, cphplayerheartbeat, cphsessioncreated, cphsessionjoined, cphleavesessionevent, cphplayerspawned, cphplayerlogin, cphplayerdownedevent, cphcreatesessionevent, cphjoinsessionevent, **cphplayerdowned**

### 미탐색 (잔여 관심)
- view_client_gpp_device_info: 14,482건, PC 하드웨어 상세(CPU/GPU/RAM struct). FPS 분석과 연계 가능
- view_client_gpp_user_entry: 로그인 flow, error_code, login_method 정보
- sessionstart/sessionend: 텔레메트리 세션(11,785/7,736건), buildversion, computername 포함
- au_base: 사용자 기반 집계 테이블 (스키마 미확인)
- connected/disconnected: 2026-02 이후 0건 (빈 테이블)

### 2026-04-13 — 플레이어 다운 패턴 리포트: 검증 후 MINOR 수정
- **검증 판정**: MINOR (4건 수정)
- **수정 내용**:
  1. 다운 원인 대분류 오류: 2분류(AI 259 + 자해 98) -> 3분류(AI 259 + 자해 61 + 환경/원인미상 37). instigatorname NULL 37건이 CAST 후 'None'으로 변환되어 자해에 잘못 포함됨
  2. 섹션 4.7 자해 분석 정합성: 98건 -> 61건 기준으로 수정, 환경 피해 37건을 별도 하위 항목으로 분리
  3. HitReact 배타적 분류 명시: CASE WHEN 우선순위에 의한 85건과 비배타적 115건의 차이를 주석으로 설명
  4. 맵별 15건 누락 맵 주석: MIS_VS_2k_ArtPrimaryPOI 6, Hub 5, GYM 2, POI 2건을 테이블 하단에 명시
- **교훈**: CAST(NULL AS STRING) -> 'None' 변환으로 인한 NULL 처리 오류. 향후 NULL 컬럼 쿼리 시 IS NULL / IS NOT NULL 조건 별도 처리 필수

## 오류 패턴 기록

### CAST(NULL AS STRING) -> 'None' 변환 문제 (중요)
- Databricks에서 CAST(NULL AS STRING)은 SQL 표준대로 NULL을 반환하지만, Python/PySpark에서 결과를 문자열로 읽을 때 'None'으로 표시될 수 있음
- instigatorname이 NULL인 환경 피해 건이 플레이어명 'None'과 혼동되어 자해로 분류됨
- **대응**: NULL 가능 컬럼은 항상 IS NULL / IS NOT NULL 조건을 CASE WHEN에 우선 배치할 것
- **영향**: 자해 건수 98->61, 환경/원인미상 0->37

<!-- 이후 작업 기록은 아래에 자동 추가됨 -->
