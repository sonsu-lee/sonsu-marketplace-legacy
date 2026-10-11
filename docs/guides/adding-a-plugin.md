# 플러그인 개발·수정·추가하기

플러그인을 로컬에서 수정하고 검증하거나 새 플러그인을 마켓플레이스에 추가하는 절차입니다.
구성 요소와 로딩 경계는 [아키텍처 개요](../architecture/overview.md), 외부 원본을 갱신하는
절차는 [업스트림 업데이트 런북](../runbooks/updating-upstream-plugin.md)을 참고하세요.

## 로컬 개발 환경

저장소를 clone하고 사용하는 에이전트에서 로컬 카탈로그를 확인합니다.

```sh
git clone https://github.com/sonsu-lee/sonsu-marketplace.git
cd sonsu-marketplace
```

Codex 데스크톱 앱은 이 저장소의 `.agents/plugins/marketplace.json`을 발견합니다.
하지만 같은 이름의 Git 등록본이 있으면 로컬 플러그인 변경이 가려질 수 있습니다.
로컬 카탈로그와 Git 등록본이 모두 표시돼도 로컬 변경이 로드됐다는 뜻은 아닙니다.
Codex CLI에서 현재 체크아웃을 시험할 때는 Git 등록본이 없는 별도 설정에서 로컬 경로를
명시적으로 등록합니다. 저장소 루트에서 다음 명령을 실행하세요.

```sh
codex_test_home="$(mktemp -d)"
CODEX_HOME="$codex_test_home" codex plugin marketplace add .
CODEX_HOME="$codex_test_home" codex plugin list --marketplace sonsu-marketplace
```

이후 플러그인 설치와 Codex CLI 실행에도 같은 `CODEX_HOME`을 전달하세요.
데스크톱 앱에서 로컬 변경을 시험할 때에도 같은 이름의 Git 등록본이 없는 환경을 사용합니다.

Claude Code에서는 저장소의 절대 경로로 로컬 marketplace를 등록합니다.

```sh
claude plugin marketplace add "$(pwd -P)"
claude plugin marketplace list
```

GitHub 소스와 로컬 경로는 같은 `sonsu-marketplace` 식별자를 사용합니다. 실제 등록·설치
검증은 기존 사용자 설정과 분리된 환경에서 진행하세요.
설치·업데이트 뒤에는 Codex의 새 작업 또는 Claude Code의 새 세션에서 최신 스킬을 확인합니다.
omp는 이름 있는 profile로 기존 사용자 설정과 분리합니다. profile은 마켓플레이스 등록, 설치 플러그인,
설정과 인증을 `~/.omp/profiles/<name>/`에 따로 둡니다. 저장소 루트에서 다음을 실행합니다.

```sh
export OMP_PROFILE=sonsu-marketplace-local
omp plugin marketplace add "$(pwd -P)"
for plugin in git tickets review fluent-korean fluent-english fluent-japanese design; do omp plugin install "$plugin@sonsu-marketplace"; done
omp plugin list
```

같은 셸에서 `omp`를 실행하면 이 profile을 사용하며, 세션 실행에는 profile 안의 인증이 따로 필요합니다.
체크아웃을 바꾼 뒤에는 `omp plugin marketplace update sonsu-marketplace`와 `omp plugin upgrade`로
반영하고, 버전이 같으면 `install --force`로 다시 설치합니다. 검증이 끝나면 `unset OMP_PROFILE`을 실행하고
`~/.omp/profiles/sonsu-marketplace-local`을 삭제합니다. 기본 profile에 로컬 경로를 등록하지 않습니다.

## 기존 플러그인 수정

1. 해당 플러그인의 README와 매니페스트에서 수정할 스킬·hook·script의 진입점을 확인합니다.
   외부 원본이 포함되어 있으면 `UPSTREAM.md`에서 원본과 로컬 변경의 경계를 확인합니다.
