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

### 2026-04-13 — PC 환경 및 로그인 흐름 분석 (다섯 번째 연구)
- **리포트**: `reports/research/copperhead/device-environment-login-flow.md`
- **데이터 기간**: 2026-02-04 ~ 2026-04-14
- **핵심 발견**:
  - 134대 고유 기기 중 60.4%(81대)가 원격/가상 접속 어댑터(Parsec 등) GPU
  - 전체 11,285 계정 중 60.3%(6,805명)가 자동화/공유 테스트 머신(PFLIGHT, MINSPEC, RECSPEC) 7대에서 접속
  - PFLIGHT: 5대, 기기당 800명, 시간당 최대 20명 동시 접속, 세션 종료 누락률 85.4%
  - MINSPEC/RECSPEC: 각 1대, 기기당 1,388~1,395명, 미션 참여율 48~51%
  - 로그인 성공률 98.7%, 실패 104건 모두 "HttpResponse is null"(서버 응답 없음)
  - 로그인 중앙값 4초, 96.1%가 10초 이내 완료
  - 전체 계정이 정확히 1대 기기에서만 사용 -- 자동 테스트 일회용 계정 구조
- **가설 판정**: H1 채택(원격/가상 GPU 60.4%), H2 채택(자동화 60.3%), H3 부분 채택(실패 집중이나 실패율 자체가 낮음)
- **상태**: 검증 MINOR 수정 완료, 팀장 리뷰 대기

## view_client_gpp_device_info 테이블 메타데이터

### 스키마
- device_info struct 내 pc.gpu, pc.cpu, pc.ram_total_capacity 등 중첩 구조
- `pc` 컬럼과 `device_info.pc` 컬럼이 별도 존재. 베타 기간 데이터는 `device_info.pc.*` 사용
- `pc.*` 컬럼은 베타 기간(2026-02~) 모두 None
- GPU 배열: `device_info.pc.gpu[0].gpu_name`으로 첫 번째 GPU 접근
- RAM: `device_info.pc.ram_total_capacity` (byte 단위, string -> DOUBLE 변환 후 / 1073741824 = GB)
- 조인키: device_id (기기 식별), analytics_id (앱 실행 세션, `{device_hash}#{timestamp}` 형식)
- 건수: 14,529 (베타 기간), 134 고유 device_id, 14,057 고유 analytics_id

### 데이터 품질
- 원격 어댑터가 GPU로 보고됨: Parsec Virtual Display Adapter, Microsoft Remote Display Adapter 등
- 9대(6.7%)가 시기에 따라 원격/물리 GPU 전환
- AMD Radeon RX 6900 XT + Threadripper PRO 3995WX 조합(5대)이 이벤트의 37.3% 차지

## view_client_gpp_user_entry 테이블 메타데이터

### 스키마
- 로그인 흐름 추적: entry_step(app_init, try_login, server_set_status_off, login_complete, login_failed)
- analytics_id: 앱 실행 세션 (`{device_hash}#{timestamp}`)
- error_code, error_message: 실패 시 에러 정보
- login_method: 'device', login_type: 'manual', login_flow_type: 'full_kid' 등
- 건수: 14,529 app_init 이벤트 (베타 기간), 14,060 고유 analytics_id
- 하나의 analytics_id에서 4개 단계(app_init, try_login, server_set_status_off, login_complete)가 기록됨

## sessionstart / sessionend 테이블 메타데이터

### 스키마
- account_id, sessionid, buildversion, computername, username, event_at
- sessionid로 start/end 조인 가능
- 건수: sessionstart 11,793건(11,285 고유 계정), sessionend 7,803건(5,837 고유 계정)
- 종료 누락: 전체 37.0%, PFLIGHT 85.4%, MINSPEC 53.1%, RECSPEC 50.6%, SDS/BBI/External 11~13%

### 자동화 테스트 머신 식별
- PFLIGHT: SDS-PFLIGHT-05, SDS-PFLIGHT-10 (주력), 03/08/11 (소량)
- MINSPEC: SDS-MINSPEC-CPH (1대, 1,388 세션, 1,388 계정)
- RECSPEC: SDS-RECSPEC-CPH (1대, 1,395 세션, 1,395 계정)
- 빌드 다양성: 자동화 952 빌드 > SDS Dev 508 > External 299 > BBI 204

