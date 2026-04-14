# Copperhead 베타 환경 — 60.3%가 자동화 테스트 계정, 원격 접속 60.4%

## 인사이트
Copperhead 베타 데이터의 60.3%(11,285명 중 6,806명)가 7대의 자동화/공유 테스트 머신(PFLIGHT, MINSPEC, RECSPEC)에서 생성된 계정이다. 이 머신들은 기기당 800~1,395명의 일회용 계정을 운용하며, 세션 종료 누락률이 50~85%에 달한다. 기기의 60.4%(134대 중 81대)가 Parsec 등 원격 접속 어댑터를 사용하며, 물리 GPU 기기는 RTX 4090·3080 Ti 등 고사양 워크스테이션이다. 로그인 성공률은 98.7%로 안정적이다.

## 근거
- 134대 기기, 11,285 계정, 11,793 세션 분석 (`view_client_gpp_device_info`, `view_client_gpp_user_entry`, `sessionstart`/`sessionend`, 2026-02-04~)
- 자동화 머신: PFLIGHT 5대(4,023명), MINSPEC 1대(1,388명), RECSPEC 1대(1,395명) = 합계 6,806명(60.3%)
- 세션 종료 누락: PFLIGHT 85.4%, MINSPEC 53.1%, RECSPEC 50.6% vs SDS Dev/BBI/External 11~13%
- 원격 접속: Parsec 75대 + MS Remote 4대 + Virtual Monitor 1대 + DisplayLink 1대 = 81대(60.4%)
- 로그인 성공 98.7%, 실패 104건 모두 "HttpResponse is null"

## 활용 제안
- **자동화 테스트 필터링 기준**: `computername`이 `*PFLIGHT*`, `*MINSPEC*`, `*RECSPEC*`인 세션을 분리하면 "실제 사용자" 행동만 추출 가능
- **미션 누락률 재검증**: 1차 연구의 80.7% 미션 텔레메트리 누락 중 상당 부분이 자동화 테스트 세션에서 발생. 필터링 후 실제 누락률 확인 필요
- **성능 데이터 해석 주의**: 60.4%가 원격 접속이므로, FPS 이상치의 일부는 원격 렌더링 환경에서 기인할 수 있음
- **세션 이탈률 보정**: 자동화 머신 분리 없이는 세션 종료 누락률이 과대평가됨

## 원본
- 연구 리포트: `reports/research/copperhead/device-environment-login-flow.md`
- 검증 리포트: `reports/research/copperhead/device-environment-login-flow-verification.md`
- 판정: PASS (검증원 MINOR → 수정 → 팀장 PASS)
- 주의: computername 기반 분류이므로 기기 이름 변경/비표준 시 누락 가능. `cph*` 이벤트 테이블에는 computername이 없는 경우도 있어 적용 범위 확인 필요.
