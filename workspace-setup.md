# README for Agent

이 문서는 `<USER_WORKSPACE_ROOT>/` 디렉토리에서 작업하는 모든 에이전트가 따라야 하는 ground rule을 정의한다.
<USER_WORKSPACE_ROOT>란, 이 문서가 있는 디렉토리를 말한다.

## Ground Rules

### 1. Language Policy

- **Thinking / Working**: 영어로 사고하고 작업한다 (코드, 커밋 메시지, 파일명, 내부 추론 등).
- **Explanation to User**: 사용자에게 설명할 때는 **반드시 한국어**로 응답한다.

### 2. Work Logging (`log/<memo>`)

모든 작업은 추적 가능해야 한다. 각 작업 단위마다 `log/` 디렉토리에 memo 파일을 남긴다.

- **위치**: 각 프로젝트 폴더 내 `log/` 디렉토리 (e.g., `<ACTIVE_PROJECT>/log/`, `<NEW_PROJECT>/log/`).
- **파일명 규칙**: `YYYYMMDD_<idx:2d>_<short-slug>.md` (e.g., `20260521_01_task_summary.md`).
- **언어**: **한국어**로 작성한다.
- **필수 섹션**:
  - `## Objective` — 이 작업의 목표 (무엇을, 왜)
  - `## Conclusion` — 최종 결과 / 결론 (성공/실패, 다음 액션)
  - `## Intermediates` — 중간 과정 (시도한 것, 발견한 것, 막혔던 지점, 의사결정 근거)
  - 필요 시 `## References` — 관련 파일 경로, `<JOB_ID>`, 로그 위치, 외부 링크 등

#### Memo Template

```markdown
# <작업 제목>

- Date: YYYY-MM-DD
- Owner: <agent / user>
- Related: <project / experiment 식별자>

## Objective
<무엇을 달성하려 했는가, 왜>

## Conclusion
<결과 / 결정 사항 / 다음 액션>

## Intermediates
<시도, 발견, 막힘, 의사결정 근거>

## References
- <파일 경로 / job id / 로그 위치>
```

### 3. Project Folder Structure

- **Root**: `<USER_WORKSPACE_ROOT>/` 가 루트이다.
- **새 프로젝트 시작 시**:
  1. 먼저 루트 아래에 **프로젝트 폴더를 생성**한다 (e.g., `mkdir <NEW_PROJECT>`).
  2. 그 폴더 안에서 작업을 시작한다 — 루트에 코드/스크립트/로그를 직접 흩뿌리지 않는다.
  3. 프로젝트 폴더 안에는 최소한 `log/` 디렉토리를 둔다.
- **기존 프로젝트** (active):
  - `<ACTIVE_PROJECT>/` — `<PROJECT_PURPOSE>`
- **Archive**: 종료/레거시 프로젝트는 `archive/` 하위로 이동.

### 4. Scope of Work — `<USER_WORKSPACE>/` Only

- 모든 작업은 **`<USER_WORKSPACE_ROOT>/` 디렉토리 트리 내부에서만** 수행한다.
- 같은 워크스페이스 형제 디렉토리(e.g., `<SIBLING_WORKSPACE>/<OTHER_USER>/`, 다른 사용자 영역)는 **읽기·쓰기·이동·삭제 어떤 것도 하지 않는다**.
- 외부 잡(e.g., 다른 사용자의 `<JOB_SCHEDULER>` job, 다른 워크스페이스의 코드/데이터)은 컨텍스트 파악용으로만 조회하고, 절대 건드리지 않는다.
- 예외 없이 `cwd` 와 모든 파일 작업의 경로는 `<USER_WORKSPACE>/` 로 시작해야 한다.
- 시스템 계정 `<SHARED_SYSTEM_ACCOUNT>` 은 여러 사용자가 공유한다. 공유 자원(`~/.gitconfig`, `~/.ssh/`, `~/.bashrc` 등)은 **수정 금지**. 본인 설정은 `<USER_WORKSPACE>/` 안에서만 관리.

### 5. Keep It Simple

복잡함은 실수를 낳는다. 모든 영역에 단순함을 우선한다.