## 이전 연구와의 교차점

### 1차 미션 누락(80.7%)과의 관련
- MINSPEC/RECSPEC의 미션 참여율 48~51% -- 미션 시작은 하지만 정상 종료 이벤트 없이 프로세스 종료
- 자동화 환경에서 미션 결과(성공/실패) 미기록이 누락률의 상당 부분을 설명할 가능성

### 2차 성능 텔레메트리와의 관련
- heartbeat의 SDS-33기기/BBI-5기기와 sessionstart의 SDS-54대/BBI-12대는 불일치 -- heartbeat 미발송 기기 존재
- 원격 접속 기기의 FPS 수치가 실제 렌더링 성능을 반영하지 않을 수 있음

### 2026-04-13 — PC 환경 및 로그인 흐름 리포트: 검증 후 MINOR 수정
- **검증 판정**: MINOR (4건 수정, 3건 필수 + 1건 권고)
- **수정 내용**:
  1. GPU 소계 불일치: Parsec만 집계한 75대(56.0%) -> Parsec+MS Remote Desktop+Virtual Monitor+DisplayLink 합산 81대(60.4%)로 수정. 순수 원격 전용 72대(53.7%), 원격+물리 양용 9대(6.7%) 주석 추가. H1 채택 판정 유지(오히려 강화)
  2. 계정 수 경미 오차 통일: MINSPEC 1,387->1,388, RECSPEC 1,394->1,395, 전체 계정 11,284->11,285. 리포트 내 5.4절에서 이미 11,285로 표기하여 자체 불일치가 있었음 -> 전체 통일
  3. 섹션 5.4(계정-기기 1:1 대응) 재분류: "반증 탐색 결과"에서 제거, 4.2절 하위에 "보강 근거"로 이동. 내용은 H2를 보강하는 근거이지 반증이 아니었음
  4. sessionstart 중복 sessionid 주석: 세션 종료 누락률 테이블에 "sessionid 기준, 중복 미제거" 주석 추가. DISTINCT 기준 시 SDS Dev 11.3%->12.9%, External 13.0%->13.7%로 소폭 상승
- **교훈**: 소계 집계 시 하위 항목 전부 포함 여부 확인 필수. 동일 수치가 리포트 내 여러 곳에 등장하면 일관성 크로스체크 필요

### 2026-04-14 — 무기 선택과 전투 생존 패턴 분석 (여섯 번째 연구)
- **리포트**: `reports/research/copperhead/weapon-loadout-combat-pattern.md`
- **데이터 기간**: 2026-02-04 ~ 2026-04-10
- **핵심 발견**:
  - 9종 무기 관측, 상위 3종(AR 25.2%, Sword 21.6%, DMR 17.6%)이 64.4% 차지
  - Sword HitReact 동반 다운 52.1% vs AR 32.7% — 19.4%p 차이
  - 서브클래스별 무기 선호 명확: DPS→LMG/AR(83.3%), Tank→Sword/Shotgun(85.7%), Support(AbilityRegen)→DMR(72.7%)
  - AAntle 1인이 DMR 다운의 50.6%(45/89건), 33건 자해(SelfRevive 테스트) — 제외 후 분석 수행
  - 무기 태그 없는 다운 43건은 이미 Health.Downed 상태의 추가 기록
  - DMR 사용자의 Husk(근접형) 취약성 45.7% — 원거리 무기의 근접 돌격 대응 어려움 시사
  - 67.9% 플레이어가 1~2종 무기만 사용
- **가설 판정**: H1 채택(상위 3종 64.4%), H2 채택(Sword HitReact 52.1% vs AR 32.7%), H3 채택(서브클래스별 집중도 38.5~72.7%)
- **상태**: 초안 완료, 검증/팀장 리뷰 대기

## 무기 시스템 메타데이터

### 데이터 원천
- 전용 무기/장비 테이블은 존재하지 않음 (35개 테이블 중 없음)
- 무기 정보는 `cphplayerdowned.targettags` 필드에 다운 시점 스냅샷으로만 존재
- `cphplayerheartbeat`에는 무기 정보 미포함

