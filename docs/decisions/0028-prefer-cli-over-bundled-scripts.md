# ADR 0028: 기존 CLI로 되는 수집은 스크립트로 감싸지 않는다

- 날짜: 2026-10-11
- 상태: 채택
- 관련 결정: [ADR 0027](0027-split-workflow-and-engineering-plugins.md)

## 배경

작성 기준은 "명령으로 수집하거나 판정할 수 있는 단계는 스크립트로 제공한다"였다. 이 기준으로 Git의
`write-pr`에는 `pr_context.py`(472줄), `inspect-prs`·`repair-pr`에는 `inspect_prs.py`가 생겼다. 두
스크립트는 `git`·`gh` 조회 결과를 자체 JSON으로 다시 묶었고, 스킬 문서는 그 JSON의 상태 이름
(`local-only`, `blockers` 8종, `unverified` 6종, 조회 단위 `complete`·`partial` 등)을 다시 설명해야 했다.
사용자는 스크립트 내부 대체 경로인 `local-only`가 무엇을 확인하라는 뜻인지 문서만으로 알 수 없었다.

같은 정보는 `gh`가 이미 제공한다.

- `gh repo view --json pullRequestTemplates`는 저장소 양식과 owner `.github` 저장소의 기본 양식을 함께
  돌려준다. 저장소 양식이 없는 `supabase/terraform-provider-supabase`에서 `supabase/.github` 양식과 같은
  본문을 받았다.
- `gh pr create`는 같은 head→base의 열린 PR이 있으면 기존 URL과 함께 실패한다.
- `gh pr checks --json`의 `bucket`이 check run과 commit status를 `pass`·`fail`·`pending`·`skipping`·
  `cancel`로 정규화한다.
- `gh api graphql --paginate`가 `pageInfo.endCursor`를 따라 review thread를 끝까지 읽는다.

Codex 문서는 "Prefer instructions over scripts unless you need deterministic behavior or external
tooling"을, Claude 문서는 깨지기 쉬운 작업에만 정해진 스크립트를 쓰라고 권한다.

## 결정

- 스킬의 수집 단계는 기존 CLI 명령을 그대로 적는다. 스킬에는 명령, 읽을 필드, 결과별 행동을 쓴다.
- 스크립트는 기존 CLI로 얻을 수 없는 계산·검증이거나, 같은 명령 조합을 손으로 따르면 실제로 결과가
  달라지는 경우에만 둔다. 산출물 형식 검증기(`validate_*.py`)와 저장소 관리 도구(`memory_store.py` 등)가
  여기에 해당한다.
- 규칙 강제를 위한 훅은 호스트 CLI가 같은 검사를 하지 않고, 위반 피해가 되돌리기 어려울 때만 검토한다.
- Git의 `pr_context.py`, `inspect_prs.py`와 그 테스트를 제거하고 `write-pr`, `inspect-prs`, `repair-pr`를
  `git`·`gh` 명령으로 전환한다.

## 대안

- **스크립트 축소**: 양식 조회·기존 PR·게시 차단 상태만 남긴다. 결과가 일정하지만 남은 항목도 `gh`
  명령 하나나 `gh` 자체 검사로 대신할 수 있어 유지 비용만 남는다.
- **훅으로 강제**: 에이전트가 지시를 건너뛰어도 막을 수 있다. 하지만 중복 PR은 `gh pr create`가 이미
  막고, Codex·Claude Code·omp마다 훅 형식이 달라 세 벌을 관리해야 한다.
- **현행 유지와 문서 보완**: 변경 범위가 가장 작지만 스크립트 용어를 설명하는 문서 부담이 남는다.

## 결과

- 조회 결과는 `gh` 출력 그대로 해석한다. 조회 단위별 `partial` 판정, 처음·마지막 head 자동 비교 같은
  수집기 전용 보장은 사라지고, 스킬이 필요한 시점에 `headRefOid`를 다시 읽어 비교한다.
- `plugins/git/tests`가 비어 제거된다. Git 플러그인 검증은 생성물 검사로 한다.
- Review의 `pr_review_snapshot.py`처럼 같은 성격의 다른 수집 스크립트는 이 결정으로 바로 바꾸지 않는다.

## 다시 볼 때

- `gh`가 `pullRequestTemplates`, `bucket`, 중복 PR 검사 동작을 바꿀 때
- 명령 조합을 따르지 않아 생긴 실패가 평가나 작업 기록에서 반복될 때
