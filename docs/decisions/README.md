# 결정 기록

이 디렉터리는 아키텍처뿐 아니라 저장소와 Codex 작업 방식에 장기간 영향을 주는 결정을
보관합니다.

## 기록 기준

다음 중 하나에 해당하면 결정 기록을 고려합니다.

- 되돌리기 어렵거나 변경 비용이 큽니다.
- 실제 대안 사이에 중요한 trade-off가 있습니다.
- 여러 후속 작업이 이 결정을 전제로 합니다.
- 나중에 선택 이유를 복원하기 어렵습니다.
- Codex가 반복해서 따라야 하는 저장소 정책입니다.

작고 쉽게 되돌릴 수 있는 변경, 일회성 실험, 아직 결론이 나지 않은 아이디어와 다른 문서에
이미 명확히 기록된 내용은 결정 기록을 만들지 않습니다.

## 형식과 상태

파일 이름은 `NNNN-<decision>.md` 형식입니다. 문서는 Context, Decision, Alternatives
Considered, Consequences와 Revisit When을 포함합니다.

상태는 `Proposed`, `Accepted`, `Rejected`, `Superseded` 중 하나를 사용합니다. 결정을
대체할 때에는 이전 파일을 덮어쓰지 않고 이전 문서와 새 문서에 서로의 번호를 기록합니다.
사용자가 제거를 명시적으로 승인한 이전 플러그인 정체성을 직접 포함한 기록은 예외적으로
[0009 결정](0009-maintain-engineering-as-an-independent-plugin.md)처럼 새 결정에 대상 번호,
삭제 이유와 현재 계약을 기록한 뒤 이전 파일을 제거할 수 있습니다.

## 현재 결정

- [0001 Use Local Marketplace](0001-use-local-marketplace.md)
- [0002 Separate Document and Commit Approval](0002-separate-doc-and-commit-approval.md)
- [0003 Keep Marketplace Plugins Independent](0003-keep-plugins-independent.md)
- [0004 Keep Research Independent](0004-keep-research-independent.md)
- [0006 Keep Prompting as an Independent Plugin](0006-keep-prompting-independent.md)
- [0007 Use Stage-Owned Quality Gates with Bounded Backtracking](0007-use-stage-owned-quality-gates.md)
- [0008 Add an Independent Product Plugin](0008-add-product-plugin.md)
- [0009 Maintain Engineering as an Independent Plugin](0009-maintain-engineering-as-an-independent-plugin.md)
- [0010 Add Figma Workflow Plugin](0010-add-figma-workflow-plugin.md)
- [0011 Use Fast Paths and Plan-Backed Red-Team Gates](0011-use-fast-path-and-plan-red-team-gates.md)
- [0012 Use Role Routing and Execution Evidence](0012-use-role-routing-and-execution-evidence.md)
- [0013 Add an Independent Design Patterns Plugin](0013-add-design-patterns-plugin.md)

- [ADR 0014: Codex 전용 Engineering과 관리형 품질 게이트](0014-use-codex-managed-engineering.md)
- [ADR 0015: 독립 스킬 중심 플러그인 경계](0015-independent-skills.md)
- [ADR 0016: Claude Code 마켓플레이스 배포와 호스트별 모델 프로필](0016-support-claude-code.md)
- [ADR 0017: 두 호스트가 공유하는 로컬 Markdown 메모리](0017-use-shared-local-memory.md)

- [ADR 0018: UI 디자인 플러그인 통합](0018-consolidate-ui-design.md)
- [ADR 0019: Design 레퍼런스 검색 스킬 추가](0019-add-design-reference-search.md)
- [ADR 0020: Career 플러그인 추가](0020-add-career-plugin.md)
- [ADR 0021: 요구사항·설계·티켓·계획 하이브리드 인계](0021-use-hybrid-product-delivery-handoff.md)
- [ADR 0022: 세 호스트 공용 작업 로그 worklog](0022-add-worklog-plugin.md)
- [ADR 0023: Career 플러그인 제거](0023-remove-career-plugin.md)
- [ADR 0024: Writing·Research·Prompting·Product의 omp 선택 배포](0024-distribute-writing-research-prompting-product-to-omp.md)
- [ADR 0025: Engineering 리뷰 진입점 통합](0025-merge-engineering-review-entry-points.md)
- [ADR 0026: Design Patterns를 omp opt-in 패키지로 배포](0026-ship-design-patterns-as-omp-opt-in.md)
- [ADR 0027: Workflow·Engineering을 Git·Tickets·Review·Dev Workflow로 분리](0027-split-workflow-and-engineering-plugins.md)
- [ADR 0028: 수집 스크립트 대신 모델이 모르는 사실만 스킬에 쓴다](0028-prefer-cli-over-bundled-scripts.md)
