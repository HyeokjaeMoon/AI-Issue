# Step 1. gh CLI로 레포 · 이슈 · 레이블 · 마일스톤 만들기

이 문서는 `AI-Issue` 프로젝트를 만들면서 실제로 실행한 명령어를 순서대로 정리한 것입니다.
각 명령어 아래에 **한 줄씩** 무슨 역할을 하는지 설명했습니다.

- 사용 도구: `git`, `gh` (GitHub CLI)
- 실행 환경: Windows 11, PowerShell / Git Bash
- 대상 레포: https://github.com/HyeokjaeMoon/AI-Issue

---

## 0. 사전 확인

### 0-1. 로그인 상태 확인

```powershell
gh auth status
```

| 부분 | 설명 |
|---|---|
| `gh` | GitHub CLI 프로그램을 실행합니다. |
| `auth` | 인증(로그인) 관련 명령 그룹입니다. |
| `status` | 지금 어떤 GitHub 계정으로 로그인되어 있는지 보여 줍니다. 로그인이 안 되어 있으면 `You are not logged into any GitHub hosts`가 나옵니다. |

### 0-2. 로그인 (사용자가 직접 실행)

```powershell
gh auth login
```

| 부분 | 설명 |
|---|---|
| `gh auth login` | GitHub 계정에 로그인합니다. 브라우저 인증이 필요해서 터미널에서 직접 실행해야 합니다. |
| 선택 순서 | `GitHub.com` → `HTTPS` → `Login with a web browser`를 고르고, 화면에 나온 코드를 브라우저에 입력하면 끝납니다. |

### 0-3. 폴더 상태 확인

```powershell
Get-ChildItem -Force | Select-Object Mode,Name
Test-Path .git
```

| 줄 | 설명 |
|---|---|
| `Get-ChildItem -Force` | 현재 폴더의 파일 목록을 보여 줍니다. `-Force`는 숨김 파일(`.git` 등)까지 보여 줍니다. |
| `\| Select-Object Mode,Name` | 출력 중 속성(Mode)과 이름(Name)만 골라서 보여 줍니다. |
| `Test-Path .git` | `.git` 폴더가 있는지 확인합니다. `False`면 아직 git 저장소가 아니라는 뜻입니다. |

---

## 1. 로컬 git 저장소 만들기

```powershell
git init -b main
Set-Content -Path README.md -Value "# AI-Issue" -Encoding utf8
git add README.md
git commit -m "Initial commit"
git log --oneline
```

| 줄 | 설명 |
|---|---|
| `git init -b main` | 현재 폴더를 git 저장소로 만듭니다. `-b main`으로 기본 브랜치 이름을 `main`으로 정합니다. |
| `Set-Content -Path README.md -Value "# AI-Issue" -Encoding utf8` | `README.md` 파일을 만들고 `# AI-Issue`라는 제목 한 줄을 씁니다. 한글이 깨지지 않게 UTF-8로 저장합니다. |
| `git add README.md` | `README.md`를 다음 커밋에 넣을 대상(스테이징 영역)으로 올립니다. |
| `git commit -m "Initial commit"` | 스테이징된 내용을 첫 번째 커밋으로 저장합니다. `-m` 뒤가 커밋 메시지입니다. |
| `git log --oneline` | 커밋 기록을 한 줄씩 짧게 보여 줍니다. 커밋이 잘 만들어졌는지 확인하는 용도입니다. |

> 원격 레포에 푸시하려면 커밋이 최소 1개 있어야 합니다. 그래서 README로 첫 커밋을 먼저 만들었습니다.

---

## 2. 원격 레포 만들고 푸시하기

```powershell
gh repo create AI-Issue --public --source . --push
git remote -v
```

| 부분 | 설명 |
|---|---|
| `gh repo create` | GitHub에 새 레포를 만듭니다. |
| `AI-Issue` | 만들 레포 이름입니다. 로그인한 계정 아래에 `HyeokjaeMoon/AI-Issue`로 만들어집니다. |
| `--public` | 누구나 볼 수 있는 공개 레포로 만듭니다. 비공개는 `--private`입니다. |
| `--source .` | 현재 폴더(`.`)의 로컬 저장소를 이 원격 레포와 연결합니다. 이때 `origin`이라는 이름의 원격이 자동으로 등록됩니다. |
| `--push` | 만들자마자 로컬 커밋을 원격으로 올리고, `main`이 `origin/main`을 따라가도록 설정합니다. |
| `git remote -v` | 연결된 원격 저장소 주소를 보여 줍니다. `fetch`(받기)와 `push`(올리기) 주소가 각각 나옵니다. |

