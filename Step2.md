# Step 2. docs 문서를 GitHub 이슈로 올리기

[Step 1](Step1.md)에서 만든 레포에 `docs/` 폴더의 기록 문서 17개를 이슈로 올리고, 저장소에 커밋·푸시한 과정입니다.
각 명령어 아래에 **한 줄씩** 무슨 역할을 하는지 설명했습니다.

- 사용 도구: `git`, `gh` (GitHub CLI), `bash`, `sed`
- 대상 레포: https://github.com/HyeokjaeMoon/AI-Issue
- 결과: 이슈 #12 ~ #28 (TIL 14개, 회고 2개, 부검 1개)

---

## 0. 올릴 문서 구조

```
docs/
├── README.md                              # 목차
├── til/2026-11-09.md ~ 2026-11-22.md      # TIL 14개
├── retrospective/sprint-1.md, sprint-2.md # 회고 2개
└── postmortem/2026-11-23-ttest-variance.md # 부검 1개
```

| 규칙 | 설명 |
|---|---|
| 첫 줄 `# 제목` | 이슈 **제목**으로 씁니다. |
| 둘째 줄부터 | 이슈 **본문**으로 씁니다. 제목이 본문에 두 번 나오지 않게 첫 줄은 뺍니다. |

> 처음에는 `bash` 한 번에 heredoc(`cat > 파일 <<'EOF'`)을 18개 이어 붙여 문서를 만들려고 했습니다. 하지만 셸이 따옴표 짝을 잘못 해석해 `unexpected EOF while looking for matching` 오류로 실패했습니다. 그래서 문서는 파일 하나씩 따로 작성했습니다.

---

## 1. 문서 종류 레이블 만들기

```bash
REPO="HyeokjaeMoon/AI-Issue"
gh label create "til"           --repo "$REPO" --color "0052cc" --description "Today I Learned 기록" --force
gh label create "retrospective" --repo "$REPO" --color "5319e7" --description "스프린트 회고" --force
gh label create "postmortem"    --repo "$REPO" --color "b60205" --description "장애·오류 부검" --force
```

| 줄 / 부분 | 설명 |
|---|---|
| `REPO="..."` | 대상 레포를 변수에 저장해 둡니다. |
| `gh label create "til"` | `til` 레이블을 만듭니다. 이슈 목록에서 TIL만 골라 볼 때 씁니다. |
| `"retrospective"` | 회고 이슈용 레이블입니다. |
| `"postmortem"` | 부검 이슈용 레이블입니다. |
| `--color` | 레이블 색상입니다. `#` 없이 6자리 16진수로 씁니다. |
| `--description` | 레이블에 마우스를 올리면 보이는 설명입니다. |
| `--force` | 같은 이름이 이미 있으면 덮어씁니다. 스크립트를 다시 실행해도 에러가 나지 않습니다. |

---

## 2. 기본 변수 준비

```bash
set -euo pipefail
BLOB="https://github.com/$REPO/blob/main/docs"
M1="Week 1 - 데이터 처리 & 기술통계"
M2="Week 2 - 추론통계 & 시각화"
TMP="$(dirname "$0")/issue_body.md"
cd /c/Users/LT-011/Files/Workspace/AI-Issue/docs
EXISTING=$(gh issue list --repo "$REPO" --state all --limit 200 --json title --jq '.[].title')
```

| 줄 | 설명 |
|---|---|
| `set -euo pipefail` | 명령이 하나라도 실패하면 스크립트를 멈춥니다. 정의되지 않은 변수를 쓰면 에러를 냅니다. 파이프 중간에서 실패해도 실패로 처리합니다. |
| `BLOB="..."` | GitHub에서 `docs` 폴더 파일을 여는 주소의 앞부분입니다. 상대 링크를 절대 링크로 바꿀 때 씁니다. |
| `M1=`, `M2=` | Step 1에서 만든 마일스톤 제목입니다. 이슈에 마일스톤을 지정하려면 **제목이 정확히 같아야** 합니다. |
| `TMP=...` | 이슈 본문을 잠시 저장할 임시 파일 경로입니다. `$0`은 스크립트 자신의 경로이고, `dirname`은 그 파일이 있는 폴더를 꺼냅니다. |
| `cd .../docs` | `docs` 폴더로 이동합니다. 이후 파일 경로를 `til/2026-11-09.md`처럼 짧게 쓸 수 있습니다. |
| `EXISTING=$(...)` | 레포에 이미 있는 이슈 제목을 모두 가져옵니다. 같은 이슈를 두 번 만들지 않기 위한 목록입니다. |

