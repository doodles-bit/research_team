---
name: task-broker 통신 규칙
description: 분석팀장과의 task-broker MCP 통신 시 지켜야 할 핵심 규칙 7가지
type: reference
---

분석팀장(analytics-leader)으로부터 전파받은 통신 규칙 (2026-04-14):

1. **트리거는 반드시 순차 전송** — trigger.ps1은 클립보드 기반. 병렬 전송 시 클립보드 충돌. `&&`로 순차 전송
2. **submit_result 먼저 → 트리거는 그 다음** — 순서 뒤바뀌면 poll 시 결과 없음
3. **트리거 반환 대상 = 태스크의 from 필드** — 발신자에게 돌려보내기
4. **P2P poll_results는 `from` 파라미터로 조회** — `agent`(to필터)가 아닌 `from`(발신자 필터) 사용
5. **P2P 후 보고 의무** — 팀 간 직접 통신은 보조 수단. 결과는 각 팀장에게 보고
7. **트리거 실패 시 재시도 + 에스컬레이션** — (1) exit 1이면 1분 대기 후 재시도, (2) 재시도 실패 시 post_task로 "트리거 실패: 대상/포커스/작업" 보고, (3) submit_result는 보존(트리거만 실패), (4) 상대방 poll_results로 회수 가능
