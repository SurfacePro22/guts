# Fetch vs Pull 차이 (origin/main vs origin main)

## 한 줄 정의

- **Fetch**: 원격의 새 커밋·오브젝트를 로컬로 받아오기만 하고, 내 로컬 브랜치는 건드리지 않는다.
- **Pull**: Fetch + 받아온 내용을 내 로컬 브랜치에 merge(또는 rebase)까지 자동으로 수행한다.
- **`origin/main`**: 로컬에 저장된 "원격 추적 브랜치(remote-tracking branch)". 원격의 `main`이 마지막으로 fetch됐을 때의 스냅샷을 가리키는 로컬 포인터.
- **`origin main`** (슬래시 없음): `git push origin main`처럼 명령의 인자로 쓰인 것으로, "origin이라는 원격"과 "main이라는 브랜치"를 각각 가리키는 별개의 단어 두 개. 원격 서버 안의 실제 `main` 브랜치 자체를 뜻한다.

## 표로 비교 — Fetch vs Pull

| | Fetch | Pull |
|---|---|---|
| 하는 일 | 원격의 새 커밋·오브젝트만 받아옴 | Fetch + 로컬 브랜치에 merge/rebase까지 수행 |
| 건드리는 것 | `objects/`(새 오브젝트), `refs/remotes/origin/*`(추적 브랜치) | 위 전부 + `refs/heads/<브랜치>`(로컬 브랜치), 워킹 트리 파일 |
| 내 로컬 브랜치 | 그대로 유지 (안전) | 변경됨 (merge 커밋 생성 또는 rebase로 재작성) |
| 충돌 가능성 | 없음 (받아오기만 함) | 있음 (merge/rebase 충돌) |
| 명령 | `git fetch origin` | `git pull` = `git fetch` + `git merge` (`pull.rebase=true`면 `git rebase`) |

## 표로 비교 — `origin/main` vs `origin main`

| | `origin/main` | `origin main` |
|---|---|---|
| 정체 | 로컬 원격 추적 브랜치 (`refs/remotes/origin/main`) | 원격 저장소(origin) 안의 실제 `main` 브랜치 |
| 위치 | 내 컴퓨터 `.git` 안 | 원격 서버 안 |
| 최신성 | 마지막 fetch/pull 시점 스냅샷 (실시간 아님) | 원격에 push되는 즉시 최신 |
| 쓰이는 예 | `git log origin/main`, `git diff origin/main` | `git push origin main`, `git fetch origin main` |

## 명령어 치트시트 (3장 — 원격 저장소)

| 명령 | 설명 |
|---|---|
| `git clone <url>` | 원격 저장소 복제 (HTTPS/SSH/File/Git 프로토콜) |
| `git init --bare` | Bare Repository 생성 |
| `git config --system\|--global\|--local` | 설정 범위별 지정 (Local이 우선) |
| `git config --list --show-origin` | 각 설정값이 어느 파일에서 왔는지 확인 |
| `git remote add origin <url>` | 원격 추가 (`.git/config`에 기록됨) |
| `git fetch origin` | 원격 정보만 받아오기 (`refs/remotes/origin/*` 갱신) |
| `git pull` (`= fetch + merge/rebase`) | 받아오고 로컬 브랜치까지 반영 |
| `git log origin/main` / `git diff origin/main` | 로컬에 저장된 원격 추적 브랜치 기준 비교 |
| `git push origin main` | 원격의 실제 `main` 브랜치로 전송 |


## 한 줄 요약

Fetch는 원격 정보를 받아만 오고(`refs/remotes/origin/*`까지), Pull은 거기에 내 로컬 브랜치 반영까지 더한 것이며, `origin/main`은 그 결과를 담아두는 로컬 포인터인 반면 `origin main`은 원격 서버의 실제 브랜치를 가리키는 명령 인자다.
