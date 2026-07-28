# pparksoo11-devkit

pparksoo11의 개인 Claude Code 설정 모음. 다른 PC에서도 단 몇 줄로 설치할 수 있습니다.

## 포함 플러그인

| 플러그인 | 내용 |
|---|---|
| `pparksoo11-skills` | `clean-code` (Uncle Bob Clean Code), `commit` (한국어 커밋 생성), `pr` (GitHub PR 생성) 스킬 |
| `pparksoo11-agents` | android-developer, flutter-developer, code-reviewer-kr, git-commander, planner, terminal-helper |
| `pparksoo11-workflow` | `workflow-guide` — Plan-mode-first 워크플로우 지침 |

## 설치

```
/plugin marketplace add pparksoo11/pparksoo11-devkit
```

### 전체 설치
```
/plugin install pparksoo11-skills@pparksoo11-devkit
/plugin install pparksoo11-agents@pparksoo11-devkit
/plugin install pparksoo11-workflow@pparksoo11-devkit
```

### 선택 설치 (예: 스킬만)
```
/plugin install pparksoo11-skills@pparksoo11-devkit
```

## 업데이트

```
/plugin marketplace update pparksoo11-devkit
```

## 스킬 사용법

플러그인 스킬은 `/플러그인명:스킬명` 형태로 호출합니다 (동명의 개인 스킬과 충돌 방지):

| 명령 | 설명 |
|---|---|
| `/pparksoo11-skills:clean-code` | 현재 코드를 Clean Code 원칙으로 리뷰/리팩터링 |
| `/pparksoo11-skills:commit` | git 변경 내용 분석 후 한국어 커밋 메시지 자동 생성 |
| `/pparksoo11-skills:pr` | 브랜치 변경사항 분석 후 GitHub PR 생성 (최초 실행 시 저장소 설정 질문) |
| `/pparksoo11-workflow:workflow-guide` | Plan-mode-first 워크플로우 지침 조회 |

## 에이전트 사용법

`Agent` 도구의 `subagent_type` 파라미터로 호출합니다:

```
android-developer   — Android/Kotlin 기능 구현 및 리팩터링
flutter-developer   — Flutter 시니어 구현 및 성능 최적화
code-reviewer-kr    — 다차원 코드 리뷰 (중요도별 분류)
git-commander       — git 커밋/브랜치/PR 워크플로우
planner             — 요구사항 정제 및 구현 플랜 수립
terminal-helper     — 터미널 명령 실행 및 빌드 프로세스
```

## 라이선스

MIT © 2026 pparksoo11