- **명령 실행**: 한 번에 한 단계만. 여러 명령을 `&&`/`;`로 길게 chain 하지 않는다. 단순 명령 → 결과 확인 → 다음 단계.
- **코드 작성**: 가장 단순한 구현을 우선. 추상화/일반화/optional 기능을 미리 도입하지 않는다. 줄 수가 적고 한눈에 읽히는 코드가 우선.
- **스크립트/설정**: 검증되지 않은 옵션, "혹시 모르니" 추가하는 fallback, 불필요한 환경변수, 과도한 에러 핸들링을 넣지 않는다.
- **작업 계획**: 한 가지를 끝내고 다음으로. 동시에 여러 변경을 묶지 않는다.
- **질문**: 필요한 것만 묻는다. 옵션을 과하게 나열하지 않는다.
- **문서**: 짧게. 필요한 만큼만.

원칙: **단순함이 안전이다.**

### 6. Parallelize Independent Work

독립적인 작업은 **병렬**로. 직렬은 의존이 있을 때만.

- 여러 파일을 동시에 읽기 / 여러 파일을 동시에 편집 / 여러 명령을 동시에 실행 — 의존 없으면 한 응답에 묶어서 동시 호출.
- 백그라운드 잡 (`<JOB_SCHEDULER>` submit + poll) 을 띄우고 그 사이에 다른 작업 (memo update, 코드 조사) 진행.
- 의존이 있으면 (e.g., submit 결과 `<JOB_ID>` 가 있어야 poll 가능) sequential.
- ground rule 5 와 모순 아님: "한 작업 안에서는 단순하게, 작업 간에는 병렬로".
- **Sub agent 활용**: 독립적이면서 약간 무거운 작업(코드베이스 조사, 다중 파일 분석, 패치 호환성 확인 등)은 `<SUB_AGENT_TOOL>` 로 sub agent 에 위임. 메인은 다른 일 (잡 폴링, memo, 새 config 작성) 계속. 결과는 한 번에 요약으로 돌아옴.
- **코드 패치를 동반한 병렬 작업은 worktree 필수**: 같은 코드 트리에 patch 를 가하면서 다른 잡/작업이 그 코드를 동시에 보면 결과가 섞인다. 두 patch 버전 동시 비교, 또는 잡 폴링 중에 코드 패치가 필요한 모든 상황에서 `git worktree add <path> <branch>` 로 별도 작업 트리를 만들고 그 안에서 patch + 잡 submit. 운영 트리는 `<ACTIVE_PROJECT>/code/<TRAINING_FRAMEWORK_REPO>` — 패치 작업이 병렬과 겹치면 이 트리의 worktree 를 분리하고 그 path 를 `TARGET_TRAINING_REPO` 로 가리키도록 잡을 제출.

### 7. Git Identity — Per-Repo Local Config

`<SHARED_SYSTEM_ACCOUNT>` 계정이 공유되고 글로벌 `~/.gitconfig` 도 placeholder 상태이므로, **`<USER_WORKSPACE>/` 아래 모든 git repo에서 본인 identity 를 per-repo 로 명시**한다.

**Identity**:
- `user.name`: `<USER_ID>`
- `user.email`: `<USER_EMAIL>`

**적용 절차** — repo 를 clone 하거나 새로 만든 직후 즉시 실행:

```bash
cd <repo-path>
git config user.name "<USER_ID>"
git config user.email "<USER_EMAIL>"
```

확인:

```bash
git config --local --get user.name
git config --local --get user.email
```

**중요**:
- `--global` 플래그를 절대 사용하지 않는다.
- 모든 commit 전에 author 가 `<USER_ID>` identity 인지 확인.
- credential / SSH key 분리는 별도 정책 필요.

## Quick Checklist

- [ ] 이 작업이 어느 프로젝트에 속하는가? 새 프로젝트라면 폴더 먼저 생성.
- [ ] `log/` 디렉토리에 memo 파일을 만들고 Objective를 먼저 적었는가?
- [ ] 작업 완료 후 같은 memo에 Conclusion과 Intermediates를 채웠는가?
- [ ] 사용자에게 한국어로 보고했는가?
