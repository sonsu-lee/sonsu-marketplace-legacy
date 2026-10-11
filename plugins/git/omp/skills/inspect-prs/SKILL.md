---
name: inspect-prs
description: 열린 GitHub PR 목록이나 지정 PR의 CI·리뷰·병합 상태·미해결 대화를 읽기 전용으로 확인할 때 사용한다. 코드 품질 리뷰, 충돌·CI 수정, 댓글 게시와 병합은 다른 스킬이 맡는다.
---

# PR 상태 조회

지정 PR 하나 또는 요청한 저장소의 열린 PR을 읽기 전용으로 조회하고, head와 조회 시점에 연결된 근거로 상태를 보고한다. PR 상태 확인·목록 정리 요청에서 자동 선택되며 이름으로 직접 호출할 수도 있다.

## 절차

1. 직접 호출이면 사용자가 고른 대상과 범위를 따른다. 요청 목적이 이 스킬과 다르면 그 차이를 설명한다.
2. [대상 확인](../../references/pr-inspection.md#대상-확인)에 따라 host, `OWNER/REPO`, PR을 정한다. 대상을 생략하면 현재 저장소로 정하고, 후보가 여럿일 때만 질문한다.
3. [수집](../../references/pr-inspection.md#수집)의 `gh` 명령으로 PR 메타데이터, check, 필수 check, 미해결 대화를 읽는다. 열린 PR 목록은 `gh pr list`로 읽고 상세 근거가 필요한 PR만 다시 읽는다. 반응과 리뷰 요청 기록은 보고에 필요할 때만 읽는다.
4. 결과에 따라 행동한다.

   | 결과 | 행동 |
   |---|---|
   | 모든 명령 성공, 마지막 `headRefOid`가 처음과 같음 | 수집한 근거로 보고한다. |
   | 일부 명령 실패 | 확인한 범위와 실패한 명령을 함께 보고한다. 실패한 항목을 빈 결과로 쓰지 않는다. |
   | 목록 결과 수가 `--limit`과 같음 | 확인한 PR 수를 전체 수로 쓰지 않는다. |
   | 마지막 `headRefOid`가 처음과 다름 | 새 head로 다시 수집하거나 혼합 관찰이라고 표시한다. |
   | `gh auth status` 실패 | `gh` 설치·인증이 필요하다고 사용자에게 알린다. |

5. [해석](../../references/pr-inspection.md#해석)에 따라 Draft, base/head SHA, `mergeStateStatus`, check `bucket`별 수, 필수 check, `reviewDecision`과 리뷰별 commit, 미해결 review thread를 각각 기록한다. 이전 commit의 리뷰와 현재 head의 리뷰를 나눠 쓴다.
6. 여러 단계 조회는 [작업 연속성](../../references/continuity.md)에 이어서 기록한다. 쓰기가 금지된 환경이면 기록을 생략한다.

## 결과

- 머리말: 조회 시점, host와 저장소, 범위(지정 PR 또는 열린 PR 목록), 확인한 PR 수와 누락 범위
- PR 표: 링크, Draft, head SHA, 병합 상태, CI, 리뷰, 미해결 대화. 목록은 요약하고 상세 근거는 필요한 PR에만 붙인다.
- CI 칸은 `pass`, `fail`, `pending`, `skipping`, `cancel`, check 없음, 조회 실패를 각각 센다. `pass`만 성공으로 센다.
- `CLEAN`·`UNSTABLE` 같은 병합 상태는 GitHub가 보고한 값으로 쓰고, 병합 승인이나 준비 완료 판정과 구분한다.
- 봇의 PR 본문 👍는 부가 정보로만 쓴다. 현재 head 검토, CI 통과, 병합 조건 충족의 근거로 쓰지 않는다.
- 리뷰 응답 지연 PR과 검토자·요청 시각. 판정은 [리뷰 응답 지연](../../references/pr-inspection.md#리뷰-응답-지연)을 따르고, 판정할 수 없으면 미확인으로 쓴다.

## 예시

입력: `ghe.example.com의 tools/widget에서 PR 31 상태를 알려 줘.`

```bash
gh pr view 31 --repo ghe.example.com/tools/widget --json isDraft,headRefOid,mergeStateStatus,reviewDecision
gh pr checks 31 --repo ghe.example.com/tools/widget --json name,bucket
gh pr checks 31 --repo ghe.example.com/tools/widget --required --json name,bucket
```

출력 발췌:

```json
{"isDraft": false, "headRefOid": "4f1c2e…", "mergeStateStatus": "CLEAN", "reviewDecision": "REVIEW_REQUIRED"}
[{"name": "test", "bucket": "pass"}, {"name": "deploy-preview", "bucket": "skipping"}]
[{"name": "test", "bucket": "pass"}]
```

review thread 조회에서 `isResolved: false`, `isOutdated: true`인 thread가 하나 나왔다.

보고:

| PR | Draft | head | 병합 상태 | CI | 리뷰 | 미해결 대화 |
|---|---|---|---|---|---|---|
| #31 | 아니요 | `4f1c2e…` | `CLEAN`(GitHub 보고값) | pass 1, skipping 1. 보고된 필수 check 1개 pass, 필수 check 전체 여부 미확인 | 리뷰 필요 | 1(outdated) |

outdated thread도 미해결 대화로 센다.

대조: `gh pr checks 31 --required`가 HTTP 403으로 실패했으면 필수 check를 "미확인"으로 쓴다. "필수 check 없음"으로 쓰지 않는다.

## 경계

- 코드 수정, CI 재실행, 댓글 게시, 대화 해결, 병합 같은 후속 복구는 자동으로 실행하지 않는다. 필요하면 `repair-pr` 같은 담당 스킬로 넘긴다.
- 인증 실패 시 로그인·계정 전환·설치를 실행하지 않는다.

## 참고 자료

- [GitHub PR 조회](../../references/pr-inspection.md): 수집 명령, 해석 규칙, 리뷰 응답 지연
- [작업 연속성](../../references/continuity.md)
- [호스트별 도구](../../references/hosts.md)
