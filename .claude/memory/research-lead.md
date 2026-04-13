# 연구 팀장 — Memory Log

> 연구 팀장의 연구 이력, 검토 결과, 피드백, 지식 전파 기록을 축적하는 메모리.
> 디테일은 연구원에게 확인하고, 여기에는 요약만 기록한다.

---

## 핵심 기억 (상시 참조)

### 팀 구성
| 별칭 | 에이전트 | 역할 |
|------|----------|------|
| 팔모R | palm-researcher | PalM 데이터 탐색·연구 |
| 하이파이R | hifirush-researcher | Hi-Fi Rush 데이터 탐색·연구 |
| 코퍼헤드R | copperhead-researcher | Copperhead 데이터 탐색·연구 |
| 연구 검증 | research-verifier | 수치 검증, 가설 논리 검토, 확증 편향 체크 |

### 반복 주의사항
- 확증 편향 경계: 가설에 유리한 데이터만 찾지 않았는지 반드시 체크
- 실무 연관성 검증: "스튜디오가 이걸 읽고 뭘 할 수 있는가?" 자문
- 리포트 독자는 비전문가 — 전문 용어 쉬운 표현 병기

### 분석팀 연동
- 분석팀 repo: `C:\Users\doodles\project_repo\ai_agent_team`
- 전파 경로: `findings/{game}/` → 분석 팀장이 읽어서 shared-context 반영
- 피드백 경로: `feedback/` 에 파일 작성 or git commit

---

## 최근 연구 요약

### 2026-04-13 [PalM] dungeon-01 실패 후 재도전 행동 분석 — 팔모R
- **주제**: 분석팀이 발견한 dungeon-01 난이도 벽(사망률 55~62%)이 좌절인지 건강한 도전인지
- **결과**: 가설 기각. 연속 실패할수록 재도전율 상승(37.3%→59.3%), 사망→이탈 2.6%(3명), 6회+ 시도자 클리어율 94.9%
- **검증**: 검증원 독립 검증 MINOR → 팔모R 수정 반영 → 팀장 재판정 **PASS**
  - 필수 수정 3건: 클리어율 산술오류, Lv33+ 누락데이터, streak 로직 오류
  - 권고 수정 4건: 인과 표현 완화, 차트 추가, 이탈 기준 명확화
  - 검증 리포트: `reports/research/palm/dungeon-01-retry-behavior-verification.md`
- **외부 피드백**: 분석 팀장 피드백 반영 완료 (v3)
  - 긍정: 주제 선정, 가설 기각 솔직 보고, 반증 탐색, 해석A/B 비교표
  - 수정: 요약에 비교 테이블 축약본 추가
  - 교훈: "요약 = 핵심 전달 장치" — 다음 리포트부터 요약에 결론 전달 장치 포함할 것
- **지식 전파**: `findings/palm/dungeon-01-healthy-wall.md` 작성 완료
- **리포트**: `reports/research/palm/dungeon-01-retry-behavior.md`
- **팀원 오류 패턴**: 팔모R — (1) 산술 크로스체크 미흡(95.9%→94.9%), (2) 쿼리 범위 누락(Lv37~46), (3) streak 로직 오류. 향후 쿼리 결과 산술 재검증 습관 필요
- **전체 완료**: Phase 1~6 완료

### 2026-04-13 [PalM] 세션 패턴 및 플레이 리듬 분석 — 팔모R
- **주제**: 알파테스트 7일간 세션 수준의 행동 패턴 (분석팀 미탐색 영역)
- **결과**: 가설 3건 모두 채택. Day3-4 몰입도 급락(116.5→40.0분, -66%), 첫 세션 길이와 잔존 상관(이탈 10.5분 vs 잔존 32.8분), 저녁 세션 +42.5% 길음
- **검증**: 검증원 MINOR → 팔모R 수정 → 팀장 **PASS**
  - 수정 3건: 시간대별 중앙값 오차, Day6 모수 불일치, 반증 섹션 수치 오류
- **외부 피드백**: 분석 팀장 피드백 반영 완료 (v3)
  - Day3-4 콘텐츠 소진 가설에 레벨/퀘스트 데이터 교차 보강
  - 제작 이벤트 86% 비중 경고를 본문에 추가
- **지식 전파**: `findings/palm/session-play-rhythm.md` 작성 완료
- **리포트**: `reports/research/palm/session-play-rhythm.md`
- **전체 완료**: Phase 1~6 완료

