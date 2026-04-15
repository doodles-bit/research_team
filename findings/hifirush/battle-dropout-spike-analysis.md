## 배틀 단위 이탈 스파이크와 플레이어 행동 변화

- **인사이트**: Track01 CR04→CR05 전투가 전체 게임 최대 이탈 병목이다. Normal 난이도에서 45.5만 명(15.4%)이 이 한 전투에서 이탈하며, 이 전투에서 Powerchord(필살기) 사용률이 0.2%→9.3%로 급등한다. 이탈은 공격 패턴의 단순함이 아니라 특정 전투의 난이도 설계에 기인하며(교란 통제 후 상관 r=0.003), 난이도별로 병목 전투가 완전히 다르다(Normal 상위 5 vs Master 상위 5 겹침 0건).
- **근거**:
  - `stage_completion.csv` 73개 CR 전투의 player_count 감소율 분석, `attacks_used.csv` 교차
  - Normal TR01_CR05: 2,950,981→2,495,434명(15.4% 이탈). Other Platforms 17.1% vs Steam 11.0%(+6.1pp)
  - 공격 다양성-이탈 상관: 전체 r=0.573이나 스테이지 통제 후 r=0.003(교란 변수)
  - Master 상위 5 이탈 전투: Track06~10 후반부에 분산(Normal은 Track01 초반에 집중)
- **활용 제안**:
  - TR01_CR05 전투의 튜토리얼/가이드 충분성 검토 — 특히 Other Platforms(게임패드) 조작 안내
  - Master 난이도 후반부(Track10 CR09: 17.4% 이탈) 밸런스가 의도된 수준인지 확인
  - 이탈 원인 분석 시 공격 패턴이 아닌 전투 설계에 초점을 맞출 것
- **원본**: reports/research/hifirush/2026-04-15-battle-dropout-spike-analysis.md
