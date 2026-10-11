# GitHub PR 규칙

GitHub PR payload를 작성하거나 새 PR을 게시할 때 읽는다. 여러 PR의 주제·의존 관계와 native stack 게시에는 [stacked PR 규칙](stacked-prs.md)을 함께 적용한다.

## 상태를 수집한다

fetch, checkout, Git 설정 변경 없이 다음을 읽는다. `<REPO>`는 `gh repo view`가 돌려준 `url`의 host와 `nameWithOwner`다. remote URL에는 자격 증명이 들어 있을 수 있으므로 `git remote -v`처럼 URL을 출력하는 명령은 쓰지 않는다. 사용자가 원격 조회를 막았으면 `gh` 명령은 실행하지 않고 해당 항목을 미확인으로 둔다.

```bash
git status
git remote
git rev-parse --abbrev-ref '@{push}'
gh repo view --json nameWithOwner,url,isFork,parent,defaultBranchRef,pullRequestTemplates
git log --oneline <base>..HEAD
git diff --stat <base>...HEAD
git ls-remote <remote> refs/heads/<branch>
gh pr list --repo <REPO> --head <branch> --state open --json number,url,isDraft,headRepositoryOwner
```

| 확인 항목 | 근거 |
|---|---|
| 대상 저장소와 fork 여부 | `gh repo view`의 `nameWithOwner`, `isFork`, `parent` |
| base | 사용자 지정, `git config branch.<branch>.gh-merge-base`, `defaultBranchRef.name` 순서 |
| head | 현재 branch, `git rev-parse HEAD`, 원격 ref의 `git ls-remote` 결과 |
| 보낼 범위 | `git log <base>..HEAD`, `git diff <base>...HEAD` |
| 미커밋 변경과 진행 중인 Git 작업 | `git status` |
| 적용할 PR 양식 | `pullRequestTemplates` ([PR 템플릿 규칙](pr-template.md)) |
| 같은 저장소 head branch의 열린 PR | `gh pr list --head`. 다른 fork에서 이름이 같은 branch로 연 PR도 나오므로 `headRepositoryOwner.login`이 push 대상 저장소 owner인 항목만 기존 PR로 본다. |

범위에 관련 없는 commit이나 파일이 있으면 포함 범위를 그대로 두고 보고한다. 미커밋 변경은 원격 PR diff에 들어가지 않으므로 별도로 보고한다. `CONTRIBUTING`·기존 PR 관례는 직접 읽는다.

`isFork`가 `true`이면 PR이 `parent` 저장소에 있을 수 있다. PR 대상 저장소와 그 저장소 기준의 base·양식·기존 PR을 사용자에게 확인하고, 확인 전에는 게시하지 않는다.

다음이면 그 PR을 만들지 않고 정확한 현재 상태와 필요한 별도 Git workflow를 보고한다.

- `git status`가 merge·rebase·cherry-pick·revert·bisect 진행 중이거나 detached HEAD를 보인다.
- head가 base와 같은 branch(로컬 branch와 그 remote-tracking ref 포함)이거나 같은 commit이다. base branch에 직접 만든 미push commit도 PR로 보내지 않는다.
- `git log <base>..HEAD`가 비어 있다.
- 같은 저장소 head branch의 열린 PR이 있다.

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

1. `git rev-parse HEAD`, `git status`, `gh pr list --head`를 다시 실행해 head SHA가 준비한 payload와 같고 미커밋 변경·진행 중인 Git 작업·같은 head의 열린 PR이 없는지 확인한다. 최종 제목·본문·티켓 연결·검증 근거·`target_pr_state`가 요청 범위에 맞는지 대조한다.
2. 필요한 일반 push를 한 번 수행하고 remote ref를 확인한다.
3. 미디어가 없으면 `draft`에 `--draft`를 사용하고 명시된 `ready`에만 non-draft로 생성한다. 미디어가 있으면 [화면 자료 업로드 절차](../../../references/media.md#github에-올린다)대로 Draft PR 생성과 첨부를 한 명령으로 하고, 명시된 Ready 전환은 첨부 확인 뒤에 한다. `--attach`를 쓸 수 없을 때의 대안도 그 절차를 따른다.
4. 반환된 URL이나 number로 PR을 다시 읽어 정확한 저장소·URL·번호·제목·본문·base·head·Draft 상태·head SHA와 ticket reference를 확인한다. 미디어가 있으면 본문에 `NN-…` 로컬 참조가 남아 있지 않은지도 확인한다.

응답이 불명확하면 같은 push·create·edit를 반복하지 않는다. stdout, remote ref, 같은 head의 기존 PR과 저장된 body를 먼저 조회하여 성공한 단계와 남은 단계를 구분한다. 첨부의 부분 실패는 [화면 자료 업로드 절차](../../../references/media.md#github에-올린다)대로 실패한 파일만 다시 올린다.

`gh pr create`가 같은 head·base의 PR이 이미 있다며 실패하면 새 PR을 만들지 않고 출력된 URL을 보고한다.
