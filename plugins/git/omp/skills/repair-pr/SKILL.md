---
name: repair-pr
description: GitHub PR의 충돌을 해결하거나 리뷰 지적·실패한 CI를 검증하고 수정할 때 사용한다. 상태 조회, PR과 무관한 일반 디버깅, 코드 리뷰만의 요청과 병합은 별도 작업이다.
---

# PR 복구

정확한 PR에서 요청된 문제의 원인을 확인하고 가장 작은 수정과 검증을 수행한다. 다른 플러그인의 설치 여부와 관계없이 이미 진행 중인 개발 작업의 필수 검증을 유지한다.

## 절차

1. [PR 조회](../../references/pr-inspection.md)의 `gh` 명령으로 URL·host/repository·base/head SHA·상태·실패 CI·미해결 대화를 확인한다. 번호 생략 시 현재 브랜치의 유일한 PR을 사용한다. CI만·리뷰만 요청했다면 그 범위를 유지하고 실행 목적이 없으면 상태와 필요한 범위를 먼저 확인한다.
2. repo 지침·현재 HEAD/branch·staged/unstaged/untracked·진행 중 Git operation을 읽는다. 다른 PR branch나 충돌 작업은 소유한 격리 worktree에서 수행하고 baseline·원격 ref를 기록한다. 새 로컬 branch가 필요하면 확인된 PR head ref를 보존하고 [이름 규칙](../../references/branch-naming.md)을 적용한다. 필요한 객체만 확인된 remote에서 fetch한다. fork는 base 저장소와 실제 push 대상 head 저장소·ref를 구분하며 정확한 head를 확보한 뒤 수정한다.
3. 충돌이 범위에 있으면 최신 base를 확인해 PR head에 merge한다. commit 승인이 없으면 격리 공간에서 `git merge --no-commit --no-ff <verified-base-ref>`로 결과를 준비한다. 코드·계약·테스트로 충돌을 해결하고 관련 검사를 실행한다. merge 실패는 실제 상태를 읽고 중간 상태·격리 경로를 보존한다. 승인된 경우에만 해당 범위의 merge commit을 생성한다. CI에 commit이 필요하지만 승인이 없으면 가능한 로컬 검증과 제한을 보고한다.
4. 미해결 리뷰를 작성자와 관계없이 읽고 [리뷰 지적 대응](../../references/responding-to-review.md)으로 판정·대응한다. 같은 원인은 묶어 최소 수정하고 회귀와 가까운 보존 동작을 검증한다. 독립적으로 승인된 수정은 계속한다. 답글은 대상 저장소와 사용자 언어 규칙을 따른다.
5. 현재 head의 실패 check와 정확한 run/job 로그로 독립 원인과 연쇄 실패를 구분한다. 프로젝트 지침·manifest에서 검사 명령을 도출하고 환경·권한·서비스 장애와 코드 결함을 구별해 관찰된 원인만 수정한다. 외부 로그가 부족하면 미확인으로 남긴다.
6. 최종 diff와 관련 검사를 확인하고 [원격 복구 결과](references/remote-results.md)를 적용한다. commit은 독립적으로 설명·되돌릴 목적별로 구성한다. 승인된 push는 로컬 수정 뒤 한 번의 일반 push를 기본으로 하며 새 원격 변경은 fetch/readback으로 확인한다. 여러 단계 복구는 [작업 연속성](../../references/continuity.md)에 target/head·승인·문제별 상태를 기록한다. 코드가 바뀌면 PR 제목·본문이 최종 diff와 맞는지 [PR 작성 지침](../write-pr/references/pr-writing.md#최종-점검)으로 확인하고, 다르면 수정안을 보고하고 승인된 경우에만 `gh pr edit`로 갱신한다.

## 결과

해결·기각·추가 확인 필요, 변경·격리 경로, 로컬 검사, 실제 원격 head·CI, 답글·대화 해결 결과와 미실행을 나눠 보고한다. 로컬 검사 성공과 push 후 CI 상태는 별도 근거로 기록한다.

## 예시

입력: “PR의 페이지 크기 검증 리뷰를 처리해 줘.” 현재 head가 이미 0 이하 값을 거부하고 해당 테스트도 있다면 코드·검사 위치를 근거로 기각하고 답글 초안을 준비한다. 답글 게시·대화 해결은 각각 승인과 원격 결과 조건을 확인한 뒤 수행한다.

## 경계

- 자동 선택·스킬 이름 지정은 commit·push·댓글·대화 해결 권한이 아니다.
- 관계없는 변경을 stash/reset/restore하지 않고 기존 공간에 무조건 `gh pr checkout`하지 않는다. 미커밋 전달 결과가 있는 worktree를 삭제하지 않는다.
- rebase·force push·일괄 ours/theirs 선택, assertion·보안 검사·hook·branch protection 약화, 원격 변경 덮어쓰기와 자동 감시·병합은 수행하지 않는다.

## 참고 자료

- [PR 조회](../../references/pr-inspection.md)
- [원격 복구 결과](references/remote-results.md)
- [새 브랜치 이름](../../references/branch-naming.md)
- [작업 연속성](../../references/continuity.md)
- [리뷰 지적 대응](../../references/responding-to-review.md)
- [호스트별 도구](../../references/hosts.md)
