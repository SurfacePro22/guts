# 전체 명령어 치트시트

강의 전체에서 다룬 명령어를 장 순서대로 모았다. 각 항목의 자세한 설명은 해당 장의 md 파일을 참고.

## 2장 — 저장소의 구조

| 명령 | 설명 |
|---|---|
| `git add <파일>` | Workspace → Index로 스테이징 |
| `git commit` | Index → Local Repo로 커밋 |
| `git push` | Local Repo → Upstream으로 전송 |
| `git checkout <경로>` / `git restore <경로>` | Workspace 변경사항 되돌리기 |
| `git reset` | Index 되돌리기 (unstage) |
| `git stash` / `git stash pop`·`apply` | 변경사항 임시 저장 / 꺼내기 |
| `git symbolic-ref HEAD refs/heads/<브랜치>` | HEAD가 가리키는 기본 브랜치 변경 |
| `HEAD~n` / `HEAD^n` | n단계 이전(첫 부모만) / n번째 부모 |

## 3장 — 원격 저장소

| 명령 | 설명 |
|---|---|
| `git clone <url>` | 원격 저장소 복제 (HTTPS/SSH/File/Git 프로토콜) |
| `git init --bare` | Bare Repository 생성 |
| `git config --system`·`--global`·`--local` | 설정 범위별 지정 (Local이 우선) |
| `git config --list --show-origin` | 각 설정값이 어느 파일에서 왔는지 확인 |
| `git remote add origin <url>` | 원격 추가 (`.git/config`에 기록됨) |
| `git fetch origin` | 원격 정보만 받아오기 (`refs/remotes/origin/*` 갱신) |
| `git pull` (`= fetch + merge/rebase`) | 받아오고 로컬 브랜치까지 반영 |
| `git log origin/main` / `git diff origin/main` | 로컬에 저장된 원격 추적 브랜치 기준 비교 |
| `git push origin main` | 원격의 실제 `main` 브랜치로 전송 |

## 4장 — 변경사항 이식하기

| 명령 | 설명 |
|---|---|
| `git diff` | Workspace의 변경사항 확인 |
| `git diff A..B` | A와 B를 있는 그대로 비교 |
| `git diff A...B` | 공통 조상과 B를 비교 (B에서만 일어난 변경) |
| `git log A..B` / `git log A...B` | 한쪽 방향 커밋 / 양쪽 대칭차집합 |
| `git diff > x.patch` / `git apply x.patch` | 순수 diff 패치 생성 / 적용 |
| `git format-patch -1 HEAD` / `git am x.patch` | 커밋 메타데이터 포함 패치 생성 / 커밋으로 재현 |
| `git apply --check`·`--reject`·`--3way` | 적용 전 검증 / 부분 적용 / 3-way 병합 시도 |
| `git cherry-pick <해시>` | 다른 브랜치의 커밋 하나 복사해오기 |
| `git cherry-pick -n` / `-x` / `A..B` | 커밋 안 함 / 원본 해시 흔적 남김 / 범위 지정 |
| `git cherry-pick --continue`·`--abort`·`--skip` | 충돌 시 계속 / 취소 / 건너뛰기 |

## 5장 — Rebase

| 명령 | 설명 |
|---|---|
| `git rebase <브랜치>` | 내 커밋들을 다른 브랜치 위로 재적용 |
| `git rebase -i <base>` | 커밋 목록을 열어 pick/reword/edit/squash/fixup/drop 지정 |
| `git commit --fixup=<해시>` + `git rebase -i --autosquash` | fixup 대상 자동 배치 |
| `git rebase --continue`·`--abort`·`--skip` | 충돌 시 계속 / 취소 / 건너뛰기 |
| `git merge <브랜치>` | 두 브랜치를 머지 커밋으로 합치기 |
| `git log --graph --oneline --all` | 히스토리가 갈래졌는지 일직선인지 시각 확인 |

## 6장 — 되돌리기 전략

| 명령 | 설명 |
|---|---|
| `git reset --soft <커밋>` | 포인터만 이동, Index·Workspace 유지 |
| `git reset --mixed <커밋>` (기본값) | 포인터+Index 이동, Workspace 유지 |
| `git reset --hard <커밋>` | 포인터+Index+Workspace 전부 되돌림 (위험) |
| `git revert <커밋>` | 취소하는 새 커밋 추가 (공유 브랜치에 안전) |
| `git reflog` | HEAD 이동 기록 확인 |
| `git reset --hard <reflog 해시>` | reflog로 찾은 시점으로 복구 |

## 7장 — 작업 공간 확장

| 명령 | 설명 |
|---|---|
| `git worktree add <경로> <브랜치>` | 다른 브랜치를 별도 디렉토리에 체크아웃 |
| `git worktree list` | 연결된 worktree 목록 확인 |
| `git worktree remove <경로>` | worktree 제거 |
| `git submodule add <url> <경로>` | 서브모듈 추가 (`.gitmodules`에 기록) |
| `git clone --recurse-submodules <url>` | 서브모듈까지 함께 clone |
| `git submodule update --init --recursive` | 비어있는 서브모듈 채우기 |
| `git submodule update --remote` | 서브모듈을 원격 최신 커밋으로 갱신 |

---

1장(Git vs GitHub), 7-3(모노레포 vs 멀티레포)은 특정 명령이 아니라 개념·전략 비교라 이 표에는 없음.
