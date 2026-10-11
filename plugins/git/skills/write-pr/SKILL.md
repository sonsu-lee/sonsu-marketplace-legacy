---
name: write-pr
description: 현재 Git 변경을 주제별 단일 PR 또는 GitHub native stacked PR의 초안·게시 payload로 만들고 티켓과 화면 자료를 연결할 때 사용한다. branch·commit 생성, 기존 PR 수정, 코드 리뷰와 병합은 담당하지 않는다.
---

# write-pr: PR 작성

현재 Git 변경에서 왜 바꿨고 어떤 관점과 결정으로 바꿨는지 설명하는 주제별 PR을 만들고, 서로 의존하는 주제는 native stacked PR로 연결한다. 현재 대화, 저장소와 확인 가능한 티켓 정보만으로 동작하며 다른 플러그인이 설치되었거나 먼저 실행되었다고 가정하지 않는다.

## 작업 연속성

여러 단계의 작업이나 외부 쓰기를 맡은 메인 controller는 [연속성 참고 자료](../../references/continuity.md)로 진행과 근거를 기록한다. 컴팩션·재개 후에는 실제 상태와 대조한다. 짧은 단발 작업과 위임된 작업자는 별도 기록을 만들지 않으며, 파일 쓰기가 금지되면 checkpoint와 Git exclude도 수정하지 않는다.

## 절차

### 모드를 정한다

