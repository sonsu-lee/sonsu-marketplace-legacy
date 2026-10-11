# GitHub stacked PR 규칙

주제별 PR 경계를 정하거나 여러 새 PR을 GitHub의 native stack으로 게시할 때 읽는다. 이 기능은 GitHub public preview이므로 실행 시점의 GitHub 문서와 설치된 `gh stack --help`를 확인한다.

## 주제와 의존 관계로 나눈다

- PR 경계는 [PR 작성 지침](pr-writing.md#pr-경계를-점검한다)을 따른다.
- 뒤 주제가 앞 주제의 코드·계약에 의존하면 아래에서 위로 branch와 PR을 배열한다. 아래 PR은 trunk, 위 PR은 바로 아래 PR의 head branch를 base로 한다. 각 PR은 그 층의 diff만 설명한다.
- 사용자가 stack을 요청했더라도 작업 순서나 같은 티켓만으로 stack을 만들지 않는다.

각 층의 payload에는 `base`, `head`, 검토할 주제, 아래 PR에 대한 의존성, 해당 층의 diff·검증·티켓 intent를 기록한다. stack 전체의 목적은 짧게 연결하되, 모든 층에 동일한 전체 diff나 검증 결과를 복사하지 않는다. 사용자에게 보이는 변경과 화면 자료는 그 변경을 소유한 PR에 둔다.

GitHub Issues closing keyword는 non-default base의 자동 종료 근거로 사용하지 않는다. stack 전체가 완료되어야 닫히는 티켓은 각 층에서 `Part of`처럼 completion을 만들지 않는 reference를 사용하고, 실제 merge 뒤 canonical ticket 상태를 별도로 확인한다. Linear도 [티켓 연결 규칙](ticket-linking.md)의 event별 automation을 따른다.

공식 참고: [GitHub stacked PR 개요](https://docs.github.com/en/pull-requests/get-started/about-stacked-prs), [GitHub stacked PR 요구사항](https://docs.github.com/en/pull-requests/reference/stacked-pull-requests)

## 게시 전 chain을 고정한다

1. 각 층을 [상태 수집](github.md#상태를-수집한다) 명령으로 읽는다. 하단 층의 base는 trunk, 위층의 base는 바로 아래 branch다. 모든 층의 push 대상 저장소가 같고 fork가 아닌지 확인한다. 다르거나 대상 저장소를 확인하지 못하면 cross-fork·대상 불명 stack으로 보고 게시를 멈춘다. `git merge-base --is-ancestor <아래 branch> <위 branch>`가 실패하면 선형 ancestry가 아니므로 게시를 멈춘다. 각 층의 범위, 원격 ref, 기존 PR을 기록한다.
2. 위 branch가 아래 branch의 변경을 포함하지 않거나 각 층에 다른 주제가 섞이면 게시를 멈추고 필요한 Git 작업을 보고한다. 이 절차는 **모든 층이 새 PR**일 때 사용한다. publish 시작 전에 존재하던 PR이 끼어 있으면 그 PR의 base나 stack membership은 그대로 두고 현재 관계와 필요한 별도 작업을 보고한다.
3. 각 층의 최종 제목·본문·양식·ticket reference·검증 상태·화면 자료를 준비한다. `target_pr_state`는 모든 층에 기본 Draft를 적용하고, 사용자가 stack 전체 또는 특정 층의 Ready를 명시한 경우에만 해당 층을 Ready로 정한다.
4. 설치된 `gh stack link --help`, 인증 주체, 저장소·remote, GitHub stack 기능과 권한을 읽기 전용으로 확인한다. `GET /repos/{owner}/{repo}/stacks`는 조회 경로지만, 빈 목록만으로 생성 권한까지 단정하지 않는다. native stack을 사용할 수 없으면 일반 종속 PR로 조용히 대체하지 않는다.

`gh stack submit --auto`와 branch 인자를 받는 `gh stack link`는 제목·본문을 자동 생성할 수 있다. 준비한 payload를 그대로 게시하려면 각 PR을 명시적인 제목·본문으로 만든 뒤 **PR URL만** `gh stack link`에 전달한다. URL 인자는 branch push·새 PR 자동 생성을 피한다. `gh stack link`는 기존 PR base를 바꿀 수 있으므로 연결 전후 chain을 대조한다. `--open`은 기존 PR까지 Ready로 바꿀 수 있어 사용하지 않는다.

공식 참고: [GitHub stack 생성](https://docs.github.com/en/pull-requests/how-tos/create-pull-requests/creating-stacked-pull-requests), [`gh stack link`](https://github.com/github/gh-stack)

## Native stack으로 게시하고 검증한다

1. publish 직전에 각 층에서 [생성 전 재확인](github.md#생성하고-검증한다)을 하고 고정한 head SHA·원격 ref·PR 부재와 대조한다. `target_pr_state`, payload와 필수 미디어 준비 상태도 확인한다. 한 층이라도 필수 조건이 부족하면 stack 게시를 시작하지 않는다.
2. 승인된 기존 remote에 각 branch를 정확한 refspec으로 일반 push하고 SHA를 재조회한다. push 결과가 불명확하면 원격 ref를 조회하며 같은 작업을 무작정 재시도하지 않는다.
3. 아래 층부터 `gh pr create --head <branch> --base <trunk-or-lower-branch> --title <title> --body-file <file> --draft`로 새 Draft PR을 만든다. 미디어가 있으면 [화면 자료 업로드 절차](../../../references/media.md#github에-올린다)에 따라 같은 명령에 `--attach`로 첨부하고 본문을 다시 읽어 확인한다. 명시된 Ready 전환은 stack 연결 뒤로 미룬다. 생성마다 URL과 원격 제목·본문·base·head·Draft 상태를 재조회한다.
4. 모든 새 PR이 정확한 chain이면 `gh stack link --base <trunk> <bottom-pr-url> ... <top-pr-url>`로 native stack을 만든다. 인자에는 검증한 PR URL만 넣는다. 연결 실패나 응답 불명은 원격 stack membership과 각 PR의 base·head를 먼저 조회한다. 새 PR 생성·link를 처음부터 반복하지 않는다.
5. stack 조회와 각 PR 재조회에서 stack 번호·순서, trunk, 각 base·head SHA, 제목·본문·Draft 상태, 미디어와 ticket reference를 확인한다. 필수 첨부와 정확한 연결이 확인된 뒤 명시된 층만 Ready로 바꾸고 다시 읽는다. 연결 결과는 기대 chain 전체가 확인되면 `applied`, 일부만 확인되면 `partial`, 어느 층에도 적용되지 않았다고 확인되면 `unapplied`, 조회로 판정할 수 없으면 `unknown`으로 보고한다. 연결 성공을 단순한 base chain만으로 주장하지 않는다.

branch push, PR 생성, 미디어 첨부 또는 stack 연결 중 한 단계라도 실패하거나 결과가 불명확하면 후속 층의 게시·연결과 모든 Ready 전환을 멈춘다. 이미 수행한 층의 remote SHA, PR URL·base·head·Draft·본문·첨부와 stack membership을 재조회하고, 층별 `applied | partial | unapplied | unknown | not_attempted`를 구분해 보고한다. 확인된 미완료 단계만 이어서 처리하며 결과가 불명확한 원격 작업을 반복하지 않는다.

기존 PR의 수정·재배치·merge, `gh stack rebase|push|submit --auto|modify|merge`는 이 절차의 부수 작업이 아니다. 특히 rebase·stack push에는 history rewrite와 `--force-with-lease`가 포함될 수 있으므로 별도 권한과 절차를 따른다.

공식 참고: [GitHub stack REST 조회](https://docs.github.com/en/rest/pulls/stacks), [GitHub stack 관리](https://docs.github.com/en/pull-requests/how-tos/create-pull-requests/managing-stacked-pull-requests)
