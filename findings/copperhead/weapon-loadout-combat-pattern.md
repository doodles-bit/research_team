# Copperhead 무기 선택 패턴 — AR/Sword/DMR 3종이 64.4%, 근접 교전 시 경직 취약

## 인사이트
Copperhead 베타 전투 다운 293건(극단값 1인 제외)에서 9종 무기가 관측되었으며, AR(25.2%)·Sword(21.6%)·DMR(17.6%) 상위 3종이 64.4%를 차지한다. 근접 교전 상황에서 피격 경직(HitReact) 동반 다운 비율이 높고(Sword 52.1%, DMR 51.4% vs AR 32.7%), 이는 근접 무기 자체보다 근접 교전 거리의 위험성이 원인으로 추정된다. 서브클래스별 역할-무기 연계가 관측되며(DPS-LMG/AR, Tank-Sword/Shotgun, Support-DMR), Tank의 근접 편중(85.7%)이 경직 취약성과 결합하면 생존성 이슈로 이어질 가능성이 있다.

## 근거
- 357건 다운 이벤트, AAntle(64건, 17.9%) 제외 후 293건, 무기 태그 보유 250건 분석 (`cphplayerdowned.targettags`, 2026-02-04~04-10, 자동화 필터링 적용)
- AR 63건(25.2%), Sword 54건(21.6%), DMR 44건(17.6%) — 상위 3종 64.4%
- Sword HitReact 52.1%(25/48), DMR 51.4%(18/35) vs AR 32.7%(18/55) — 근접 교전 상황이 공통 원인으로 추정
- Sword 공격 중 역습 다운 13건(27.1%), 경직 연쇄 다운 19건(39.6%)
- 서브클래스-무기 연계: DPS LMG+AR 83.3%, Tank Sword+Shotgun 85.7%, Support(AbilityRegen) DMR 72.7% — 단, 서브클래스 태그 보유 128건(51.2%)만 분석 가능
- SMG(4건), Heavy(5건)는 거의 미사용

## 활용 제안
- **근접 교전 밸런스**: Sword·DMR 모두 근접 거리에서 경직 취약 — 근접 교전 시 경직 회복 시간 또는 슈퍼아머 메커니즘 검토
- **SMG/Heavy 매력도 점검**: 9종 중 2종이 거의 미사용(합산 9건). 무기 밸런스 또는 해금 접근성 문제인지 확인 필요
- **Tank 생존성 모니터링**: Tank의 Sword+Shotgun 85.7% 편중 + 경직 취약성 조합이 "Tank가 가장 많이 다운되는 역할" 가능성 내포 — 역할별 다운 빈도 추적 필요
- **Sword 공격 중 역습 보호**: 27.1%가 근접 공격 도중 역습 패턴 — 공격 모션 중 피격 감소 등 검토

## 원본
- 연구 리포트: `reports/research/copperhead/weapon-loadout-combat-pattern.md`
- 검증 리포트: `reports/research/copperhead/weapon-loadout-combat-pattern-verification.md`
- 판정: PASS (검증원 MINOR → 수정 → 팀장 PASS)
- 주의: 본 분석은 "다운 시점의 무기"이므로 전체 게임에서의 무기 사용 비율과 다를 수 있음. 서브클래스 분석은 전체의 51.2%만 대상. 표본 293건(AAntle 제외)으로 하위 무기는 통계적 검정 불가.