- `draft`: 저장소를 읽고 PR별 제목, 본문, base·head, 티켓 연결, 검증 상태와 화면 자료 계획을 완성한다. push, 미디어 업로드, PR 생성 없이 끝나는 준비 모드이며 GitHub Draft 상태와 다르다.
- `publish`: 사용자가 현재 대화에서 새 PR 생성을 명시적으로 요청한 경우에 쓴다. 검증된 branch를 일반 push하고 새 PR을 만든다.
- 단순한 작성 요청은 `draft`로 처리한다.
- `publish`의 `target_pr_state`는 `draft`로 시작한다. 사용자가 Ready, non-draft 또는 즉시 review 가능한 상태를 명시했을 때만 `ready`로 정하고 근거를 payload에 남긴다. “PR을 올려 줘”, 검증 통과, 미디어 부재는 상태 지정으로 보지 않는다. 대상이 Draft PR을 지원하지 않으면 Draft 요청을 유지한 채 그 사실을 보고한다.
- 명시적인 publish 요청은 정확한 기존 remote로 대상 branch를 일반 push하는 권한을 포함한다. 같은 권한을 다시 묻지 않고, [경계](#경계)는 그대로 지킨다.

### 작성 단계와 확인 근거

단계의 산출물은 작성·게시 판단에 필요한 정보이지 별도 파일이나 PR 본문 체크리스트가 아니다. 입력·최종 diff·초안·도구 결과를 대조하며, 검토했다는 자기 보고를 근거로 삼지 않는다.

| 단계 | 작업과 산출물 | 완료를 판단할 근거 |
| --- | --- | --- |
| W1 대상·양식·권한 | 아래 번호 절차로 저장소, 최종 base/head와 전체 diff 범위, 양식·언어, draft/publish 및 미확인 조건을 정한다. | 조회 결과나 제공 자료와 일치하고 양식 미확인을 부재로 바꾸지 않는다. |
| W2 사실 수집 | [작성 지침](references/pr-writing.md)에 따라 티켓과 최종 diff에서 바뀐 동작·선택 이유·남은 작업을 확보한다. | 각 설명에 근거가 있으며 이전 초안과 다르면 최종 diff를 따른다. |
| W3 제목·본문 | 같은 지침으로 제목과 Why·Changes, 필요한 선택 절을 쓴다. | 코드와 티켓에 없는 결정만 남고 확인되지 않은 효과나 이유를 만들지 않는다. |
| W4 화면 자료 | [화면 자료 규칙](../../references/media.md)으로 필요 여부와 매체를 정하고 캡처·변환한다. | 보이는 변화에 맞는 매체를 고르고 준비 계획과 실제 생성·확인을 구분한다. |
| W5 최종 대조 | 초안을 최종 diff·양식·티켓 연결 문법·자료와 비교해 고치고 게시 전 조건을 확인한다. | 범위 불일치·요구 누락·연결 변조·하지 않은 검증이나 첨부의 완료 주장이 없다. |
| W6 게시·재조회 | publish일 때만 아래 번호 절차로 게시·첨부하고 결과를 남긴다. draft는 미게시 초안으로 끝낸다. | 원격 ID·URL·본문·연결·자료 표시를 실제 trace와 재조회로 확인한다. URL만으로 영상 재생을 단정하지 않는다. |

권한 확인→쓰기, 자료 검사→업로드, 쓰기→재조회의 의존 순서는 지킨다. diff나 자료가 달라지면 W2–W5를 다시 대조한다. 동등한 도구나 병렬 읽기는 허용하며 호출 순서만으로 이행 여부를 판단하지 않는다.

### PR을 준비하고 게시한다

1. **입력 확보.** 사용자가 base, head, 전체 diff 같은 확인된 조회 결과를 제공하고 추가 조회를 막았다면 그 자료를 입력으로 쓴다. 그 밖에는 [상태 수집](references/github.md#상태를-수집한다)의 `git`·`gh` 명령으로 대상 저장소, 명의, base·head, 전체 diff, 작업 트리, 양식, 기존 PR을 읽는다. 사용자가 원격 조회를 막았으면 `gh` 명령은 실행하지 않는다.

   | 확인 결과 | 행동 |
   |---|---|
   | 진행 중인 Git 작업, detached HEAD, base와 같은 head, 보낼 commit 없음 | 사유와 필요한 별도 Git 작업을 보고하고 그 PR은 만들지 않는다. |
   | 같은 저장소 head branch의 열린 PR 있음 | URL과 현재 상태를 보고하고 새 PR은 만들지 않는다. |
   | 커밋하지 않은 변경 있음 | 원격 PR에 포함되지 않는 변경으로 보고한다. |
   | 조회 실패 | 그 조회에 기대는 결정은 확정하지 않고 미확인으로 남긴다. |

   `git log <remote>/<base>..HEAD`와 `git diff <remote>/<base>...HEAD`를 다음 단계의 입력으로 쓴다.
2. **PR 경계.** [PR 작성 지침](references/pr-writing.md#pr-경계를-점검한다)으로 경계를 정하고, 여러 PR이 서로 의존하면 [stacked PR 규칙](references/stacked-prs.md)을 적용한다. 현재 branch에 여러 주제가 섞였거나 필요한 branch·commit이 없으면 PR 계획과 필요한 Git 작업을 제시하고, 그 작업이 준비되거나 사용자가 함께 허가한 뒤 게시한다.
3. **양식과 언어.** [PR 템플릿 규칙](references/pr-template.md)으로 `pullRequestTemplates` 조회 결과를 해석해 양식을 고르고 출력 언어를 정한다.
4. **본문.** [PR 작성 지침](references/pr-writing.md)에 따라 전체 diff와 현재 검증 근거로 제목·본문을 쓴다. commit 제목이나 `--fill` 결과는 참고만 한다. Writing·Fluent를 함께 쓸 때는 [작성 지침 함께 적용하기](../../references/writing-composition.md)에 따라 적용 양식과 확인 상태, 출력 언어, 근거·편집 범위와 보호할 연결 문법을 전달한다. 양식이 확인되기 전의 초안은 임시 초안으로 보고한다.
5. **티켓 연결.** 티켓 ID나 URL이 있거나 사용자가 연동을 요청하면 [티켓 연결 규칙](references/ticket-linking.md)을 따른다. 기존 branch 이름의 ID는 가장 낮은 신뢰도의 hint로만 쓰고, provider는 확인된 근거로 정한다. 지원하지 않는 tracker 연결이 필수이면 연결 없는 PR로 바꾸지 않고 범위를 알린다. 티켓 intent와 PR event의 status effect를 나누고, native automation이 처리하는 transition은 automation에 맡긴다.
6. **화면 자료.** [화면 자료 규칙](../../references/media.md)으로 필요 여부를 판단하고, 필요하면 캡처·변환해 `## Screenshots`에 배치한다. 보이는 변화가 없으면 그 절을 지운다.
7. **게시.** `publish`에서만 실행한다. 직전에 head SHA, 작업 트리, 같은 head의 기존 PR을 다시 읽고, 양식 출처·언어·티켓 연결·검증 상태·`target_pr_state`·필수 미디어가 현재 요청과 맞는지 대조한다. 단일 PR과 독립 PR은 [GitHub 게시 절차](references/github.md#생성하고-검증한다)를, native stack은 [stacked PR 게시 절차](references/stacked-prs.md#native-stack으로-게시하고-검증한다)를 따른다.
8. **결과 확인.** PR과 미디어를 다시 읽어 최종 payload와 대조하고, 가능하면 canonical ticket도 다시 읽어 link와 status effect를 확인한다.

## 결과

- PR마다 제목, 본문, base·head, 양식 출처와 확인 상태, 출력 언어, 티켓 연결, 검증 상태, 화면 자료 계획, `target_pr_state`를 쓴다.
- branch별 push, PR 생성, native stack 연결, 미디어별 업로드·본문 반영, ready 전환, 티켓 link와 status effect의 결과를 단계별로 보고한다.
- 준비한 초안, 실제 게시, 부분 성공을 구분한다. 실행하지 않은 검증과 확인하지 못한 상태는 미실행·미확인으로 쓴다.

## 예시

입력: “현재 branch로 PR 올려 줘.”

```bash
gh pr list --repo o/r --head feature/save-toast --state open --json number,url,isDraft,headRepositoryOwner
```

```json
[{"number": 42, "url": "https://github.com/o/r/pull/42", "isDraft": true, "headRepositoryOwner": {"login": "o"}}]
```

push 대상 저장소 `o/r`의 같은 branch에서 열린 PR #42가 이미 있다. URL과 Draft 상태를 보고하고 새 PR은 만들지 않는다.

대조: `gh repo view o/r --json pullRequestTemplates`가 인증 오류로 실패했다. 양식이 없다고 보지 않는다. 확인된 diff로 Why·Changes만 임시 초안으로 쓰고 양식 미확인을 본문 밖에 보고한다. `gh auth status`로 인증을 확인한 뒤 다시 조회해 양식을 정한다.

## 경계

- branch·worktree·commit 생성, branch rename, commit rewrite, rebase·squash, force push, merge는 각 담당 스킬이나 사용자의 별도 Git 작업으로 넘긴다.
- 필요한 객체나 ref가 로컬에 없거나 오래되었으면 사용자 요청 없이 fetch·checkout·branch 전환을 하지 않고 그 상태를 blocker로 보고한다.
- publish 시작 전에 존재하던 PR은 수정하지 않는다. 현재 publish 흐름에서 방금 만든 PR의 stack 연결, 검토한 미디어 첨부, 필수 첨부 검증 뒤 사용자가 명시한 ready 전환만 담당하며 이때 제목·티켓·reviewer·label과 다른 본문 내용은 유지한다.
- 코드 구현, 일반적인 작업 완료, 티켓 작성, code review 또는 push 요청만으로는 이 스킬을 시작하지 않는다.

## 참고 자료

- [GitHub PR 규칙](references/github.md): 상태 수집 명령, payload, publish 권한, 생성·검증
- [PR 템플릿 규칙](references/pr-template.md), [PR 작성 지침](references/pr-writing.md), [PR 기본 양식](assets/templates/pull-request.md)
- [stacked PR 규칙](references/stacked-prs.md), [티켓 연결 규칙](references/ticket-linking.md)
- [커밋 메시지 규칙](../../references/conventional-commits.md): PR 제목 형식
- [화면 자료 규칙](../../references/media.md), [호스트별 도구](../../references/hosts.md)
