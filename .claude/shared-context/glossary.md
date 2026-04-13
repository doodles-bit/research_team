# 공통 용어 및 지표 정의

> 모든 팀원 에이전트가 참조하는 공통 용어집.
> 새로운 용어나 지표가 정의되면 여기에 추가한다.

## 공통 지표
| 지표 | 정의 | 비고 |
|------|------|------|
| DAU | Daily Active Users, 일일 활성 사용자 수 | 로그인 기준 |
| GAU | Game Active Users, 인게임 로그인 기준 활성 사용자 | PalM에서 사용 |
| MAU | Monthly Active Users, 월간 활성 사용자 수 | |
| Retention D1/D7/D30 | 설치 후 1/7/30일째 복귀율 | |
| 이탈 (Churn) — 모바일 | 연속 14일 이상 미접속 | KRAFTON 모바일 게임 공식 기준. 14일 이상 미접속 유저의 80%+ 미복귀 패턴 기반. 테스트 환경(7일 등)에서는 기간 제약으로 별도 정의 필요 |
| ARPU | Average Revenue Per User | |
| ARPPU | Average Revenue Per Paying User | |

## 분석 원칙 (팀 공통)
- fact(확인된 사실)과 estimate(추정)를 명확히 구분
- 상관관계 ≠ 인과관계
- 수치에는 반드시 출처(테이블, 컬럼, 필터조건) 명시
- 복수 아이템/유저 합산 시 유니크 중복 제거 확인 필수