2. 수정 대상의 정본을 갱신합니다. Fluent Languages는 각 `skills/fluent-<language>/SKILL.md`와
   해당 스킬의 참고 자료를 직접 편집합니다. 언어 사이에 공통 원본을 주입하지 않습니다.
   Design·Git·Tickets·Review·Fluent Korean의 omp 배포본은 `plugins/<name>/omp/`에 생성합니다.
   생성본과 설치 캐시는 직접 편집하지 않고 원본 스킬이나 `scripts/render-omp-compat.py` 변환을 고친 뒤
   `python3 scripts/render-omp-compat.py`로 재생성합니다. Fluent Korean은 `codex/skills/`의 단일 호출을
   투영하고 English·Japanese는 기존 패키지를 배포합니다.
3. 변경한 동작에 맞는 평가를 [evals/](../../evals)에서 선택하고 아래 검증을 실행합니다.
   사용법·계약·개발 절차가 달라졌다면 [문서 배치 기준](../README.md)에 따라 담당 문서를 갱신합니다.

## 새 플러그인 추가

### 사전 조건

- 추가할 플러그인의 이름, 출처와 라이선스를 확인합니다.
- 외부 플러그인이면 가져올 정확한 tag 또는 commit을 선택합니다.
- 기존 `plugins/`와 `.agents/plugins/marketplace.json`에서 같은 이름이 없는지 확인합니다.

### 절차

1. `plugins/<plugin-name>/`을 만들고 `.codex-plugin/plugin.json`을 작성합니다.
2. 필요한 구성 요소만 추가합니다. 스킬은 `skills/<skill-name>/SKILL.md`에 두고 매니페스트가
   해당 경로를 가리키게 합니다.
3. 외부 플러그인은 원본 파일과 실행 권한을 검증하고 `UPSTREAM.md`에 출처, 기준 commit,
   버전, 라이선스와 포함 범위를 기록합니다.
4. `.agents/plugins/marketplace.json`의 `plugins` 배열 끝에 등록합니다.
5. `python3 scripts/render-claude-compat.py`로 Claude Code catalog와 plugin manifest를 생성합니다.
6. `python3 scripts/render-omp-compat.py`로 omp catalog와 `ISOLATED`에 등록한 플러그인의 전용 패키지를 생성합니다.
   omp 기본 배포는 `git`, `tickets`, `review`, `fluent-korean`, `fluent-english`, `fluent-japanese`, `design` 7개로 고정합니다.
   Codex catalog에 추가해도 omp 배포 대상은 늘어나지 않습니다. 대상을 바꾸려면 배포 정책을 명시적으로 변경하고
   `scripts/render-omp-compat.py`의 `OMP_PLUGINS`와 `evals/plugin-compat/test_compat.py`의 omp 목록을 함께 갱신합니다.
   opt-in 패키지는 `OMP_OPTIN_PLUGINS`에 등록하면 카탈로그에 추가되며 사용자가 직접 설치합니다. 현재 대상은 `writing`, `research`, `prompting`, `product`([ADR 0024](../decisions/0024-distribute-writing-research-prompting-product-to-omp.md)), `design-patterns`([ADR 0026](../decisions/0026-ship-design-patterns-as-omp-opt-in.md)), `worklog`([ADR 0022](../decisions/0022-add-worklog-plugin.md))입니다.
   원본의 hook·연속성 실행기를 제외해야 하는 패키지는 `ISOLATED`에도 등록해 `plugins/<name>/omp` 사본과 omp 순정 연속성 안내를 생성합니다.
   runtime extension이 있는 플러그인만 `RUNTIME_EXTENSIONS`에도 등록합니다.

```json
{
  "name": "my-plugin",
  "source": {
    "source": "local",
    "path": "./plugins/my-plugin"
  },
  "policy": {
    "installation": "AVAILABLE",
    "authentication": "ON_INSTALL"
  },
  "category": "Productivity"
}
```

폴더명, 플러그인 매니페스트와 마켓플레이스 항목의 `name`은 같아야 합니다.
`source.path`는 저장소 루트 기준입니다.
`.agents/plugins/marketplace.json`과 plugin별 `.codex-plugin/plugin.json`이 패키지의 정본입니다.

