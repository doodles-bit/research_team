# Copperhead 미션 성공/실패 패턴 — 데이터 품질 80.7% 누락 + Defend Mission 9.1% 성공률

## 인사이트
Copperhead 베타 미션 텔레메트리의 80.7%가 결과 미기록(시작 9,111건 중 결과 1,757건만 기록)이라 정밀 밸런스 분석이 불가하다. 기록된 데이터에서 Defend Mission Normal의 성공률은 9.1%로 극단적 이상치이며, Very Easy 난이도가 Normal보다 소요 시간이 길어지는 역전 현상이 대부분의 미션에서 관찰된다.

## 근거
- 428명, 9,111건 미션 시작 / 1,121건 성공 / 636건 실패 분석 (`main.log_copperhead_beta`, 2026-02-04~03-24)
- 미션 결과 누락률: 80.7% (7,354/9,111건), Mission 1 기준 73.4%(314/428명)
- Defend Mission Normal: 성공 1건 / 실패 10건 = 9.1% (목표 달성 수 항상 0, 메커니즘 이슈 가능)
- 난이도 역전: Very Easy 소요 중앙값 > Normal 소요 중앙값 (대부분 미션에서 관찰)
- [가설 기각] 실패 후 재도전 비율 35%(14/40) — "대부분 재도전 안 함" 가설 기각
- 미션명 미기록 86.5%(7,883건 NULL/빈 값)

## 활용 제안
- **텔레메트리 누락 긴급 수정**: 80.7% 누락은 데이터 기반 의사결정의 근본적 장애 — 미션 종료 이벤트 발생 로직 점검 필요
- **Defend Mission 메커니즘 검증**: 성공률 9.1%, 목표 달성 수 항상 0 — 의도된 설계인지 버그인지 확인 필요
- **난이도 역전 점검**: Very Easy > Normal 소요 시간 패턴이 콘텐츠 볼륨 차이(VE가 더 길다) 때문인지 확인
- **미션명 로깅 보완**: 86.5% 미기록은 미션별 분석을 근본적으로 제한

## 원본
- 연구 리포트: `reports/research/copperhead/mission-success-failure-pattern.md`
- 검증 리포트: `reports/research/copperhead/mission-success-failure-pattern-verification.md`
- 판정: PASS (검증원 MINOR → 수정 → 팀장 PASS)
- 주의: 데이터 품질 80.7% 누락으로 모든 수치는 "기록된 데이터 한정"
