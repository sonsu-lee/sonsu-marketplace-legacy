# Git

Git branch·commit·push와 GitHub PR 작성·상태 조회·복구를 승인 범위 안에서 수행하고 초안·실제 반영·미확인을 구분해 보고합니다.

## 설치

```bash
codex plugin add git@sonsu-marketplace
claude plugin install git@sonsu-marketplace
omp plugin install git@sonsu-marketplace
```

## 스킬

| 스킬 | 사용할 때 | 결과 |
| --- | --- | --- |
| `branch` | 브랜치 이름을 제안·검토하거나 새 branch 생성을 요청할 때 | 규칙에 맞는 이름 또는 생성된 branch와 확인된 HEAD |
| `commit` | 메시지를 준비하거나 요청한 변경만 commit할 때 | Conventional Commits 메시지 또는 새 commit과 남은 변경 |
| `push` | 현재 commit을 정확한 remote ref로 보낼 때 | 실제 원격 SHA 또는 미확인 상태 |
| `write-pr` | 새 PR 초안을 준비하거나 게시할 때 | PR payload 또는 게시 단계별 결과 |
| `inspect-prs` | 열린 PR이나 지정 PR의 CI·리뷰·병합 상태를 볼 때 | head와 조회 시점이 연결된 상태 보고, 응답이 늦은 리뷰 요청 |
| `repair-pr` | PR 충돌·리뷰 코멘트·실패 CI를 처리할 때 | 해결·기각·추가 확인 필요와 원격 반영 결과 |

“열린 PR 상태 정리해 줘”처럼 자연어로 요청하거나 `$inspect-prs`처럼 이름으로 호출합니다. omp에서는 `/skill:inspect-prs`로 호출합니다. 이름 지정은 commit·push·댓글·대화 해결 권한을 추가하지 않습니다.

## 사용 예시

요청: “현재 branch의 PR 초안만 준비해 줘. 게시는 하지 마.”

`write-pr`가 `git`·`gh` 명령으로 base·head·commit 범위와 저장소 PR 양식을 읽고, 원격 쓰기 없이 Conventional Commits 형식의 제목과 `Why`·`Changes` 본문, 미확인 항목을 보고합니다. 보이는 변화가 있으면 [화면 자료 기준](references/media.md)대로 Before/After를 준비합니다.

## 구성

- 공통 기준: [Git 안전 규칙](references/git-safety.md), [전달 권한](references/delivery-authority.md), [새 브랜치 이름](references/branch-naming.md), [커밋 메시지 규칙](references/conventional-commits.md), [문장 형식](references/tracker-prose.md), [화면 자료](references/media.md), [호스트별 도구](references/hosts.md), [PR 조회](references/pr-inspection.md), [리뷰 지적 대응](references/responding-to-review.md)
- 티켓 작성과 lifecycle 변경은 `tickets` 플러그인이, 코드 리뷰는 `review` 플러그인이 담당합니다.
- 공유 기준은 `shared/`의 정본에서 생성기로 복사합니다. 생성물은 직접 고치지 않습니다.
- [작업 연속성](references/continuity.md)은 여러 단계 작업의 계약·진행·근거 위치를 `.sonsu/continuity/`에 기록합니다. omp 배포본은 omp 순정 todo·session을 씁니다.
- 작성 지침·양식의 출처 고지는 [MIT 고지](WRITING_LICENSE.md)에, Conventional Commits 인용과 설계 참고는 [UPSTREAM.md](UPSTREAM.md)에 있습니다.

## 검증

```bash
python3 scripts/render-shared-files.py --check
python3 scripts/render-omp-compat.py --check
```
