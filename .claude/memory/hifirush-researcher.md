# Hi-Fi Rush 연구원 (하이파이R) — Memory

> Hi-Fi Rush 유저 행동 데이터 탐색·연구 전담.
> 자율적으로 데이터를 탐색하고 가설을 세우고 검증하여 인사이트를 발굴한다.

---

## 최우선 원칙
1. 데이터 기반 팩트만 말한다 (할루시네이션 금지)
2. 확증 편향 경계 — 가설을 세우면 반증도 함께 탐색
3. 가설 기각은 실패가 아니라 발견

## 분석팀 축적 지식 (참고)
- 분석팀이 완료한 Hi-Fi Rush 분석: 플레이스타일 전환, 이탈 퍼널, 아이템 구매, 플랫폼 비교, 국가별 트렌드
- 분석팀 repo: `C:\Users\doodles\project_repo\ai_agent_team`

## 데이터 파일 참조

| 파일 | 행 수 | 주요 컬럼 | 비고 |
|------|-------|----------|-----|
| `2025_10_HiFi_Rush_Review_stage_completion.csv` | 1,248 | stage_id, stage_name, platform, battle_id, difficulty, player_count | 전투별 완료 인원 |
| `2025_10_HiFi_Rush_Review_attacks_used.csv` | 19,288 | difficulty, stage_id, battle_id, attack_name, attack_category, total_attack_count, total_battle_count | 공격 사용 통계 |
| `2025_10_HiFi_Rush_Review_bought_item_ability.csv` | 6,055 | difficulty, stage_id, item_name, item_type, total_transaction_count | 아이템 구매 |
| `2025_10_HiFi_Rush_Review_percent_country_users.csv` | 1,021 | date, platform, country_code_2, percent_aquired | 국가별 유저 비율 |

- 파일 위치: `C:/Users/doodles/project_repo/ai_agent_team/hifirush/logs/`
- 조인 키: `stage_id + battle_id + difficulty` (stage_completion과 attacks_used 간)
- stage_completion 416키 전체가 attacks_used에 존재 (10개 키는 attacks_used에만 있음: TR03_CR07, TR04_CR07)
- 배틀 유형: CR(Combat Round) = 본편 전투, SR(Side Round) = 선택 콘텐츠 (완료 인원 본편의 1~3%)
- 스테이지: 12개 트랙 + SpectraHub + RhythmTower, CR 배틀 73개

## 연구 이력

| 날짜 | 주제 | 가설 | 결과 | 리포트 |
|------|------|------|------|--------|
| 2026-04-15 | 배틀 단위 이탈 스파이크와 플레이어 행동 변화 | (1) 스테이지 내 전투 간 이탈 편차 크다 (2) 이탈 높은 전투에서 공격 다양성 낮다 (3) 난이도별 병목 전투 다르다 | (1) 부분 지지: Normal 7/10 스테이지 편차 <5pp, Track01/03만 >10pp. 고난이도에서 급증 (2) 기각: 스테이지 통제 시 r=0.003 (3) 강하게 지지: Normal/Master top5 겹침 0건 | `reports/research/hifirush/2026-04-15-battle-dropout-spike-analysis.md` |

### 주요 발견 요약
- TR01_CR05가 전 난이도 공통 최대 이탈 지점 (Normal 15.4%, 45.5만명 이탈)
- CR05에서 Powerchord(필살기) 사용이 처음 대량 발생 (0.2% -> 9.3%)
- Other Platforms 이탈률이 Steam보다 6.1pp 높음 (17.1% vs 11.0%)
- Master 난이도는 후반부(T06~T10)에서 집중 이탈, Normal은 T01 초반에 집중

## 데이터 품질 주의사항

1. **player_count 정의 불확실**: 고유 인원인지 완료 횟수인지 불명확. 일부 전투에서 음수 이탈(직전 전투보다 완료 인원 증가)이 5개 난이도 전체에서 관찰됨 (TR07_CR10, TR03_CR06 등). 리플레이를 포함한 집계일 가능성 높음.
2. **total_battle_count vs player_count 불일치**: attacks_used의 total_battle_count와 stage_completion의 player_count 비율이 0.81~1.12 범위로 변동. 두 지표의 정확한 정의 확인 필요.
3. **TR03_CR07, TR04_CR07**: attacks_used에만 존재하고 stage_completion에 없음. 이 전투들의 완료 데이터 누락.

<!-- 이후 작업 기록은 아래에 자동 추가됨 -->
