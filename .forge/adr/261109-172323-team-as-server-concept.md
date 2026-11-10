---
author: gyuha
decided: 2026-11-09 17:23
---
# 디렉토리별 에이전트 팀은 데몬 서버의 새 "팀" 개념으로 구현한다

디렉토리별 에이전트 팀(오케스트레이터와 역할들)은 paseo 서버(`packages/server`)에 새 도메인 개념 "팀"으로 구현한다.

## 검토한 대안
- **paseo 플러그인으로 구현**: 서버를 고치지 않아도 되는 방법이다.
  - 플러그인 훅 `agent.create.before`/`agent.open.before`와 `defineAgentTool`로 `spawn_role` 도구와 역할 주입을 만들 수 있다(`public-docs/plugins/hooks.md`).
  - 이 방법은 채택하지 않았다. 팀을 플러그인 계층이 아니라 서버의 일급 개념으로 둔다.
- **Agent profile 확장**: profile은 호스트 전체에 하나뿐이고 시스템 프롬프트를 의도적으로 갖지 않는다(`agent-profile.ts`). 그래서 맞지 않는다.

## 결과
- 서버 수정 범위가 커지므로 upstream 병합 비용이 늘어난다.
- 충돌을 줄이려면 팀 관련 코드는 서버 안에서도 별도 모듈로 격리하고, 기존 파일은 연결 지점만 고친다. [ADR-261109-152306](261109-152306-paseo-fork-additive-packages.md)의 "서버 수정은 최소, 격리" 원칙을 따른다.
- 플러그인 방식은 나중에 서버 수정이 감당하기 어려워지면 다시 검토할 대안으로 남겨 둔다.
