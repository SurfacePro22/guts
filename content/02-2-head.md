# HEAD란? (HEAD, 브랜치, 커밋의 관계)

## 한 줄 정의

- **HEAD**: "지금 내가 어디에 있는지"를 가리키는 포인터. 대부분 브랜치를 가리키고, 그 브랜치가 다시 커밋을 가리킨다.
- 실체는 `.git/HEAD` 파일 하나. 보통은 `ref: refs/heads/main`처럼 브랜치 이름을 담고 있다 (커밋 해시를 직접 담지 않는다).
- **Detached HEAD**: 브랜치가 아니라 커밋 해시를 직접 가리키는 상태. `git checkout <커밋해시>`처럼 브랜치명이 아닌 커밋으로 이동하면 발생.

## 표로 비교

| | 평소 (브랜치에 있음) | Detached HEAD |
|---|---|---|
| HEAD가 가리키는 것 | 브랜치 (예: `main`) | 커밋 해시 직접 |
| 가리키는 경로 | HEAD → 브랜치 → 커밋 (간접) | HEAD → 커밋 (직접) |
| 새 커밋 시 | 브랜치가 함께 앞으로 이동 | 어느 브랜치에도 속하지 않는 커밋 생성 (브랜치 안 만들면 나중에 유실 위험) |
| `.git/HEAD` 내용 | `ref: refs/heads/main` | 커밋 해시 자체 |
| 발생 상황 | `git checkout main` | `git checkout <해시>`, `git checkout <태그>`, rebase 도중 |
| 빠져나오기 | - | `git checkout <브랜치명>` (필요하면 먼저 `git branch <새이름>`으로 커밋 구제) |

## 참고: HEAD 표기법

- `HEAD~n`: HEAD로부터 첫 번째 부모만 따라 n단계 이전 커밋 (머지 커밋이어도 항상 첫 부모)
- `HEAD^n`: HEAD의 n번째 부모 (머지 커밋처럼 부모가 여러 개일 때 몇 번째 부모인지 선택)

## 명령어 치트시트 (2장 — 저장소의 구조)

| 명령 | 설명 |
|---|---|
| `git add <파일>` | Workspace → Index로 스테이징 |
| `git commit` | Index → Local Repo로 커밋 |
| `git push` | Local Repo → Upstream으로 전송 |
| `git checkout <경로>` / `git restore <경로>` | Workspace 변경사항 되돌리기 |
| `git reset` | Index 되돌리기 (unstage) |
| `git stash` / `git stash pop`/`apply` | 변경사항 임시 저장 / 꺼내기 |
| `git symbolic-ref HEAD refs/heads/<브랜치>` | HEAD가 가리키는 기본 브랜치 변경 |
| `HEAD~n` / `HEAD^n` | n단계 이전(첫 부모만) / n번째 부모 |


## 한 줄 요약

HEAD는 "현재 위치" 포인터이며 보통은 브랜치를 통해 간접적으로 커밋을 가리키지만, 커밋을 직접 가리키는 Detached HEAD 상태에서는 새 커밋이 브랜치에 속하지 않아 유실될 수 있으니 주의해야 한다.