### 관측된 무기 목록 (12종, 9카테고리)
| 태그 | 카테고리 | 다운 건수(AAntle 제외) |
|------|---------|---------------------|
| Weapon.Ranged.AR.AK47 | 돌격소총 | 63 |
| Weapon.Melee.Sword | 근접 | 54 |
| Weapon.Ranged.DMR.G3 | 지정사수소총 | 41 |
| Weapon.Ranged.LMG.M249 | 경기관총 | 32 |
| Weapon.Ranged.Shotgun.R870 | 산탄총 | 19 |
| Weapon.Ranged.Pistol.M1911 | 권총 | 14 |
| Weapon.Ranged.Carbine.MK | 카빈 | 11 |
| Weapon.Ranged.Heavy.Bazooka | 중화기 | 5 |
| Weapon.Ranged.SMG.MP | 기관단총 | 4 |
| Weapon.Ranged.DMR.M110 | 지정사수소총 | 3 |
| Weapon.Ranged.Shotgun.KeltecKSG | 산탄총 | 2 |
| Weapon.Ranged.Pistol.VP9 | 권총 | 2 |

### 서브클래스 목록 (5종)
- SubclassPassive.DPS.WeaponDamage: DPS(무기 데미지)
- SubclassPassive.DPS.AbilityDamage: DPS(어빌리티 데미지)
- SubclassPassive.Support.AbilityRegen: 서포트(어빌리티 재생)
- SubclassPassive.Support.GasRegen: 서포트(가스 재생)
- SubclassPassive.Tank.DamageResist: 탱크(피해 저항)

### targettags 구조
- 쉼표+공백으로 구분된 태그 목록
- 포함 정보: CharacterModelType(Male/Female), Character.ID.Player, Weapon.*, SubclassPassive.*, CharacterState.*(HitReact/ADS/Reloading/Jumping/UsingInteractable/Health.Downed), Ability.*(Player.WeaponFire/MeleeAttack/Jump/Reload 등)
- WeaponCombo.Sword.Default.{N}: 콤보 공격 단계 정보

### 주요 분석 주의사항
- 서브클래스 태그 미표기 비율 48.8% (250건 중 122건) — 대표성 한계
- "다운 시점 무기" ≠ "주 사용 무기" — 인기도 vs 취약성 구분 불가
- SubclassPassive.DPS.WeaponDamage 태그 내 "Weapon" 문자열이 무기 태그 검색에 혼입 가능 — LIKE '%Weapon.Ranged%' 또는 '%Weapon.Melee%'로 정밀 필터링 필요