omp 생성기는 필요한 스킬·참고 자료·asset·스크립트·Figma companion·라이선스와 Review의 reviewer 전용 규칙(`omp-rules/` → `rules/`)을
동봉하고 스크립트 실행 권한을 유지합니다. 독자 hook, evidence gate, `task-continuity.py`, runtime extension은
포함하지 않으며 연속성 자료를 omp 순정 todo·session 안내로 바꿉니다. 예외로 opt-in 패키지 `worklog`는
`omp-extension/worklog.ts`를 `extension/worklog.ts`로 복사하고 이를 선언하는 `package.json`을 생성합니다.
hook은 opt-in 패키지에도 포함하지 않습니다.
Design의 품질 계약·프로필, Git·Tickets의 권한, English·Japanese 스킬은 유지합니다.
Fluent Korean은 현재 호스트 모델로 단일 호출을 실행하며 Claude agent와 다중 호출·strict 모드에 의존하지 않습니다.
개발 실행·task·todo·session은 omp 순정 기능을 사용하고 리뷰는 Review 플러그인을 기본 배포합니다.
Research·Product·Writing·Prompting은 기본 배포와 분리해 opt-in으로 배포합니다. Dev Workflow의 omp 프로필은
직접 설치한 consumer용으로 보존하고 기본 설정에는 추가하지 않습니다. omp 대응을 위해 기존 Codex·Claude Code hook이나
연속성 자료를 변경하지 않습니다.

생성기의 `--check`는 생성물의 최신 상태와 불필요한 이전 생성물을 검사합니다. 일반 실행이 정리할 수 있는
파일은 생성기가 소유한다고 확인한 이전 extension 파일뿐입니다. 사용자 파일, `.sonsu`·`.engineering`
기록과 설치 캐시는 삭제하거나 편집하지 않습니다.

배포 대상 플러그인의 변경을 게시할 때는 해당 플러그인의 카탈로그 버전도 올립니다.
`marketplace.autoUpdate: auto` 업데이트는 omp 시작 시 오래된 카탈로그를 가져오는 것이 전제이며
같은 버전의 캐시를 실행 중에 바꾸는 기능은 아닙니다. 이전 구성의 제거와 설정 정리는
[이전 버전에서 이동하기](migrating-from-earlier-versions.md#이전-omp-구성에서-이동)를 참고하세요. 정리 뒤 세션을 재시작해 이전 hook·agent를 해제합니다.

## 검증

저장소 루트에서 다음 정적 검사를 실행합니다.

```sh
find .agents plugins evals .github -name '*.json' -print0 \
  | xargs -0 -n1 python3 -m json.tool >/dev/null
python3 scripts/render-shared-files.py --check
python3 scripts/render-agent-policy.py --check
python3 scripts/render-continuity.py --check
python3 scripts/render-claude-compat.py --check
python3 scripts/render-omp-compat.py --check
claude plugin validate . --strict
python3 evals/language-style/eval.py validate
python3 -m unittest -v evals/language-style/test_eval.py
python3 -B -m unittest discover -s evals/plugin-compat -p 'test_*.py' -v
python3 -B -m unittest discover -s evals/worklog -p 'test_*.py' -v
python3 -B -m unittest discover -s plugins/review/tests -p 'test_*.py' -v
python3 -B -m unittest discover -s plugins/dev-workflow/tests -p 'test_*.py' -v
python3 scripts/validate_refactor_inventory.py check
python3 -B -m unittest discover -s evals/refactor-inventory -p 'test_*.py' -v
git diff --check
```

플러그인을 추가하거나 구조를 변경했다면 `<plugin-name>`을 해당 이름으로 바꿔 개별 패키지도 검사합니다.

```sh
python3 -m json.tool .agents/plugins/marketplace.json
python3 -m json.tool plugins/<plugin-name>/.codex-plugin/plugin.json
```

JSON 문법과 참조 경로를 확인한 뒤 가능한 경우 Codex의 실제 플러그인 읽기 경로에서
이름, 버전과 구성 요소 목록을 확인합니다. 정적 검증을 실제 로딩 성공으로 간주하지 않습니다.
API 키와 토큰은 저장하지 않으며 필요한 환경 변수 이름만 `.env.example`에 기록합니다.