---

## 3. 문서 하나를 이슈로 만드는 함수

```bash
make_issue() {
  local file="$1" label="$2" milestone="$3"
  local dir; dir="$(dirname "$file")"
  local title; title="$(head -n 1 "$file" | sed 's/^# //')"

  if grep -qxF "$title" <<<"$EXISTING"; then echo "skip: $title"; return; fi

  {
    tail -n +2 "$file" | sed -e "s#](\.\./#]($BLOB/#g" -e "s#](\([a-z0-9-]*\.md\))#]($BLOB/$dir/\1)#g"
    printf '\n---\n원본 문서: [%s](%s/%s)\n' "docs/$file" "$BLOB" "$file"
  } > "$TMP"

  local args=(--repo "$REPO" --title "$title" --label "$label" --body-file "$TMP")
  [ -n "$milestone" ] && args+=(--milestone "$milestone")

  for i in 1 2 3; do
    gh issue create "${args[@]}" && return
    echo "retry $i: $title"; sleep 3
  done
  return 1
}
```

### 3-1. 입력값과 제목 만들기

| 줄 | 설명 |
|---|---|
| `make_issue() { ... }` | 문서 파일 하나를 이슈 하나로 바꾸는 함수입니다. 아래에서 17번 호출합니다. |
| `local file="$1" label="$2" milestone="$3"` | 함수에 넘긴 값을 이름 있는 변수로 받습니다. `local`은 함수 안에서만 쓰는 변수라는 뜻입니다. |
| `dir="$(dirname "$file")"` | 파일이 있는 폴더 이름을 꺼냅니다. 예: `til/2026-11-09.md` → `til` |
| `head -n 1 "$file"` | 파일의 첫 줄만 읽습니다. 예: `# TIL 2026-11-09 (월) · ...` |
| `\| sed 's/^# //'` | 줄 맨 앞의 `# `를 지워 제목만 남깁니다. `^`는 "줄의 시작"을 뜻합니다. |

### 3-2. 중복 방지

| 부분 | 설명 |
|---|---|
| `grep -qxF "$title" <<<"$EXISTING"` | 같은 제목의 이슈가 이미 있는지 검사합니다. `-q`는 출력 없이 결과만 돌려줍니다. `-x`는 한 줄 전체가 일치해야 맞는 것으로 봅니다. `-F`는 괄호나 `·` 같은 특수문자를 글자 그대로 비교합니다. |
| `echo "skip: ..."; return` | 이미 있으면 건너뜁니다. 그래서 스크립트를 여러 번 실행해도 이슈가 중복되지 않습니다. |

### 3-3. 이슈 본문 만들기

| 부분 | 설명 |
|---|---|
| `{ ... } > "$TMP"` | 중괄호 안 명령들의 출력을 합쳐 임시 파일 하나에 저장합니다. |
| `tail -n +2 "$file"` | 2번째 줄부터 끝까지 출력합니다. 제목 줄을 본문에서 빼는 역할입니다. |
| `sed -e "s#](\.\./#]($BLOB/#g"` | `](../postmortem/...)`처럼 **상위 폴더를 가리키는 상대 링크**를 `](https://github.com/.../docs/postmortem/...)`로 바꿉니다. 이슈 안에서는 상대 경로 링크가 열리지 않기 때문입니다. `#`은 `s/찾을것/바꿀것/` 대신 쓴 구분자입니다. 주소에 `/`가 많아서 `#`으로 바꿨습니다. |
| `-e "s#](\([a-z0-9-]*\.md\))#]($BLOB/$dir/\1)#g"` | `](sprint-1.md)`처럼 **같은 폴더 안 파일 링크**를 절대 링크로 바꿉니다. `\( \)`로 묶은 파일 이름은 `\1` 자리에 다시 들어갑니다. |
| `printf '\n---\n원본 문서: ...'` | 본문 끝에 구분선과 원본 `.md` 파일 링크를 덧붙입니다. |

### 3-4. 이슈 생성 옵션 조립과 재시도