### 2026-04-14 — 플레이어 스폰 패턴과 맵 진행 퍼널 분석 (일곱 번째 연구)
- **리포트**: `reports/research/copperhead/player-spawn-map-progression.md`
- **데이터 기간**: 2026-02-04 ~ 2026-04-14
- **핵심 발견**:
  - 자동화 제외 후 1,795명, 6,599 스폰 이벤트, 21개 맵 관측
  - 37.4%(671명) 로비에서 미션 미진입, 이 중 69.7%(468명)가 단 1회 스폰 후 이탈
  - 로비→MIS_01 전환율 37.6%, MIS_01→MIS_02 전환율 22.4%로 급감
  - 2개 이상 미션 맵 방문 플레이어 5.7%(103명)에 불과
  - 미션 맵은 순차 구조가 아닌 독립 선택 구조 — MIS_04(128명) > MIS_03(87명)
  - 모든 맵 전환이 로비를 경유하는 허브-스포크(hub-and-spoke) 패턴
  - 리스폰 많을수록 성공률 높음(1회 10.6% → 4+ 55.4%) = 생존자 편향 (#4 연구 재확인)
  - 스폰 텔레메트리 커버리지 40.1% (sessionstart 4,480명 중 1,795명만 스폰 기록)
- **가설 판정**: H1 채택(로비 이탈 37.4%), H2 기각(리스폰↑=성공↑, 생존자 편향), H3 기각(비순차 구조)
- **상태**: 초안 완료, 검증/팀장 리뷰 대기

## cphplayerspawned 테이블 메타데이터 (상세)

### 스키마
- account_id, onlinesessionid, sessionid, mapname, playerlocation, playerrotation, playername, username, computername, buildversion, eventid
- queue_time: 맵 로드 시간 추정 (ms 단위, string), 중앙값 25~36초
- 모든 값 컬럼 string 타입 → CAST 필수

### 데이터 품질
- 전체 24,935+ 건, 자동화 제외 후 6,599건, 1,795 고유 계정
- 스폰 커버리지: 비자동화 sessionstart 4,480명 중 1,795명(40.1%)만 스폰 기록 존재
- 로비-only 671명 중 13명(1.9%)이 cphmissionstarted에 미션 시작 기록 있음 → 스폰 이벤트 100% 발화 보장 안됨
- 0초 리스폰 간격 54.4%: event_at이 초 단위 절삭이나 log_created_at에서 밀리초 차이 존재
- 21개 맵 관측(전체 기간): Lobby_Basic, Hub_Prototype, MIS_Proto2_2k_01~04, MIS_Proto2_2k_Sentinel, MIS_VS_2k_ArtPrimaryPOI, MIS_POI_Test_01, Gym_MetaContent, Gym_WeaponMods, WB_StaticGC, GYM_InteractTesting, GYM_FlyingAI, GYM_AI, GYM_Player, GYM_SpiderBossCity, POI_PRI5_DestroyFactory01_ART/ART2, POI_SEC4_Bunker02_GEN, MAP_ENV128M_VisTargetMS2

### 맵 분류
- **로비**: Lobby_Basic (플레이어 1,781명, 진입점)
- **허브**: Hub_Prototype (122명, 로비 다음 선택지)
- **미션(Proto2)**: MIS_Proto2_2k_01(669) > 02(150) > 04(128) > 03(87) > Sentinel(25)
- **기타 미션**: MIS_VS_2k_ArtPrimaryPOI, MIS_POI_Test_01
- **테스트/GYM**: Gym_*, GYM_*, WB_*, POI_*, MAP_* (소규모)

## 미탐색 테이블 현황 (7차 연구 시점)
### 탐색 완료
cphmissionstarted, cphmissionsucceeded, cphmissionfailed, cphplayerheartbeat, cphsessioncreated, cphsessionjoined, cphleavesessionevent, cphplayerspawned, cphplayerlogin, cphplayerdownedevent, cphcreatesessionevent, cphjoinsessionevent, cphplayerdowned, view_client_gpp_device_info, view_client_gpp_user_entry, sessionstart, sessionend

### 미탐색 (잔여 관심)
- view_lobby_concurrent_user: 758,737건 — 동시접속자 시계열 데이터
- view_lobby_user_heartbeat: 239,714건 — 로비 체류 heartbeat
- view_lobby_connected: 15,952건 — 로비 접속 이벤트 (analytics_id, device_id, duration 등)
- view_lobby_disconnected: 15,738건 — 로비 접속 종료 (duration, loggedin_at 포함)
- view_iam_logged_in: 14,265건, view_iam_registered: 14,020건 — IAM 로그인/등록
- view_client_gpp_token_refresh: 1,443건 — 토큰 갱신 이벤트
- au_base: 0건, concurrent_user: 0건, logged_in: 0건, registered: 0건 — 빈 테이블
- telemetry_session_start: 0건 — 빈 테이블
- connected/disconnected: 이전 확인 시 0건

### 2026-04-14 팀 상호 피드백 Round 1 (13건 기준)

**팀장 피드백 — 잘한 점:**
- 데이터 품질 이슈 발견 능력 우수 (미션 80.7% 누락, 자동화 60.3%, heartbeat 50.7% 누락)
- 미탐색 테이블(cphplayerspawned, cphplayerheartbeat 등) 적극 도전
- targettags 파싱으로 무기 분석 수행 등 창의적 데이터 활용
- REVISE 후 회복력: 성능 텔레메트리 REVISE → 수정 → 재검증 PASS, 이후 동일 유형 미재발

**팀장 피드백 — 개선할 점 (핵심):**
- **매 연구마다 다른 유형의 새 오류 발생** — 7건 7가지:
  1. 측정 지표 정의 혼동 (재시작율 vs 성공율)
  2. 쿼리 결과 전사 오류
  3. COUNT DISTINCT 오류, mean of ratios vs ratio of means
  4. NULL→'None' 변환 분류 오류
  5. 소계 산출 시 행 누락
  6. 요약-본문 수치 불일치, 가중 평균 산술 오류
  7. 분모 미명시, JOIN 세션 범위 불일치
- 공통 원인: **수치를 옮기고 집계할 때 원본과 대조하지 않음**
- 비율 제시 시 **분모 반드시 명시**: "37.4%" → "37.4% (468/1,251 세션)"

**셀프 체크리스트 (제출 전 반드시 확인):**
1. 핵심 수치 5개를 원본 쿼리 결과와 대조했는가?
2. 요약에 쓴 수치가 본문과 일치하는가?
3. 모든 비율에 분모(N=?)가 명시되어 있는가?
4. 소계/합계가 개별 행의 합과 일치하는가?
5. NULL 컬럼을 IS NULL/IS NOT NULL로 별도 처리했는가?

> 상세: `reports/research/team-feedback-round1.md`

### 2026-04-14 — 로비 동시접속 및 체류 패턴 분석 (여덟 번째 연구)
- **리포트**: `reports/research/copperhead/lobby-concurrent-user-dwell-pattern.md`
- **데이터 기간**: 2026-02-04 ~ 2026-04-14
- **핵심 발견**:
  - CCU 전체 평균 2.8명, 금요일 UTC 21시(PT 1pm) 평균 12.5~20.4명, 최대 44명
  - 10주 중 9주에서 금요일 피크 반복 — 정기 플레이테스트 패턴
  - 비자동화 로비 세션 7,412건(5,726명)의 체류 시간 이중 분포: 20.0% 30초 이내 이탈 + 19.4% 42~53분 집중
  - 42~53분 세션 중 80.6%가 미션 미참여 — 로비 세션 타임아웃 추정
  - 88.7% 사용자가 1회만 접속, 주말 CCU는 평일의 1/3~1/5
  - US IP에서 30초 이내 이탈 39.5%, KR IP에서 42~53분 체류 60.6% (단일 공유 기기 지배)
- **가설 판정**: H1 채택(금요일 피크 4~7배), H2 채택(이중 분포 확인), H3 부분 채택(장기 체류 미션 참여율이 중간 구간보다 낮으나 전체 평균과 비슷)
- **상태**: 초안 완료, 검증/팀장 리뷰 대기

## 로비 테이블 메타데이터

### view_lobby_concurrent_user
- 건수: ~1,971,028 (전체), 베타기간 383,237건(group IS NULL 기준)
- 스키마: ccu(double), event_at(timestamp), group(string), group_value(string), pod_name(string), event_date, event_hour
- **pod별 분할 기록**: 각 행은 개별 로비 서버 pod의 CCU. 전체 CCU = 동일 시점 pod 합산 필요
- group 종류: NULL(전체), platform(SteamStore only), device(PC only), country_code(US only), app_version(3,561종)
- pod 수: 전체 33개, 동시 활성 최대 5개
- 측정 주기: 1분(매 :30초)

### view_lobby_user_heartbeat
- 건수: 534,532 (전체), 240,515 (베타 기간)
- 스키마: connected_at(string), context(struct: analytics_id, app_version, device_id 등), event_at(timestamp), krafton_id, kos_user_id, pod_name
- heartbeat 주기: **50초** (정확)
- 사용자 식별: krafton_id(globalaccount UUID), kos_user_id(sessionstart의 account_id와 동일 형식)
- 비자동화 기기: 4,071명(heartbeat), 5,726명(connected). heartbeat 커버리지 71.1%

### view_lobby_connected
- 건수: 49,925 (전체), 16,004 (베타 기간)
- 스키마: analytics_id, device_id, krafton_id, kos_user_id, model(하드웨어 모델), country_code_ip, app_version 등
- 자동화 8,480건(53.0%), 비자동화 7,524건(47.0%)
- reason 컬럼: 98.5% None, 1.5% au_base
- computername 없음 — device_id로 자동화 식별 필요

### view_lobby_disconnected
- 건수: 49,311 (전체), 15,791 (베타 기간)
- 스키마: connected와 동일 + duration(double, 초), loggedin_at(string)
- connected-disconnected 페어링: 비자동화 analytics_id 기준 99.98% 매칭
- 비자동화 7,412건, 자동화 8,380건

### 자동화 기기 device_id 매핑
| computername | device_id |
|-------------|-----------|
| SDS-PFLIGHT-10 | fb74c741db9c3b720f1740657ec1e5b0 |
| SDS-PFLIGHT-05 | 04b153914acd931ee10a4fea61b1f747 |
| SDS-RECSPEC-CPH | c5d9c80e9c01dce06228adb1e74610b5 |
| SDS-MINSPEC-CPH | 2ebcf8687bbca35262d26876956898af |
| SDS-PFLIGHT-08 | 1914568039d5103b7b25025e9e5764ba |
| SDS-PFLIGHT-03 | 455699d6bc8a2ebf95a0c3eeee11a326 |
| SDS-PFLIGHT-11 | 43c20f9f55e08b19afebc0cda57b49a0 |

### 조인 키 정리
- lobby 테이블 ↔ sessionstart: `kos_user_id = account_id`
- lobby 테이블 간: `analytics_id` (세션), `krafton_id` (사용자), `device_id` (기기)
- **krafton_id (globalaccount.UUID)와 account_id (hex) 형식이 다름** — 직접 조인 불가, kos_user_id 사용

## 미탐색 테이블 현황 (8차 연구 시점)
### 탐색 완료
cphmissionstarted, cphmissionsucceeded, cphmissionfailed, cphplayerheartbeat, cphsessioncreated, cphsessionjoined, cphleavesessionevent, cphplayerspawned, cphplayerlogin, cphplayerdownedevent, cphcreatesessionevent, cphjoinsessionevent, cphplayerdowned, view_client_gpp_device_info, view_client_gpp_user_entry, sessionstart, sessionend, **view_lobby_concurrent_user**, **view_lobby_user_heartbeat**, **view_lobby_connected**, **view_lobby_disconnected**

### 미탐색 (잔여 관심)
- view_iam_logged_in: 14,265건 — IAM 로그인 이벤트
- view_iam_registered: 14,020건 — IAM 등록 이벤트
- view_client_gpp_token_refresh: 1,443건 — 토큰 갱신
- au_base: 0건, concurrent_user: 0건 — 빈 테이블

## 2026-04-14 8차 연구(로비 CCU·체류 패턴) 검증 피드백 반영
- **검증 결과**: MINOR (필수 5건 + 권고 3건 → 전수 반영)
- **수정 내역**:
  1. 섹션 2 자동화 비율에 "(connected 기준)" 출처 명시, 섹션 4.2 자동화 비교 테이블에 "(disconnected 기준)" 명시
  2. 금요일 주차별 테이블 02-13 최대 CCU: 44 → 42 (hour=21 기준 정정, 44는 hour=22 값)
  3. "9주 중 8주" → "10주 중 9주" (실제 금요일 10주 존재)
  4. 체류 시간 분포 테이블 구간 정리: "0분 미만(30초 이내)" → "30초 미만", 누락된 "30초~2분" 구간(1,196건) 추가 (섹션 4.3과 일관성 확보)
  5. sessionstart 매칭 수치: 345명(26.4%) → 310명(23.7%), 나머지 961명(73.6%) → 996명(76.3%)
  6. 섹션 6.1 소제목 단정 완화 + 본문 [Estimate] 태그 추가 (외부 스케줄 미확인)
  7. "53분 초과" 세션 수: 101 → 100 (경미, 4.2/4.3 양쪽 반영)
  8. "10~42분" 세션 수/미션 참여: 1,377/334 → 1,378/335 (경미)
- **파급 수정**: 요약(최대 44→42), 반증(22~44→22~42), 결론(20~44→20~42, 73.6%→76.3%), 전체 미션 합계(1,383→1,384)
- **교훈**:
  - 테이블에 데이터 출처(어떤 테이블 기준인지)를 항상 명시할 것 (connected vs disconnected)
  - 히스토그램 구간 설계 시 합계가 전체 N과 일치하는지 반드시 검산할 것 (누락 구간 주의)
  - hour 필터가 정확한지 원본 쿼리에서 재확인할 것 (인접 hour 값 혼동 위험)
  - 조인 방식에 따라 매칭 수가 달라질 수 있음 — 복수 방식으로 크로스체크 권장

<!-- 이후 작업 기록은 아래에 자동 추가됨 -->