### 2026-04-13 [PalM] 팰 포획 행동 패턴 분석 — 팔모R
- **주제**: 핵심 게임 루프인 팰 포획의 행동 패턴 (분석팀 미탐색)
- **결과**: 가설 3건 (채택/채택/부분채택). 포획 다양성-잔존 상관(36.1%→82.8%), HP 75%+ 전원 실패(9,836건 UX 이슈), 상위 10종 54.3% 집중
- **검증**: 검증원 MINOR → 팔모R 수정 → 팀장 **PASS**
  - 수정 3건: 다양성별 유저 수 불일치, "100종" 표현(실제 52종), 반증1 유저 수
- **외부 피드백**: 분석 팀장 피드백 반영 완료 (v3)
  - FlowerRabbit 파밍 이유 연결 (73.4%가 단련 재료 소비)
  - 반증2 n=18 한계를 결론/시사점에서 더 솔직히 반영
- **지식 전파**: `findings/palm/pal-capture-behavior.md`
- **리포트**: `reports/research/palm/pal-capture-behavior.md`
- **전체 완료**: Phase 1~6 완료

### 2026-04-13 [PalM] 경제/자원 흐름 패턴 분석 — 팔모R
- **주제**: 자원 수집-제작-소비 흐름 (분석팀 미탐색)
- **결과**: 가설 2건 채택, 1건 기각. 기지 자원 Day3 역전(20.4%→69.3%), 음식 제작 73.6%(Baked Berries 70.7%), D1 기지→잔존 효과는 레벨 통제 시 소멸
- **검증**: 검증원 **PASS** (경미 표기 수정 7건 권고, 결론 무영향) → 팀장 **PASS**
- **지식 전파**: `findings/palm/economy-resource-flow.md`
- **리포트**: `reports/research/palm/economy-resource-flow.md`
- **특기**: 팔모R 첫 검증 즉시 PASS — 이전 오류 패턴(산술, 범위 누락) 개선됨
- **전체 완료**: Phase 1~4, 6 완료 (Phase 5 스킵)

### 2026-04-13 [PalM] 콘텐츠 소비 순서 분석 — 팔모R
- **주제**: 유저 여정(journey) — 콘텐츠 경험 순서, 레벨별 비중 변화, 이탈 직전 활동
- **결과**: 가설 3건 (채택/조건부채택/채택). 71.4%가 동일 첫 경험 시퀀스, Lv1-5 이탈 51.7%(핵심 콘텐츠 미경험), 이탈 유저 포획 비율 3.9배 높음
- **검증**: 검증원 **PASS** (1명 차이 수준 경미 오차만) → 팀장 **PASS**
- **지식 전파**: `findings/palm/content-consumption-sequence.md`
- **리포트**: `reports/research/palm/content-consumption-sequence.md`
- **특기**: 2연속 검증 즉시 PASS. 기존 4건 연구를 유저 여정으로 통합하는 메타 분석 성격
- **전체 완료**: Phase 1~4, 6 완료 (Phase 5 스킵)

---

## 대기 항목

- **Phase 7 회고 #1 완료** (2026-04-13): 5건 교차 분석 수행
  - 핵심 발견: 고다양성 유저(잔존율 82.8%)의 세션 감소가 가장 급격(-76.6%) — 잔존 수치에 가려진 활동 강도 급감
  - 회고 리포트: `reports/research/palm/retrospective-01-cross-analysis.md`
### 2026-04-13 [PalM] 멀티레이드 참여 행동 분석 — 팔모R
- **주제**: 유일한 협동 콘텐츠인 멀티레이드 참여 패턴 (탐색적 분석)
- **결과**: 가설 3건 (기각/채택/부분채택). 참여→잔존 효과 기각(레벨 통제 시 소멸), 86.9% 솔로, R2 사망률 급증
- **검증**: 검증원 MINOR (퍼널 구조 10명 누락) → 팔모R 수정 → 팀장 **PASS**
- **지식 전파**: `findings/palm/multiraid-participation.md`
- **리포트**: `reports/research/palm/multiraid-participation.md`
- **전체 완료**: Phase 1~4, 6 완료 (Phase 5 스킵)

**PalM 알파테스트 미탐색 영역 소진 — Copperhead로 전환**

### 2026-04-13 [Copperhead] 미션 성공/실패 패턴 분석 — 코퍼헤드R
- **주제**: 베타 미션 텔레메트리 품질 + 미션별 성공/실패 패턴 (연구팀 첫 Copperhead 연구)
- **결과**: 가설 3건 (채택/부분기각/기각). 데이터 누락 80.7%, Defend Mission 9.1% 이상치, 재도전율 35%로 가설 기각
- **검증**: 검증원 MINOR (H3 판정 오류, 산술 1건) → 코퍼헤드R 수정 → 팀장 **PASS**
- **지식 전파**: `findings/copperhead/mission-success-failure-pattern.md`
- **리포트**: `reports/research/copperhead/mission-success-failure-pattern.md`
- **팀원 오류 패턴**: 코퍼헤드R — 재시작 비율 vs 재시작 후 성공 비율 혼동. 측정 지표 정의 정확히 확인 필요
- **전체 완료**: Phase 1~4, 6 완료