| 부분 | 설명 |
|---|---|
| `local args=(...)` | `gh issue create`에 넘길 옵션을 **배열**로 모읍니다. 배열에 담으면 띄어쓰기가 있는 값(마일스톤 제목 등)도 안전하게 전달됩니다. |
| `--title "$title"` | 이슈 제목입니다. |
| `--label "$label"` | 붙일 레이블입니다. `postmortem,bug`처럼 쉼표로 여러 개를 줄 수 있습니다. |
| `--body-file "$TMP"` | 본문을 문자열 대신 **파일에서 읽습니다**. 코드 블록, 표, 백틱이 많은 긴 마크다운도 셸 이스케이프 없이 그대로 올라갑니다. |
| `[ -n "$milestone" ] && args+=(--milestone "$milestone")` | 마일스톤 값이 비어 있지 않을 때만 `--milestone` 옵션을 추가합니다. 부검 이슈는 마일스톤 없이 만듭니다. |
| `for i in 1 2 3; do ... done` | 네트워크 오류에 대비해 최대 3번까지 시도합니다. Step 1에서 타임아웃을 겪고 넣은 장치입니다. |
| `gh issue create "${args[@]}" && return` | 배열의 옵션을 모두 펼쳐 이슈를 만들고, 성공하면 함수를 끝냅니다. |
| `echo "retry ..."; sleep 3` | 실패하면 메시지를 출력하고 3초 기다린 뒤 다시 시도합니다. |
| `return 1` | 3번 모두 실패하면 실패 코드를 돌려줍니다. `set -e` 때문에 스크립트도 멈춥니다. |

---

## 4. 문서 종류별로 함수 호출하기

```bash
for f in til/*.md; do
  day="$(basename "$f" .md)"
  if [[ "$day" < "2026-11-16" ]]; then ms="$M1"; else ms="$M2"; fi
  make_issue "$f" "til" "$ms"
done

make_issue "retrospective/sprint-1.md" "retrospective" "$M1"
make_issue "retrospective/sprint-2.md" "retrospective" "$M2"

make_issue "postmortem/2026-11-23-ttest-variance.md" "postmortem,bug" ""

rm -f "$TMP"
```

| 줄 | 설명 |
|---|---|
| `for f in til/*.md; do` | `til` 폴더의 `.md` 파일을 이름순(=날짜순)으로 하나씩 꺼냅니다. |
| `day="$(basename "$f" .md)"` | 경로와 확장자를 떼고 날짜만 남깁니다. 예: `til/2026-11-09.md` → `2026-11-09` |
| `if [[ "$day" < "2026-11-16" ]]` | 날짜를 문자열로 비교합니다. `YYYY-MM-DD` 형식은 글자 순서가 곧 날짜 순서라서 이렇게 비교할 수 있습니다. |
| `then ms="$M1"; else ms="$M2"; fi` | 11/16 이전은 1주차, 이후는 2주차 마일스톤으로 정합니다. 주말(11/14~15) TIL은 1주차 이슈를 마무리한 날이라 1주차에 넣었습니다. |
| `make_issue "$f" "til" "$ms"` | TIL 문서를 `til` 레이블과 해당 주 마일스톤을 붙여 이슈로 만듭니다. |
| `make_issue "retrospective/sprint-1.md" ...` | 스프린트 1 회고를 1주차 마일스톤에 넣습니다. |
| `make_issue "retrospective/sprint-2.md" ...` | 스프린트 2 회고를 2주차 마일스톤에 넣습니다. |
| `make_issue "postmortem/..." "postmortem,bug" ""` | 부검은 `postmortem`과 `bug` 레이블을 붙입니다. 릴리스 이후에 일어난 일이라 마일스톤은 비워 둡니다(`""`). |
| `rm -f "$TMP"` | 임시 본문 파일을 지웁니다. `-f`는 파일이 없어도 에러를 내지 않습니다. |

### 스크립트 실행

```bash
bash docs_to_issues.sh
```

| 부분 | 설명 |
|---|---|
| `bash docs_to_issues.sh` | 1~4단계를 담은 스크립트를 실행합니다. 이슈 주소가 17줄 출력되고 마지막에 `DONE`이 나오면 성공입니다. |

---

## 5. 결과 확인

