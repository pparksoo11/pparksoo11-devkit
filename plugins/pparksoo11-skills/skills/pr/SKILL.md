---
name: pr
description: "현재 브랜치의 변경사항을 분석하여 GitHub Pull Request를 생성합니다. 최초 실행 시 저장소별 설정을 질문으로 수집합니다."
argument-hint: "[선택사항: PR 제목]"
model: haiku
---

다음 단계를 순서대로 수행하세요.

## Phase 0: 설정 확인

### 0-1. gh CLI 인증 확인

```bash
gh auth status
```

인증되어 있지 않으면 **중단**하고 안내:
> "GitHub CLI 인증이 필요합니다. `! gh auth login`을 실행한 뒤 다시 시도하세요."

### 0-2. 저장소 설정 로드 (최초 실행 시 설정 수집)

`.claude/pr-config.json` 파일을 읽습니다.

**파일이 있으면** 그 값을 사용하고 Phase 1로 진행합니다.

**파일이 없으면 최초 설정을 진행합니다:**

1. 저장소 기본 브랜치를 감지합니다:
   ```bash
   gh repo view --json defaultBranchRef -q .defaultBranchRef.name
   ```
2. AskUserQuestion 도구로 다음 항목을 질문합니다 (도구를 쓸 수 없으면 텍스트로 질문):
   - **베이스 브랜치**: PR 대상 브랜치 (기본값: 위에서 감지한 브랜치)
   - **리뷰어**: GitHub 사용자명, 복수면 쉼표 구분 (건너뛰기 가능)
   - **Draft 여부**: PR을 기본으로 draft로 생성할지 (기본: 아니오)
   - **기본 라벨**: 항상 적용할 라벨 (건너뛰기 가능)
3. 답변을 `.claude/pr-config.json`에 저장합니다:
   ```json
   {
     "baseBranch": "main",
     "reviewers": [],
     "draft": false,
     "labels": []
   }
   ```
4. 저장된 설정을 요약해 보여주고 계속 진행합니다.

> 설정을 바꾸려면 `.claude/pr-config.json`을 직접 수정하거나 삭제 후 재실행하면 됩니다.

---

## Phase 1: Validation

### 1-1. 현재 브랜치 확인

```bash
git branch --show-current
```

현재 브랜치가 베이스 브랜치와 같으면 **중단**:
> "베이스 브랜치(`<baseBranch>`)에서는 PR을 생성할 수 없습니다. 작업 브랜치를 먼저 만드세요."

### 1-2. 미커밋 변경사항 확인

```bash
git status --porcelain
```

미커밋 변경사항이 있으면 **중단**:
> "커밋되지 않은 변경사항이 있습니다. 먼저 커밋하거나 stash하세요."

### 1-3. 기존 PR 확인

```bash
gh pr list --head <current-branch> --state open
```

이미 열린 PR이 있으면 해당 PR 정보를 표시하고 계속 진행 여부를 묻습니다.

---

## Phase 2: Diff 분석

### 2-1. 최신 상태 동기화

```bash
git fetch origin <baseBranch> 2>/dev/null || true
```

### 2-2. 커밋 목록 확인

```bash
git log --oneline origin/<baseBranch>..HEAD
```

커밋이 없으면 **중단**:
> "베이스 브랜치(`<baseBranch>`) 대비 변경사항이 없습니다."

### 2-3. 변경 파일 요약

```bash
git diff origin/<baseBranch>...HEAD --stat
```

### 2-4. 전체 Diff 확인

```bash
git diff origin/<baseBranch>...HEAD
```

변경 규모가 크면 주요 파일을 Read 도구로 추가 확인하세요.

---

## Phase 3: Title & Body 생성

### 3-1. 제목 결정

`$ARGUMENTS`가 제공된 경우:
- 제목: `<$ARGUMENTS>`

`$ARGUMENTS`가 없는 경우:
- diff와 커밋 메시지를 분석하여 변경사항을 가장 잘 표현하는 한국어 제목 생성
- 제목은 간결하게 (50자 이내), 마침표 없음

### 3-2. Body 생성

다음 템플릿을 기반으로 작성하되, **항목이 없는 섹션은 완전히 생략**합니다.

```
## 개요

<diff 분석 기반 1-3문장 요약 (한국어)>

---

## 주요 변경사항

<신규/변경/수정 중 해당하는 것만 작성>

### 신규
- <새로 추가된 기능, 파일, 클래스>

### 변경
- <수정된 기존 코드, 리팩토링, 동작 변경>

### 수정
- <버그 수정 사항>

---

## 주의사항

- <빌드 설정, 의존성, 권한, 동작 방식 변경 등 리뷰어가 주의할 사항>
- 특별한 주의사항이 없으면 이 섹션 생략

---

## 테스트 체크리스트

- [ ] <변경된 기능에 대한 구체적 테스트 항목>
- [ ] <엣지케이스 확인>
```

**작성 규칙:**
- `신규` / `변경` / `수정` 중 해당 항목이 있는 섹션만 작성 (없으면 해당 `###` 헤더도 생략)
- `주의사항`에 특별한 내용이 없으면 섹션 전체 생략
- 테스트 체크리스트는 실제 변경사항 기반으로 구체적으로 작성
- 브랜치명 또는 커밋 메시지에서 이슈 번호(`#123` 또는 `123-` 접두 브랜치)가 확인되면 body 맨 끝에 `Closes #<번호>` 한 줄 추가
- `Co-Authored-By:`, `🤖 Generated with` 등 AI attribution 줄은 절대 포함하지 않음
- 전체 한국어로 작성

---

## Phase 4: Push & PR 생성

### 4-1. 브랜치 Push

upstream 설정 확인:
```bash
git rev-parse --abbrev-ref @{u} 2>/dev/null
```

upstream이 없거나 push가 필요하면:
```bash
git push -u origin <current-branch>
```

push 실패 시 **중단**하고 오류 메시지 표시.

### 4-2. 사용자 확인

생성할 제목, body, 적용될 옵션(베이스 브랜치, 리뷰어, draft, 라벨)을 출력하고 진행 여부를 확인합니다.

### 4-3. PR 생성

```bash
gh pr create \
  --title "<title>" \
  --body "$(cat <<'PRDESC'
<generated body>
PRDESC
)" \
  --base "<baseBranch>" \
  --assignee "@me"
```

설정값에 따라 다음 옵션을 추가합니다:
- `reviewers`가 있으면: `--reviewer "<user1>,<user2>"`
- `draft`가 true면: `--draft`
- `labels`가 있으면: `--label "<label>"`

생성 실패 시 오류 메시지를 그대로 출력합니다.

---

## Phase 5: 결과 확인

PR 생성 완료 후 다음을 출력합니다:

- PR URL
- PR 번호
- Source → Base 브랜치
- 변경 파일 수