---

## 3. 같은 이름의 레포가 있는지 확인하기

```powershell
gh repo view HyeokjaeMoon/ai-issue --json name,url,visibility,hasIssuesEnabled
```

| 부분 | 설명 |
|---|---|
| `gh repo view` | 레포 정보를 조회합니다. |
| `HyeokjaeMoon/ai-issue` | `계정/레포이름` 형식의 조회 대상입니다. |
| `--json name,url,visibility,hasIssuesEnabled` | 이름, 주소, 공개 여부, 이슈 기능 사용 여부만 JSON으로 출력합니다. |

> 소문자 `ai-issue`로 조회했는데 `AI-Issue`가 나왔습니다. **GitHub 레포 이름은 대소문자를 구분하지 않기 때문에** `ai-issue`를 새로 만들 수 없었습니다.

---

## 4. 첫 이슈 만들기

```powershell
gh issue create --repo HyeokjaeMoon/AI-Issue --title "배포됨" --body "배포됨"
```

| 부분 | 설명 |
|---|---|
| `gh issue create` | 새 이슈를 만듭니다. |
| `--repo HyeokjaeMoon/AI-Issue` | 이슈를 만들 레포를 지정합니다. 레포 폴더 안에서 실행하면 생략해도 됩니다. |
| `--title "배포됨"` | 이슈 제목입니다. |
| `--body "배포됨"` | 이슈 본문입니다. 생략하면 편집기가 열려서 자동 실행이 멈추므로 직접 넣었습니다. |

---

## 5. 현재 상태 확인하기 (레이블 · 마일스톤 만들기 전)

```bash
gh issue view 1 --json title
gh label list --limit 50
gh api repos/HyeokjaeMoon/AI-Issue/milestones
```

| 줄 | 설명 |
|---|---|
| `gh issue view 1 --json title` | 1번 이슈의 제목만 조회합니다. 한글 제목이 깨지지 않고 저장됐는지 확인하는 용도입니다. |
| `gh label list --limit 50` | 레포에 있는 레이블을 최대 50개까지 보여 줍니다. 새 레포에는 `bug`, `enhancement` 같은 기본 레이블 9개가 들어 있습니다. |
| `gh api repos/HyeokjaeMoon/AI-Issue/milestones` | GitHub REST API를 직접 호출해 마일스톤 목록을 가져옵니다. `gh`에는 마일스톤 전용 명령이 없어서 `gh api`를 씁니다. `[]`이면 하나도 없다는 뜻입니다. |

---

## 6. 레이블 만들기

스크립트 안에서 반복을 줄이려고 함수로 묶었습니다.

```bash
REPO="HyeokjaeMoon/AI-Issue"
lbl() { gh label create "$1" --repo "$REPO" --color "$2" --description "$3" --force; }
lbl "area:data-io" "1d76db" "데이터 입출력 및 전처리"
```

| 줄 / 부분 | 설명 |
|---|---|
| `REPO="HyeokjaeMoon/AI-Issue"` | 대상 레포 이름을 변수에 저장해 아래 명령에서 계속 씁니다. |
| `lbl() { ... }` | `lbl`이라는 셸 함수를 정의합니다. `$1`, `$2`, `$3`은 함수에 넘긴 첫째·둘째·셋째 값입니다. |
| `gh label create "$1"` | 이름이 `$1`인 레이블을 만듭니다. |
| `--color "$2"` | 레이블 색상입니다. `#` 없이 6자리 16진수로 씁니다. |
| `--description "$3"` | 레이블에 마우스를 올리면 보이는 설명입니다. |
| `--force` | 같은 이름의 레이블이 이미 있으면 에러를 내지 않고 색·설명을 덮어씁니다. 그래서 여러 번 실행해도 안전합니다. |
| `lbl "area:data-io" "1d76db" "..."` | 함수를 실제로 호출하는 예시입니다. |

