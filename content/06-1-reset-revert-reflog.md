# 되돌리기 전략 — Reset(soft/mixed/hard) vs Revert, Reflog로 복구하기

## 한 줄 정의

- **Reset**: HEAD(와 브랜치 포인터)를 다른 커밋으로 옮기는 것. `--soft`/`--mixed`/`--hard` 옵션에 따라 Index·Workspace까지 건드리는 범위가 다르다.
- **Revert**: 특정 커밋의 변경을 취소하는 **새 커밋을 추가**하는 것. 히스토리를 지우지 않고 거꾸로 되돌린다.
- **Reflog**: HEAD가 가리켰던 위치들의 로컬 기록. reset/rebase 등으로 "사라진 것처럼 보이는" 커밋을 되찾는 안전망.

## 표로 비교 — `reset --soft` / `--mixed` / `--hard`

| | `--soft` | `--mixed` (기본값) | `--hard` |
|---|---|---|---|
| HEAD·브랜치 포인터 | 이동 | 이동 | 이동 |
| Index(스테이징) | 그대로 유지 | 초기화(unstage) | 초기화 |
| Workspace(파일) | 그대로 유지 | 그대로 유지 | 되돌린 커밋 상태로 덮어씀 |
| 취소된 커밋의 변경 내용 | 스테이징된 채로 남음 | 워킹 디렉토리에 unstaged로 남음 | 완전히 사라짐 |
| 쓰는 상황 | 커밋만 취소하고 다시 커밋하고 싶을 때 | 커밋도, 스테이징도 취소하되 수정 내용은 남기고 싶을 때 | 그 시점으로 완전히 되돌리고 싶을 때 (위험, 작업물 유실 가능) |

2-1에서 다룬 5가지 영역 기준으로 보면, 세 옵션은 정확히 "Local Repo(커밋)까지만 되돌릴지 / Index까지 되돌릴지 / Workspace까지 되돌릴지"의 범위 차이다.

## 표로 비교 — Reset vs Revert

| | Reset | Revert |
|---|---|---|
| 방식 | 브랜치 포인터를 과거로 이동 | 취소하는 새 커밋을 추가 |
| 기존 히스토리 | 그 이후 커밋들이 브랜치에서 빠짐 | 그대로 보존, 새 커밋만 하나 늘어남 |
| 공유 브랜치에 사용 | 위험 (rebase와 마찬가지로 히스토리 재작성) | 안전 (기존 커밋을 안 건드림) |
| 쓰는 상황 | 아직 push 안 한 로컬 실수 정리 | 이미 push되어 남들도 받아간 커밋을 취소해야 할 때 |

## Reflog로 복구하기

`reset --hard`나 rebase 때문에 커밋이 브랜치에서 떨어져 나가도, 실제 커밋 데이터(2-2/3-1에서 다룬 `objects/`)는 바로 지워지지 않는다. `git reflog`가 HEAD의 이동 기록을 전부 남겨두기 때문에, 여기서 되돌리기 전 해시를 찾아 복구할 수 있다.

```
git reflog
# 예: a1b2c3d HEAD@{0}: reset: moving to HEAD~2
#     789abcd HEAD@{1}: commit: fix bug

git reset --hard 789abcd    # reflog에서 찾은 해시로 복구
```

단, reflog는 **로컬 저장소에만 있고(원격엔 없음)**, 기본적으로 일정 기간(기본 90일)이 지나면 만료되어 사라진다는 한계가 있다.

## 명령어 치트시트 (6장 — 되돌리기 전략)

| 명령 | 설명 |
|---|---|
| `git reset --soft <커밋>` | 포인터만 이동, Index·Workspace 유지 |
| `git reset --mixed <커밋>` (기본값) | 포인터+Index 이동, Workspace 유지 |
| `git reset --hard <커밋>` | 포인터+Index+Workspace 전부 되돌림 (위험) |
| `git revert <커밋>` | 취소하는 새 커밋 추가 (공유 브랜치에 안전) |
| `git reflog` | HEAD 이동 기록 확인 |
| `git reset --hard <reflog 해시>` | reflog로 찾은 시점으로 복구 |


## 한 줄 요약

Reset은 포인터를 옮겨 히스토리 자체를 바꾸는 것(soft/mixed/hard로 Index·Workspace 반영 범위가 다름)이고, Revert는 새 커밋으로 안전하게 취소하는 것이며, Reflog는 이런 작업으로 사라진 것처럼 보이는 커밋도 되찾을 수 있는 로컬 안전망이다.
