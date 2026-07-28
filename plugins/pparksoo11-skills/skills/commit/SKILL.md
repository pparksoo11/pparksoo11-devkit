---
name: commit
description: "변경 내용을 분석하여 적절한 Conventional Commit 메시지를 생성하고 커밋합니다."
model: haiku
argument-hint: "[선택사항: 커밋 범위 또는 메시지 힌트]"
---

다음 절차를 순서대로 수행하세요.

## 1. 변경 내용 파악

```bash
git status
git diff --staged
git diff
```

staged 파일이 없으면 변경된 파일 전체를 staging합니다:
```bash
git add -A
```

단, 다음 파일은 절대 staging하지 마세요:
- `.env`, `*.key`, `*secret*`, `*credentials*` 등 민감 정보 파일
- `local.properties`, `*.local.*` 등 로컬 전용 파일

## 2. 커밋 타입 선택

변경 내용을 보고 가장 적합한 타입을 하나 선택하세요:

| 타입 | 사용 조건 |
|------|-----------|
| `feat` | 새로운 기능 추가 |
| `fix` | 버그 수정 |
| `refactor` | 동작 변경 없는 코드 구조 개선 |
| `chore` | 빌드 설정, 의존성, 설정 파일 변경 |
| `docs` | 문서만 변경 |
| `style` | 포맷, 공백 등 코드 의미 없는 변경 |
| `test` | 테스트 추가 또는 수정 |
| `perf` | 성능 개선 |
| `ci` | CI/CD 파이프라인 변경 |

## 3. 커밋 메시지 작성 규칙

- **제목**: `<타입>: <변경 내용>` — 50자 이내, 마침표 없음, 명령형
- **본문**: 변경 이유나 맥락이 필요할 때만 추가. 불릿(`-`) 형태로 간결하게
- **언어**: 제목과 본문 모두 한국어로 작성 (단, 코드/파일명은 영어 유지)

좋은 예:
```
feat: 경로 캐시 정밀도 선택 UI 추가

- 소수점 4~7자리 선택 가능
- 선택값은 SharedPreferences에 저장
```

나쁜 예: `여러 파일 수정함`, `기능 개선`, `업데이트`

## 4. 커밋 실행

Attribution 줄 없이 커밋하세요:

```bash
git commit -m "$(cat <<'EOF'
<타입>: <제목>

- 설명1 (필요시)
- 설명2 (필요시)
EOF
)"
```

`Co-Authored-By:` 또는 `🤖 Generated with` 줄을 절대 포함하지 마세요.

## 5. 결과 확인

```bash
git log --oneline -1
```

커밋된 해시와 메시지를 간단히 출력하고 완료를 알립니다.
