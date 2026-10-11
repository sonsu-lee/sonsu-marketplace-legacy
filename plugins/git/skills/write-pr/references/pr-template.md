# PR 템플릿 선택과 작성 규칙

PR title과 body를 작성하기 전에 읽는다. 대상 저장소의 유효한 템플릿을 우선하고, 없으면 owner의 account-level default template을 쓴다. 번들 기본 템플릿은 [기본 템플릿 적용 조건](#기본-템플릿-적용-조건을-확인한다)을 따른다.

## 양식 조회 결과를 해석한다

`gh repo view <REPO> --json pullRequestTemplates`가 대상 저장소 default branch의 양식을 `filename`과 `body`로 돌려준다. 저장소에 양식이 없으면 owner `.github` 저장소의 기본 양식이 대신 나온다.

| 결과 | 해석 |
|---|---|
| 항목 하나 이상 | 돌려받은 양식 중에서 [아래 순서](#관련-템플릿-하나를-선택한다)로 하나를 고른다. |
| 빈 배열 | 저장소와 owner 기본 양식 모두 없다. [기본 템플릿](#기본-템플릿-적용-조건을-확인한다)을 쓸 수 있다. |
| 명령 실패 | 양식을 확인하지 못했다. 기본 양식으로 확정하지 않는다. |

사용자가 확인된 조회 결과로 양식 상태나 양식 본문을 제공했으면 그 상태를 같은 표로 해석한다.

현재 feature branch에만 추가·수정된 템플릿은 사용자가 그 변경본을 쓰라고 명시한 경우에만 사용한다.

공식 참고: [GitHub PR template 만들기](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository), [PR template 개요](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/about-issue-and-pull-request-templates), [account-level default community health file](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file)

`CONTRIBUTING`, `AI_POLICY.md`, `AGENTS.md`의 PR 본문 요구는 양식의 필수 항목과 같이 확인한다. 다만 AI 공개 요구는 [공통 AI 사용 표기 규칙](../../../references/tracker-prose.md#ai-사용-표기)에 따라 작성 권한과 충돌을 처리하며 자동으로 채우지 않는다.

## 관련 템플릿 하나를 선택한다

선택 순서는 다음과 같다.

1. 사용자가 정확한 양식 path나 이름을 지정했으면 그 파일이 default branch에 존재하는지 확인하여 사용한다.
2. `CONTRIBUTING`, 저장소 문서나 양식 자체가 change type·경로별 선택 규칙을 제공하면 실제 diff에 맞는 파일을 사용한다.
3. 결과가 하나뿐이면 그 양식을 사용한다.
4. 여러 양식 중 하나가 실제 변경 유형에 명확히 대응하면 그 근거를 기록하고 사용한다.
5. 여러 후보가 동등하게 맞고 저장소 근거로 고를 수 없으면 가능한 제목, 변경 요약과 후보 목록까지 준비한 뒤, 최종 본문 확정이나 publish 전에 사용자에게 양식 선택을 요청한다. 후보를 합치거나 fallback으로 바꾸지 않는다.

선택한 양식의 `filename`을 기록한다.

## 출력 언어를 결정한다

양식 선택과 출력 언어 선택은 별개로 처리한다. 생성하는 제목, 설명, validation과 caption의 언어는 다음 근거를 순서대로 사용한다.

1. 사용자가 PR 언어를 명시했으면 그 언어를 사용한다.
2. repository의 `CONTRIBUTING`, `AGENTS.md`·`CLAUDE.md`, PR 지침이나 일관된 최근 PR 관례가 언어를 정하면 따른다.
3. 연결할 기준 티켓, 승인된 specification이나 제품 문서가 명확한 주 언어를 사용하면 그 언어를 따른다.
4. 근거가 없으면 영어를 사용한다. 대화 언어와 결과 보고 언어는 PR 언어의 근거로 쓰지 않는다.

저장소 template에 이미 있는 제목, checklist와 고정 안내 문구는 원문 그대로 두고, 채워 넣는 내용만 결정된 PR 언어로 쓴다. template이 특정 언어로 전체 작성을 요구하면 그 지시를 따른다. 코드, 명령어, 로그, identifier, ticket ID와 URL은 원문을 유지한다. 언어 판단은 다른 언어 플러그인이나 스킬 없이 이 규칙으로 한다.

고정 제목이 한국어나 일본어라는 사실만으로 자유 설명의 언어를 정하지 않는다. 전체 작성 언어를 요구하는 지침이나 위의 언어 근거가 없으면 고정 제목은 원문대로 보존하고 자유 설명은 영어로 쓴다.

기본 양식의 섹션 제목(Why, Changes, Notes, Screenshots, Verification)은 출력 언어와 관계없이 영어로 둔다.

## 기본 템플릿 적용 조건을 확인한다

조회 결과가 빈 배열이면 번들 [PR 기본 양식](../assets/templates/pull-request.md)을 사용한다. 조회에 실패했으면 확인된 사실로 준비 가능한 설명만 임시 초안으로 쓰고, 양식 확인 상태와 남은 게시 조건을 본문 밖에서 알린다. 이때 확인된 변경 요약과 미확인 사항은 전달하되 양식 부재나 게시 준비 완료로 표현하지 않는다.

## 저장소에 PR 양식을 설치한다

사용자가 요청하면 [PR 기본 양식](../assets/templates/pull-request.md)을 대상 저장소의 `.github/pull_request_template.md`로 복사한다.