### 2026-04-13 [Copperhead] 성능 텔레메트리(heartbeat) 분석 — 코퍼헤드R
- **주제**: FPS/Ping 분포, 맵별·플레이어수별 성능, 세션 경과 하락
- **결과**: 가설 3건 (채택/기각/채택). 맵별 FPS 2.66배, 4+명 -33%, 세션 10-20분 -31% 하락
- **검증**: 검증원 **REVISE** (날짜별 건수 오류, 3/30 유효율 오류, 세션 FPS 오류, SDS 기기 수) → 코퍼헤드R 수정 → 재검증 **PASS** → 팀장 **PASS**
- **지식 전파**: `findings/copperhead/performance-telemetry-analysis.md`
- **리포트**: `reports/research/copperhead/performance-telemetry-analysis.md`
- **팀원 오류 패턴**: 코퍼헤드R — (1) 날짜별 집계 쿼리 결과 전사 오류, (2) 유효율 일반화 오류. 쿼리 결과를 리포트에 옮길 때 원본 대조 필수
- **전체 완료**: Phase 1~4, 6 완료

### 2026-04-13 [Copperhead] 세션 생명주기와 멀티플레이어 접속 품질 분석 — 코퍼헤드R
- **주제**: 세션 참여 실패 패턴, 빌드별 실패율, 솔로/멀티 비율 (세션 테이블 미탐색 영역)
- **결과**: 가설 3건 (부분채택/채택/채택-인과불확정). 참여 실패 12.1%(첫시도 3.5%), 76명에 100% 집중, 빌드별 0~34.8%, 솔로 77.2%
- **검증**: 검증원 MINOR (Success 고유 계정 911→997, 평균 실패율 표기 혼동) → 코퍼헤드R 수정 → 팀장 **PASS**
- **지식 전파**: `findings/copperhead/session-lifecycle-multiplayer-quality.md`
- **리포트**: `reports/research/copperhead/session-lifecycle-multiplayer-quality.md`
- **팀원 오류 패턴**: 코퍼헤드R — (1) 고유 계정 수 COUNT DISTINCT 오류(911→997), (2) mean of ratios vs ratio of means 혼동. 통계량 정의 확인 필요
- **전체 완료**: Phase 1~4, 6 완료

### 2026-04-13 [Copperhead] 플레이어 다운·전투 패턴 분석 — 코퍼헤드R
- **주제**: 전투 중 다운 원인, 적 유형별 위협도, 경직 연쇄, 다운-미션 결과 관계
- **결과**: 가설 3건 (채택/채택/기각). Trooper 45.6%, 경직 동반 41.3%, 다운 많을수록 성공(생존자 편향)
- **검증**: 검증원 MINOR (NULL instigator 분류 오류, 자해 정합성, HitReact 분류, 맵 누락) → 코퍼헤드R 수정 → 팀장 **PASS**
- **지식 전파**: `findings/copperhead/player-down-combat-pattern.md`
- **리포트**: `reports/research/copperhead/player-down-combat-pattern.md`
- **팀원 오류 패턴**: 코퍼헤드R — CAST(NULL AS STRING)→'None' 변환으로 인한 NULL 처리 오류. IS NULL 조건 별도 처리 필수
- **전체 완료**: Phase 1~4, 6 완료

- **Phase 7 회고 #2 완료** (2026-04-13): Copperhead 4건 + PalM 1건 교차 분석
  - 핵심 발견: Copperhead "사실상 솔로 게임" (77.2% 솔로 + PvP 0건), 빌드 버전이 데이터/접속 품질 전방위 영향, PalM·Copperhead 공통 "코옵 설계→솔로 소비" 패턴
  - 코퍼헤드R 오류 패턴 추이: 매 연구 다른 유형 오류 → 수치 정의/계산 과정 명시 가이드 필요
  - 회고 리포트: `reports/research/copperhead/retrospective-02-cross-analysis.md`

**누적 10건 완료**

---

## 아카이브 인덱스

(아카이브 분리 시 여기에 인덱스 기록)

<!-- 이후 작업 기록은 아래에 자동 추가됨 -->
