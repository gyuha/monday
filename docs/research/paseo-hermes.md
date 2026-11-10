# 조사: paseo · Hermes Agent

조사일: 2026-11-09. 두 저장소를 shallow clone해서 직접 확인함 (paseo HEAD `180f075`, v0.15.3).
표기: **[확인]** 저장소에서 직접 읽음 · **[추론]** 확인한 내용에서 추론함.

## paseo — https://github.com/getpaseo/paseo

### 라이선스
- **Apache-2.0** [확인]. 루트 LICENSE 앞에 저작권 머리말이 붙어 있어서 GitHub API는 `NOASSERTION`으로 표시함. 각 패키지의 `package.json`에도 Apache-2.0이 명시됨.
- 예외: `packages/expo-two-way-audio`는 MIT [확인].
- NOTICE 파일은 없음 [확인].
- 재사용할 때 지킬 것:
  - LICENSE와 저작권 표기를 유지한다.
  - 수정한 파일에는 수정했다고 표시한다 (§4b).
  - "Paseo" 이름과 브랜드는 쓰지 않는다 (§6).

### 스택
- TypeScript npm-workspaces 모노레포 [확인].
- 패키지 구성: `packages/{server, client, protocol, relay, cli, app (Expo), desktop (Electron), plugin, website}`.
- 데몬은 Node.js에서 실행됨.
- 주요 의존성: `@anthropic-ai/claude-agent-sdk`, `@agentclientprotocol/sdk`, `@opencode-ai/sdk`, `@modelcontextprotocol/sdk`.

### 데몬
- Node 프로세스 하나가 WebSocket API를 연다. 기본 주소는 `127.0.0.1:6767`이고 `PASEO_LISTEN`으로 바꿀 수 있다 [확인].
- CLI, Electron 앱, Expo 앱(모바일/웹)이 모두 WebSocket 클라이언트로 붙는다 [확인].
- 인증은 선택 사항이며 비밀번호를 bcrypt bearer 토큰으로 검사한다 (`auth.ts`) [확인].
- 원격 접속은 relay를 거친다 [확인]:
  - 데몬이 relay로 바깥 방향 연결을 맺으므로 포트를 열 필요가 없다.
  - QR 페어링으로 데몬 공개키를 넘긴다.
  - Noise NK 핸드셰이크와 ChaCha20-Poly1305로 종단 간 암호화한다.
  - 기본 relay `relay.paseo.sh`는 paseo가 운영하는 서비스다. 자체 호스팅용 코드는 `packages/relay`에 있다.

### 프로바이더
- 위치: `packages/server/src/server/agent/` [확인].
- 각 프로바이더는 `AgentClient` / `AgentSession` 인터페이스(`agent-sdk-types.ts`)를 구현한다.
- `provider-registry.ts`에 등록되고, `agent-manager.ts`가 생명주기를 관리한다.
- 구현체 (`agent/providers/`):
  - `claude/`: Claude Agent SDK 사용
  - `codex/`: app-server를 JSON-RPC로 호출
  - `opencode/`, `pi/`, `omp/`
  - ACP 계열: generic, Copilot, Cursor, Kimi, Kiro, Trae
- 사용자 정의 프로바이더는 `~/.paseo/config.json`의 `agents.providers`에 추가한다.

### 폴더, 멀티 에이전트, 저장소
- 모델 구조는 Project(루트 경로 1개) → Workspace(정확한 `cwd`, 선택적으로 git worktree) → Agents [확인].
- 같은 cwd에 에이전트를 여러 개 둘 수 있다. e2e 테스트 `workspace-same-cwd-isolation`이 이를 검증한다 [확인].
- 부모-자식 에이전트 [확인]:
  - 에이전트가 MCP 도구 `create_agent`로 다른 에이전트를 만든다.
  - 부모는 `paseo.parent-agent-id`로 기록된다.
  - 부모를 보관 처리하면 자식도 함께 보관 처리된다.
  - Claude 자체 서브에이전트는 `provider-subagents/store.ts`에서 따로 추적한다.
- 저장은 `~/.paseo` 아래 JSON 파일로 한다. 원자적으로 쓰고 Zod로 검증하며, SQLite는 쓰지 않는다 [확인].
- 재사용 시 결합도 위험 [추론]:
  - `agent/providers/*`만 떼어내도 `agent-sdk-types.ts`, `@getpaseo/protocol`, 그 밖의 내부 import가 줄줄이 따라온다.
  - 따라오는 범위가 어디까지인지는 확인하지 않았다.

## Hermes Agent — https://github.com/NousResearch/hermes-agent

- **MIT** 라이선스 (Copyright 2025 Nous Research) [확인].
- 스택은 Python [확인]:
  - 텔레그램: `python-telegram-bot`
  - 슬랙: `slack-bolt`
  - 세션 저장: SQLite
- 게이트웨이 [확인]:
  - `hermes gateway start` 프로세스 하나가 모든 플랫폼에 연결된다.
  - 플랫폼별 어댑터는 `plugins/platforms/<name>/`에 있다.
  - 슬랙은 Socket Mode로 연결하므로 공개 URL이 필요 없다.
- 세션은 채팅, 토픽, 스레드 단위로 나뉜다 [문서에서 확인, 세부 키 규칙은 미확인].
- 접근 제어 [확인]:
  - 플랫폼별 허용 사용자 목록을 둔다.
  - 또는 DM 페어링을 쓴다. 일회용 코드는 1시간 동안 유효하고, `hermes pairing approve`로 승인한다.
- 명령어 [확인]: `/new` `/stop` `/approve` `/deny` `/model` `/resume` `/sessions` `/status` `/undo` `/retry` 등.
- 위험한 명령은 버튼으로 승인받는다. 제한 시간 안에 응답이 없으면 실행하지 않는다 (`base_exec_approval.py`) [확인].
- Hermes는 자체 LLM 루프를 돌린다. paseo처럼 Claude Code나 Codex CLI를 감싸는 구조가 아니다 [구조상 확인].

## 이 프로젝트에 주는 함의
1. **라이선스**: 두 라이선스 모두 재사용에 문제가 없다 [높음]. 단, 제품명에 "Paseo"를 쓰면 안 된다.
2. **언어**: Hermes는 Python이라 코드를 가져올 수 없다. 설계만 TypeScript로 다시 구현한다 [높음]. 후보 라이브러리는 grammY 또는 Telegraf, 그리고 `@slack/bolt`(Socket Mode)다.
3. **통합 지점**: 메신저 봇은 데몬 WebSocket에 붙는 별도 클라이언트로 만드는 쪽이 상류 병합에 유리하다 [중간]. 권한 요청 프로토콜(`AgentPermissionRequestKind`, `permission-response.ts`)이 이미 있어서 메신저 버튼에 연결할 수 있다.
4. **결합도**: paseo 서버는 크고 변경이 잦다. 음성, worktree, relay 서브시스템까지 포함하고 있다 [중간].
