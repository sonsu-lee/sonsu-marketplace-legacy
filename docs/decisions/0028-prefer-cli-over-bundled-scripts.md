# ADR 0028: 수집 스크립트 대신 모델이 모르는 사실만 스킬에 쓴다

- 날짜: 2026-10-11
- 상태: 채택
- 관련 결정: [ADR 0027](0027-split-workflow-and-engineering-plugins.md)

## 배경

작성 기준은 "명령으로 수집하거나 판정할 수 있는 단계는 스크립트로 제공한다"였다. 이 기준으로 Git의
`write-pr`에는 `pr_context.py`(472줄), `inspect-prs`·`repair-pr`에는 `inspect_prs.py`가 생겼다. 두
스크립트는 `git`·`gh` 조회 결과를 자체 JSON으로 다시 묶었고, 스킬 문서는 그 JSON의 상태 이름
(`local-only`, `blockers` 8종, `unverified` 6종 등)을 다시 설명해야 했다. 사용자는 스크립트 내부 대체
경로인 `local-only`가 무엇을 확인하라는 뜻인지 문서만으로 알 수 없었다.

스크립트를 지우고 정확한 `git`·`gh` 명령을 스킬에 적자, PR #109의 자동 리뷰 세 라운드에서 지적 24개가
나왔다. 대부분은 플래그(`-f`/`-F`), 페이지네이션, URL 인코딩, upstream 없는 branch처럼 실행하다 오류를
보면 고칠 수 있는 예외였고, 반영할수록 문서가 스크립트를 글로 옮긴 형태가 됐다. 정확한 명령을 지정한
탓에 새 버그도 생겼다. stack 하단 층에 `git merge-base --is-ancestor <trunk> <branch>`를 지정해, trunk가
전진한 정상 상황에서 게시가 멈췄다.

Anthropic의 스킬 작성 가이드는 "Claude is already very smart. Only add context Claude doesn't have."라고
하고, 깨지기 쉬운 작업에만 자유도를 낮추라고 권한다. Codex 문서는 "Prefer instructions over scripts unless
you need deterministic behavior or external tooling"이라고 쓴다.

## 결정

- 스킬에는 모델이 모르거나 틀리면 결과가 달라지는 것만 쓴다. 사용자·팀의 정책과 결정, 놓치기 쉬운 외부
  도구·서비스 동작, 되돌리기 어려운 작업의 경계가 여기에 해당한다.
- 절차는 무엇을 확인하는지로 쓰고 명령·플래그·쿼리 문법은 쓰지 않는다.
- 반드시 막아야 하는 경계는 호스트 권한 설정에 둔다. 절차를 스크립트·훅으로 고정하지 않는다. 스크립트는
  외부 도구로 얻을 수 없는 계산·검증(`validate_*.py`, `memory_store.py` 등)에만 둔다.
- 드문 예외는 방어 문장 대신 아래 알려진 한계로 남긴다.
- Git의 `pr_context.py`, `inspect_prs.py`와 그 테스트를 제거하고 `write-pr`, `inspect-prs`, `repair-pr`를
  이 기준으로 다시 쓴다. fork에서 parent로 여는 PR도 "head 저장소는 base 저장소와 다를 수 있다"는 사실로
  지원한다.

## 대안

- **스크립트 축소**: 양식 조회·기존 PR·게시 차단 상태만 남긴다. 결과가 일정하지만 남은 항목도 `gh` 명령
  하나나 `gh` 자체 검사로 대신할 수 있어 유지 비용만 남는다.
- **정확한 명령 지정**: 스크립트 없이도 실행이 일정하다. 하지만 예외마다 문장이 늘고, 지정한 명령이 틀린
  상황에서도 그대로 따르게 된다.
- **훅으로 강제**: 에이전트가 지시를 건너뛰어도 막을 수 있다. 하지만 중복 PR은 `gh pr create`가 이미 막고,
  Codex·Claude Code·omp마다 훅 형식이 달라 세 벌을 관리해야 한다.

## 결과

- 조회 방법은 실행하는 모델이 정한다. 수집기가 보장하던 조회 단위별 `partial` 판정은 사라지고, 실패한
  조회는 확인한 범위와 함께 보고한다.
- 알려진 한계:
  - commit에만 연결되고 PR 병합에 영향을 주지 않는 check run은 `gh pr checks`에 나오지 않는다.
  - 아직 시작하지 않은 필수 check는 `gh pr checks --required`에 나오지 않는다.
- `plugins/git/tests`가 비어 제거된다. Git 플러그인 검증은 생성물 검사로 한다.
- Review의 `pr_review_snapshot.py`처럼 같은 성격의 다른 수집 스크립트는 이 결정으로 바로 바꾸지 않는다.

## 다시 볼 때

- 스킬에서 뺀 조회 방법 때문에 같은 실패가 평가나 작업 기록에서 반복될 때
- GitHub·`gh`가 위에 적은 동작을 바꿀 때
