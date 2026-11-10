---
author: gyuha
decided: 2026-11-09 16:03
---
# 제품명은 monday이고, 사용자에게 보이는 이름만 바꾼다

제품명은 `monday`로 정한다. 비공개 팀 도구라서 monday.com 상표와 부딪힐 위험은 낮다고 판단했다.

사용자에게 보이는 이름만 바꾼다:
- CLI 명령
- 홈 폴더 (`~/.monday`)
- 환경변수 접두사
- UI 문구

포크 내부의 패키지 이름(`@getpaseo/*`)과 코드 식별자는 바꾸지 않는다. 스코프까지 바꾸면 upstream을 병합할 때마다 거의 모든 import에서 충돌이 나서, [ADR-261109-152306](261109-152306-paseo-fork-additive-packages.md)의 주기 병합 원칙이 무너진다. 그래서 코드에 `@getpaseo` 이름이 남아 있는 것은 의도한 것이다.