```bash
gh issue list --state all --limit 50 --json number,title,milestone,labels \
  --jq '.[] | select(.number>=12) | "#\(.number) [\(.milestone.title // "-")] \(.title) | \([.labels[].name]|join(","))"'
gh issue view 25 --json body --jq .body | grep -n "github.com"
gh issue view 28 --json body --jq .body | head -5
```

| 줄 / 부분 | 설명 |
|---|---|
| `--limit 50` | 이슈가 30개를 넘었으므로 기본값(30)보다 크게 잡습니다. |
| `select(.number>=12)` | 이번에 만든 #12 이후 이슈만 골라 출력합니다. |
| `\(.milestone.title // "-")` | 마일스톤 제목을 출력하고, 없으면 `-`를 출력합니다. |
| `\([.labels[].name]\|join(","))` | 레이블 이름을 쉼표로 이어 붙입니다. |
| `gh issue view 25 --json body --jq .body` | #25 이슈(11/22 TIL)의 본문만 가져옵니다. |
| `\| grep -n "github.com"` | 본문에서 링크가 들어간 줄만 찾습니다. `../postmortem/...` 링크가 절대 주소로 바뀌었는지 확인하는 용도입니다. |
| `gh issue view 28 ... \| head -5` | 부검 이슈 본문의 앞 5줄만 봅니다. 제목 줄이 빠지고 본문부터 시작하는지 확인합니다. |

**만들어진 이슈**

| 번호 | 종류 | 마일스톤 | 레이블 |
|---|---|---|---|
| #12 ~ #18 | TIL 11/09 ~ 11/15 | Week 1 | `til` |
| #19 ~ #25 | TIL 11/16 ~ 11/22 | Week 2 | `til` |
| #26 | 스프린트 1 회고 | Week 1 | `retrospective` |
| #27 | 스프린트 2 회고 | Week 2 | `retrospective` |
| #28 | 부검: v0.1.0 t-검정 p-value 오류 | 없음 | `postmortem`, `bug` |

TIL 본문에 적힌 `#2`, `#7` 같은 번호는 GitHub가 자동으로 해당 이슈 링크로 바꿔 줍니다. 그래서 작업 이슈와 TIL이 서로 연결됩니다.

---

## 6. 커밋하고 푸시하기

```bash
git add docs Step2.md
git commit -F - <<'EOF'
Add TIL, retrospectives, postmortem docs and Step2.md
EOF
git push
git status -sb
```

| 줄 | 설명 |
|---|---|
| `git add docs Step2.md` | `docs` 폴더 전체와 이 문서를 스테이징합니다. 폴더 이름을 주면 안의 파일이 모두 포함됩니다. |
| `git commit -F -` | 커밋 메시지를 파일 대신 표준 입력(`-`)에서 읽습니다. 바로 아래 heredoc으로 여러 줄 메시지를 넘길 수 있습니다. |
| `<<'EOF' ... EOF` | heredoc입니다. `EOF`가 나올 때까지의 내용이 입력으로 전달됩니다. 따옴표로 감싼 `'EOF'`는 `$` 같은 특수문자를 해석하지 않고 그대로 넘깁니다. |
| `git push` | 로컬 커밋을 `origin/main`에 올립니다. 이슈 본문의 "원본 문서" 링크는 이때부터 열립니다. |
| `git status -sb` | 브랜치 상태를 한 줄로 확인합니다. `## main...origin/main` 뒤에 `[ahead N]`이 없으면 원격과 같은 상태입니다. |

---

## 자주 쓰는 명령 요약 (Step 2에서 새로 쓴 것)

| 하고 싶은 일 | 명령어 |
|---|---|
| 파일 내용을 본문으로 이슈 만들기 | `gh issue create --title "..." --body-file 파일.md` |
| 레이블 여러 개 붙이기 | `--label "postmortem,bug"` |
| 특정 레이블 이슈만 보기 | `gh issue list --label til` |
| 특정 마일스톤 이슈만 보기 | `gh issue list --milestone "Week 1 - 데이터 처리 & 기술통계"` |
| 이슈 본문만 출력 | `gh issue view <번호> --json body --jq .body` |
| 파일 첫 줄 / 둘째 줄부터 | `head -n 1 파일` / `tail -n +2 파일` |
| 문자열 치환 | `sed 's#찾을것#바꿀것#g'` |
