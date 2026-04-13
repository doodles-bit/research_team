# Copperhead 성능 텔레메트리 — 맵별 FPS 2.66배 차이, 4+명 세션 시 -33% 하락

## 인사이트
Copperhead 베타의 heartbeat 텔레메트리에서 맵 간 FPS 차이가 최대 2.66배(MIS_VS_2k_ArtPrimaryPOI 64.2 vs MIS_Proto2_2k_01 170.7)에 달하며, 4명 이상 동시 접속 시 FPS가 솔로 대비 -33.1% 하락한다. 세션 10~20분 경과 시점에서 시작 대비 31% FPS 하락도 관찰된다.

## 근거
- 190,902건 heartbeat 중 신뢰 가능 구간(4/10 이후) 52,873건, 339명 분석
- FPS/Ping 누락률 50.7% — 단, 시기별 차이(3/31 유효율 1.0% → 4/10 이후 100%). 텔레메트리 수정이 단계적 적용
- 맵별 FPS: MIS_VS_2k_ArtPrimaryPOI 64.2 (최저) ~ MIS_Proto2_2k_01 170.7 (최고)
- 플레이어 수별 FPS: Solo 105.4 → 2명 96.3 → 3명 84.3 → 4+ 70.5
- 세션 경과 FPS: 0-1분 109.6 → 10-20분 76.0 (-31%)
- Listen Server(호스트)의 30FPS 미만 비율이 전체의 89% — 호스트 부하 집중

## 활용 제안
- **맵 최적화 우선순위**: MIS_VS_2k_ArtPrimaryPOI(64.2 FPS), MIS_Proto2_2k_02(73.9 FPS, 30미만 5.8%) 우선
- **멀티 플레이어 성능**: 4+명 세션 시 -33% 하락 — 서버/네트워크 아키텍처 점검
- **세션 경과 하락**: 10-20분 구간 -31% — 메모리/오브젝트 관련 원인 조사 필요
- **호스트 부하**: Listen Server 30FPS 미만 89% 집중 — 호스트 부하 분산 또는 전용 서버 검토

## 원본
- 연구 리포트: `reports/research/copperhead/performance-telemetry-analysis.md`
- 검증 리포트: `reports/research/copperhead/performance-telemetry-analysis-verification.md`
- 판정: PASS (검증원 REVISE → 수정 → 재검증 PASS → 팀장 PASS)
