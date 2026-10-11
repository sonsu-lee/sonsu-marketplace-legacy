# GitHub PR 규칙

GitHub PR payload를 작성하거나 새 PR을 게시할 때 읽는다. 여러 PR의 주제·의존 관계와 native stack 게시에는 [stacked PR 규칙](stacked-prs.md)을 함께 적용한다.

## 상태를 수집한다

fetch, checkout, Git 설정 변경 없이 다음을 확인한다. 사용자가 원격 조회를 막았으면 원격에 기대는 항목은 미확인으로 둔다.

- 대상: PR을 만들 base 저장소와 head 저장소, 활성 `gh` 계정
- base: 사용자 지정, `branch.<branch>.gh-merge-base` 설정, 저장소 default branch 순서
- 보낼 범위: 원격 base부터 head까지의 commit과 diff
- push 상태: head branch가 원격에 있는지와 그 SHA
- PR에 들어가지 않는 변경: 미커밋·untracked 변경
- 양식: [PR 템플릿 규칙](pr-template.md)
- 기존 PR: 같은 head 저장소·branch에서 열린 PR
- `CONTRIBUTING`과 기존 PR 관례

놓치기 쉬운 GitHub 동작:

- PR은 base 저장소의 base branch와 head 저장소의 head branch를 비교하고, 활성 `gh` 계정 명의로 만들어지며, push된 commit만 담는다.
- fork에서 여는 PR은 head 저장소(fork)와 base 저장소(`parent`)가 다르다. base·양식·기존 PR은 base 저장소 기준이고 head branch는 fork에 있다. 사용자가 어느 저장소로 보낼지 정하지 않았으면 확인한 뒤 게시한다.
- 로컬의 remote-tracking base는 마지막 fetch 시점의 값이다. 원격 base가 더 나아갔으면 범위가 실제 PR과 달라지므로 fetch가 필요하다고 보고한다.
- branch 이름으로 찾은 PR에는 다른 fork의 같은 이름 branch PR도 섞인다. head 저장소 owner가 같은 것만 기존 PR이다.
- remote URL에는 자격 증명이 들어 있을 수 있으므로 출력하지 않는다.

다음이면 그 PR을 만들지 않고 정확한 현재 상태와 필요한 별도 Git workflow를 보고한다.

- merge·rebase·cherry-pick·revert·bisect가 진행 중이거나 detached HEAD다.
- head가 base와 같은 branch이거나 같은 commit이다. base branch에 직접 만든 미push commit도 PR로 보내지 않는다.
- 보낼 commit이 없다.
- 같은 head의 열린 PR이 있다.

PR에 새 branch가 필요하면 이름을 제안만 하고 생성·rename은 사용자의 별도 Git 작업으로 넘긴다. 이름을 제안할 때는 [공통 이름 규칙](../../../references/branch-naming.md)을 읽는다. 조회에 실패한 항목은 확인하지 못한 상태로 보고하고 그 항목에 기대는 결정을 확정하지 않는다.

## payload를 준비한다

title과 body를 명시적으로 완성한다. [PR 템플릿 규칙](pr-template.md)에 따라 저장소 template과 언어를 결정하고 `--fill`의 자동 생성 결과만 사용하지 않는다. ticket reference, 검증 내용과 화면 자료는 실제 근거가 있을 때만 넣는다.

`target_pr_state`는 [메인 스킬의 모드 규칙](../SKILL.md#모드를-정한다)으로 정한다. Draft 지원 여부를 확인하며, Draft 요청을 Ready로 대체하지 않는다.

CLI에서는 완성한 multiline body를 임시 파일에 기록하고 `gh pr create --body-file`로 전달한다. `--template`은 base 저장소가 노출한 양식 filename으로 시작 본문만 제공하고 `--body`·`--body-file`과 함께 쓸 수 없으므로 사용하지 않는다. 실행 시점의 `gh pr create --help`, target host와 base 저장소 권한을 확인한다. `--dry-run`도 Git push를 수행할 수 있으므로 read-only 검사로 사용하지 않는다. 미디어가 있으면 생성 전에 [화면 자료 규칙](../../../references/media.md)을 읽어 파일 형식·용량과 `--attach` 지원 여부를 확인한다.

공식 참고: [GitHub CLI `gh pr create`](https://cli.github.com/manual/gh_pr_create), [`gh pr edit`](https://cli.github.com/manual/gh_pr_edit)

## publish 권한을 적용한다

명시적인 새 PR publish 요청이 있고 대상 branch가 아직 승인된 기존 remote에 게시되지 않았다면 필요한 일반 push를 수행할 수 있다. 정확한 refspec을 사용하고 결과를 다시 읽는다.

다음이 필요하면 중단한다.

- force push
- 새 fork 또는 remote
- branch 생성·rename
- commit 수정
- Git 설정 변경
- 기존 PR 수정
- 로그인, 계정 전환 또는 scope 확대

같은 head의 기존 PR이 있으면 새 PR을 만들지 않는다. publish 시작 전에 이미 존재하던 PR은 업데이트하지 않고 URL과 현재 상태를 보고한다. 방금 만든 PR에 대한 예외는 [경계](../SKILL.md#경계)를 따른다.

## 생성하고 검증한다

1. 준비할 때와 활성 계정, head SHA, 원격 head가 같고, PR에 들어가지 않는 변경과 같은 head의 열린 PR이 없는지 다시 확인한다. 다르면 게시하지 않고 보고한다. 최종 제목·본문·티켓 연결·검증 근거·`target_pr_state`가 요청 범위에 맞는지 대조한다.
2. 필요한 일반 push를 한 번 수행하고 원격 head SHA를 확인한다.
3. 미디어가 없으면 `draft`에 `--draft`를 사용하고 명시된 `ready`에만 non-draft로 생성한다. 미디어가 있으면 [화면 자료 업로드 절차](../../../references/media.md#github에-올린다)대로 Draft PR 생성과 첨부를 한 명령으로 하고, 명시된 Ready 전환은 첨부 확인 뒤에 한다. `--attach`를 쓸 수 없을 때의 대안도 그 절차를 따른다.
4. 반환된 URL이나 number로 PR을 다시 읽어 정확한 저장소·URL·번호·제목·본문·base·head·Draft 상태·head SHA와 ticket reference를 확인한다. 미디어가 있으면 본문에 `NN-…` 로컬 참조가 남아 있지 않은지도 확인한다.

응답이 불명확하면 같은 push·create·edit를 반복하지 않는다. stdout, remote ref, 같은 head의 기존 PR과 저장된 body를 먼저 조회하여 성공한 단계와 남은 단계를 구분한다. 첨부의 부분 실패는 [화면 자료 업로드 절차](../../../references/media.md#github에-올린다)대로 실패한 파일만 다시 올린다.

`gh pr create`가 같은 head·base의 PR이 이미 있다며 실패하면 새 PR을 만들지 않고 출력된 URL을 보고한다.