만든 레이블 10개:

| 분류 | 레이블 | 색상 | 설명 |
|---|---|---|---|
| 종류 | `test` | `bfd4f2` | 테스트 코드 추가 또는 개선 |
| 종류 | `refactor` | `c5def5` | 동작 변경 없는 코드 구조 개선 |
| 영역 | `area:data-io` | `1d76db` | 데이터 입출력 및 전처리 |
| 영역 | `area:descriptive` | `0e8a16` | 기술통계 |
| 영역 | `area:hypothesis` | `5319e7` | 가설검정 |
| 영역 | `area:regression` | `b60205` | 회귀분석 |
| 영역 | `area:visualization` | `fbca04` | 시각화 |
| 우선순위 | `priority:high` | `d93f0b` | 우선순위 높음 |
| 우선순위 | `priority:medium` | `f9d0c4` | 우선순위 보통 |
| 우선순위 | `priority:low` | `ededed` | 우선순위 낮음 |

`bug`, `enhancement`, `documentation`은 기본 레이블을 그대로 썼습니다.

---

## 7. 마일스톤 만들기

```bash
ms() { gh api "repos/$REPO/milestones" -f title="$1" -f due_on="$2" -f description="$3" --jq .title; }
M1="Week 1 - 데이터 처리 & 기술통계"
M2="Week 2 - 추론통계 & 시각화"
ms "$M1" "2026-11-13T23:59:59Z" "2026-11-09 ~ 2026-11-13: 프로젝트 기반, 데이터 로딩/전처리, 기술통계 모듈 완성"
ms "$M2" "2026-11-20T23:59:59Z" "2026-11-16 ~ 2026-11-20: 가설검정, 회귀분석, 시각화 및 문서화"
```

| 줄 / 부분 | 설명 |
|---|---|
| `ms() { ... }` | 마일스톤을 만드는 셸 함수 `ms`를 정의합니다. |
| `gh api "repos/$REPO/milestones"` | 마일스톤 API 주소로 요청을 보냅니다. |
| `-f title="$1"` | `-f`는 요청 본문에 값을 넣는 옵션입니다. `-f`가 하나라도 있으면 자동으로 **POST(생성)** 요청이 됩니다. |
| `-f due_on="$2"` | 마감일입니다. ISO 8601 형식(`YYYY-MM-DDTHH:MM:SSZ`)으로 써야 합니다. GitHub 화면에는 날짜만 표시됩니다. |
| `-f description="$3"` | 마일스톤 설명입니다. |
| `--jq .title` | 응답 JSON에서 `title` 값만 꺼내 출력합니다. 잘 만들어졌는지 바로 확인할 수 있습니다. |
| `M1=...`, `M2=...` | 마일스톤 제목을 변수에 저장합니다. 이슈를 만들 때 **같은 제목**을 다시 써야 하므로 변수로 관리합니다. |
| `ms "$M1" ...` | 1주차 마일스톤(마감 11/13)을 만듭니다. |
| `ms "$M2" ...` | 2주차 마일스톤(마감 11/20)을 만듭니다. |

---

## 8. 이슈 만들기 (마일스톤 · 레이블 함께 지정)

```bash
iss() { gh issue create --repo "$REPO" --title "$1" --milestone "$2" --label "$3" --body "$4"; }
iss "CSV/Excel 데이터 로더 구현" "$M1" "enhancement,area:data-io,priority:high" "## 목표 ..."
```

| 부분 | 설명 |
|---|---|
| `iss() { ... }` | 이슈를 만드는 셸 함수 `iss`를 정의합니다. |
| `--title "$1"` | 이슈 제목입니다. |
| `--milestone "$2"` | 이슈를 넣을 마일스톤입니다. **마일스톤 제목과 정확히 같아야** 합니다. |
| `--label "$3"` | 붙일 레이블입니다. 여러 개는 쉼표로 구분합니다. 레포에 없는 레이블을 쓰면 에러가 납니다. |
| `--body "$4"` | 이슈 본문입니다. 마크다운을 쓸 수 있습니다. `- [ ]`는 체크박스가 됩니다. |

