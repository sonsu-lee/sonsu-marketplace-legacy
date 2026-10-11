# Repository guidance

## 작성 언어

- 문서, 스킬, 참고 자료와 생성기 문구는 한국어로 쓴다. 일본어로 쓰지 않는다.
- 예외는 언어 자체가 대상인 내용이다.
  - Fluent English·Japanese 플러그인과 그 평가 자료는 해당 언어로 쓴다.
  - 다른 언어 사용자의 요청을 재현하는 평가 프롬프트와 예시 문장은 그 언어 그대로 둔다.
  - 履歴書·職務経歴書처럼 형식 이름이 그 언어인 용어, 원문 인용, 출처 제목은 원문을 유지한다.
  - `README.en.md`, `README.ja.md`는 `README.md`의 번역본으로 유지한다.
- 코드 식별자, 파일 경로, 명령, 로그는 원문 그대로 둔다.

## 스킬·문서 작성 기준

- 스킬, 참고 자료, 플러그인 README는 [작성 기준](docs/guides/writing-skills.md)을 따른다.
- 스킬에는 모델이 모르거나 틀리면 결과가 달라지는 것만 쓴다.
  - 쓴다: 사용자·팀의 정책과 결정, 놓치기 쉬운 외부 도구·서비스 동작, 되돌리기 어려운 작업의 경계
  - 쓰지 않는다: 명령·플래그·쿼리 문법, 도구 오류를 보고 모델이 고칠 수 있는 예외
- 반드시 막아야 하는 경계는 스킬 문장보다 호스트 권한 설정(승인 모드, permissions)으로 건다. 절차를
  스크립트·훅으로 고정하지 않는다.
- 리뷰가 세부 명령이나 드문 예외 처리를 요구하면 위 기준으로 판단한다. 반영하지 않는 예외는 근거를
  답글에 남기고, 결과를 바꿀 수 있으면 해당 결정 기록에 알려진 한계로 적는다.

## Git and Tracker Language

- Write commit messages, pull request titles and bodies, and GitHub issues in English.

## Code Review Rules

- Write the review summary and every inline review comment in Korean. Keep code identifiers,
  file paths, commands, log output, and quoted source text in their original form.

## Subagent Delegation

- For nontrivial work, the root agent identifies independently useful investigation,
  implementation, and review tasks. Assign eligible tasks to subagents in parallel when
  their results can be verified separately and the benefit exceeds coordination cost.
- Give each subagent a bounded goal, inputs, output, and read/write scope. Parallel writers
  use separate worktrees; the root agent integrates their results and verifies the final state.
- For recurring development and review roles, select the matching project Codex agent in `.codex/agents/`
  only when that role is discoverable and selectable in the current project/session. Host support
  for named selection alone is insufficient because project configuration may not be loaded.
  Read `shared/agent-policy/profiles.json` and pass the role's model and reasoning effort at spawn
  unless the user specifies otherwise. Claude Code uses the matching packaged `dev-workflow:<role>`
  agent. In both hosts, provide the concrete task brief at spawn time; a role name does not
  determine task count or file ownership. If the Codex role is unavailable or selection fails,
  read its definition and pass the role boundaries in the brief; report native role selection as
  unverified.
  A brief-only fallback does not apply the agent file's `sandbox_mode`; report the actual permission
  mode separately from the read-only instruction.
- Keep dependent work and shared writes sequential. When a task appears divisible but is
  handled directly, briefly state the concrete dependency or cost that prevented delegation.
- The root agent owns further assignments. A subagent returns requests for more workers to
  the root agent instead of delegating on its own.
