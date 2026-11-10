---
author: gyuha
decided: 2026-11-09 16:07
---
# 프로바이더 계정은 조직 키를 기본으로 하고, 팀원 개인 계정 연결도 허용한다

에이전트는 기본적으로 관리자가 등록한 프로바이더별 조직 API 키로 실행한다. 팀원이 원하면 자기 계정을 연결해서 쓸 수도 있다.

데몬 호스트 한 사람의 구독 로그인을 팀 전체가 같이 쓰는 방식은 쓰지 않는다. 비용은 가장 적게 들지만 다음 규정에 어긋난다:
- Anthropic Agent SDK 문서: "Unless previously approved, Anthropic does not allow third party developers to offer claude.ai login or rate limits for their products" (code.claude.com/docs/en/agent-sdk/overview, 2026-11-09 확인)
- 소비자 약관 2조: 계정을 다른 사람에게 제공하는 것을 금지한다.

조직 키만 허용하면 사용량에 따라 요금이 붙어서 비용 부담이 크다. 그래서 개인 계정 연결도 함께 허용한다.

개인 계정은 paseo와 같은 방식으로 처리한다(2026-11-09 추가 결정).
- 계정은 사용자에게 붙이지 않고 **프로바이더 항목**에 붙인다.
- 관리자가 기본 프로바이더를 바탕으로(`extends`) 자기 env를 가진 별도 항목을 만든다. 예: `claude-<이름>` 항목에 `ANTHROPIC_API_KEY`나 `CLAUDE_CONFIG_DIR`를 지정한다.
- 연결 방식(API 키인지 구독 로그인인지)은 제한하지 않는다.
- 구독 로그인을 쓸 때 약관을 지킬 책임은 그 계정 주인에게 있다.