본문 안의 백틱(`` ` ``)은 셸이 명령 실행으로 해석하지 않도록 `` \` ``처럼 역슬래시로 이스케이프했습니다.

만든 이슈:

| 번호 | 마일스톤 | 제목 | 레이블 |
|---|---|---|---|
| #2 | Week 1 | 프로젝트 초기 구조 및 개발 환경 설정 | enhancement, priority:high |
| #3 | Week 1 | CSV/Excel 데이터 로더 구현 | enhancement, area:data-io, priority:high |
| #4 | Week 1 | 결측치·이상치 처리 유틸리티 | enhancement, area:data-io, priority:medium |
| #5 | Week 1 | 기술통계 함수 구현 (평균, 중앙값, 분산, 사분위수) | enhancement, area:descriptive, priority:high |
| #6 | Week 1 | 기술통계 모듈 단위 테스트 작성 | test, area:descriptive, priority:medium |
| #7 | Week 1 | [Bug] 표본 분산 계산 시 ddof 기본값이 0으로 적용됨 | bug, area:descriptive, priority:high |
| #8 | Week 2 | 가설검정 기능 구현 (t-검정, 카이제곱 검정) | enhancement, area:hypothesis, priority:high |
| #9 | Week 2 | 선형회귀 분석 모듈 구현 | enhancement, area:regression, priority:medium |
| #10 | Week 2 | 통계 시각화 함수 구현 (히스토그램, 박스플롯, 산점도) | enhancement, area:visualization, priority:medium |
| #11 | Week 2 | README 사용 예제 및 API 문서 작성 | documentation, priority:low |

### 스크립트 실행

```bash
bash setup_issues.sh
```

| 부분 | 설명 |
|---|---|
| `bash setup_issues.sh` | 6~8단계의 명령을 모아 둔 스크립트 파일을 한 번에 실행합니다. |
| `set -euo pipefail` (스크립트 첫 줄) | 명령이 하나라도 실패하면 즉시 멈추게 합니다(`-e`). 정의 안 된 변수를 쓰면 에러를 냅니다(`-u`). 파이프 중간 명령이 실패해도 실패로 처리합니다(`pipefail`). |

---

## 9. 중간 실패 확인 후 이어서 실행하기

첫 실행에서 #5까지 만든 뒤 **네트워크 타임아웃**으로 스크립트가 멈췄습니다.

### 9-1. 어디까지 만들어졌는지 확인

```bash
gh issue list --state all --json number,title --jq '.[] | "\(.number) \(.title)"'
```

| 부분 | 설명 |
|---|---|
| `gh issue list` | 이슈 목록을 조회합니다. |
| `--state all` | 열린 이슈와 닫힌 이슈를 모두 가져옵니다. 기본값은 열린 이슈만입니다. |
| `--json number,title` | 번호와 제목만 JSON으로 받습니다. |
| `--jq '.[] \| "\(.number) \(.title)"'` | 배열을 하나씩 꺼내 `번호 제목` 형태의 한 줄로 바꿔 출력합니다. |

### 9-2. 중복 없이 이어서 만들도록 함수 수정

```bash
EXISTING=$(gh issue list --repo "$REPO" --state all --limit 200 --json title --jq '.[].title')
iss() {
  if grep -qxF "$1" <<<"$EXISTING"; then echo "skip: $1"; return; fi
  for i in 1 2 3; do
    gh issue create --repo "$REPO" --title "$1" --milestone "$2" --label "$3" --body "$4" && return
    echo "retry $i: $1"; sleep 3
  done
  return 1
}
```

| 줄 | 설명 |
|---|---|
| `EXISTING=$(...)` | 이미 있는 이슈 제목을 모두 가져와 변수에 저장합니다. `$( )`는 명령 결과를 값으로 받는 문법입니다. |
| `--limit 200` | 최대 200개까지 가져옵니다. 기본값은 30개입니다. |
| `--jq '.[].title'` | 제목만 한 줄에 하나씩 출력합니다. |
| `grep -qxF "$1" <<<"$EXISTING"` | 만들려는 제목이 기존 목록에 있는지 검사합니다. `-q`는 출력 없이 결과만 돌려주고, `-x`는 한 줄 전체가 일치해야 하며, `-F`는 괄호 같은 특수문자를 글자 그대로 비교합니다. `<<<`는 변수 내용을 입력으로 넘깁니다. |
| `then echo "skip: $1"; return` | 이미 있으면 `skip`을 출력하고 만들지 않습니다. 이렇게 해서 중복 생성을 막습니다. |
| `for i in 1 2 3; do ... done` | 이슈 생성을 최대 3번까지 시도합니다. |
| `gh issue create ... && return` | 생성에 성공하면(`&&`) 바로 함수를 끝냅니다. |
| `echo "retry $i: $1"; sleep 3` | 실패하면 재시도 메시지를 출력하고 3초 기다립니다. |
| `return 1` | 3번 모두 실패하면 실패 코드를 돌려줘 스크립트를 멈춥니다. |

마일스톤 생성 부분은 다시 실행하면 같은 이름 때문에 에러가 나므로 조건문으로 감쌌습니다.

```bash
if [ -z "${SKIP_SETUP:-}" ]; then
  ms "$M1" ...
  ms "$M2" ...
fi
```

| 줄 | 설명 |
|---|---|
| `${SKIP_SETUP:-}` | `SKIP_SETUP` 변수 값을 읽습니다. 없으면 빈 문자열로 처리해 `set -u` 에러를 피합니다. |
| `[ -z ... ]` | 값이 비어 있으면 참입니다. 즉 `SKIP_SETUP`을 지정하지 않았을 때만 마일스톤을 만듭니다. |

### 9-3. 이어서 실행

```bash
SKIP_SETUP=1 bash setup_issues.sh
```

| 부분 | 설명 |
|---|---|
| `SKIP_SETUP=1` | 이 실행에만 환경 변수를 넘겨 마일스톤 생성을 건너뜁니다. |
| `bash setup_issues.sh` | 스크립트를 다시 실행합니다. 레이블은 `--force` 덕분에 덮어써지고, 이미 만든 이슈 #2~#5는 `skip`되며, #6~#11만 새로 만들어집니다. |

---

## 10. 최종 결과 확인

```bash
gh issue list --state all --json number,title,milestone,labels \
  --jq '.[] | "#\(.number) [\(.milestone.title // "-")] \(.title) | \([.labels[].name]|join(","))"'
gh api repos/HyeokjaeMoon/AI-Issue/milestones --jq '.[] | "\(.title) due \(.due_on) open=\(.open_issues)"'
gh label list --limit 50 | wc -l
```

| 줄 / 부분 | 설명 |
|---|---|
| `--json number,title,milestone,labels` | 이슈 번호, 제목, 마일스톤, 레이블을 함께 가져옵니다. |
| `\(.milestone.title // "-")` | 마일스톤 제목을 출력합니다. `//`는 값이 없을 때 대신 쓸 값이며, 마일스톤이 없는 #1은 `-`로 나옵니다. |
| `\([.labels[].name]\|join(","))` | 레이블 이름만 모아 쉼표로 이어 붙입니다. |
| `gh api .../milestones --jq ...` | 마일스톤마다 제목, 마감일, 열린 이슈 수를 한 줄로 출력합니다. 결과는 Week 1 `open=6`, Week 2 `open=4`였습니다. |
| `gh label list --limit 50 \| wc -l` | 레이블 목록의 줄 수를 셉니다. 기본 레이블 10개와 추가 레이블 10개를 합쳐 20개였습니다. |

---

## 자주 쓰는 명령 요약

| 하고 싶은 일 | 명령어 |
|---|---|
| 로그인 상태 확인 | `gh auth status` |
| 공개 레포 만들고 푸시 | `gh repo create <이름> --public --source . --push` |
| 레포 정보 보기 | `gh repo view <계정>/<레포>` |
| 이슈 만들기 | `gh issue create --title "..." --body "..." --label "a,b" --milestone "..."` |
| 이슈 목록 보기 | `gh issue list --state all` |
| 이슈 하나 보기 | `gh issue view <번호>` |
| 레이블 만들기/수정 | `gh label create <이름> --color <색> --description "..." --force` |
| 레이블 목록 보기 | `gh label list` |
| 마일스톤 만들기 | `gh api repos/<계정>/<레포>/milestones -f title="..." -f due_on="YYYY-MM-DDT23:59:59Z"` |
| 마일스톤 목록 보기 | `gh api repos/<계정>/<레포>/milestones` |
